---
layout: post
title: "CodeIgniter :: Library 로딩이 동작하는 원리"
author: "leehyunjae"
tags: ["codeigniter", "php"]
---

> 이번 글은 CodeIgniter 2.x 기준으로 작성된 글이며, 커스터마이징 되어 있는 부분이 있기에 일반적인 상황과 조금은 다를 수 있습니다.

<br>

### 개요

회사에서 PHP 프레임워크 CodeIgniter(2.x, 3.x)를 사용하고 있다. 

CodeIgniter는 MVC 패턴으로, 대부분이 그렇듯 controller 는 client 의 요청을 받고, view 는 화면 노출을 위해 사용되며 model 은 DB I/O 역할을 수행한다. 여기에 추가로 library 라는 개념(디렉터리 구조에 포함)이 있다. Library 는 말그대로 공통으로 사용될 기능들에 대한 클래스(library)를 만들어두고 사용하기 위해 존재한다.

또, CodeIgniter 는 기본적으로 싱글톤 패턴으로 동작한다.

<br>

예를 들어 아래와 같이 사용한다.

```php
// github_library 라는 라이브러라가 있다고 가정한다.
$this->load->library("github_library");

$this->github_library->commit();
$this->github_library->pull();

...
```

일반적으로 위와 같이 사용하는데, 이 library 에 아래와 같이 alias 를 적용시켜 사용할 수도 있다.

```php
// $this->load->library(library파일명(경로), library 생성자 데이터, Alias)
$this->load->library("githubLibrary", $constructParameter, "github")

$this->github->commit();
$this->github->pull();
```

<br>

프로젝트마다 컨벤션(camelCase, snake_case)이 조금씩 다르다. 그래서 종종 alias 를 걸어 사용하곤 했다.

<br>

### 문제

그런데 우연히 아래와 같은 오류를 만났다.

```php
// A.class

$this->load->library("github_library", $constructParameter, "github");

$this->github->pull();

...
```

```php
// B.class

$this->load->library("github_library");

$this->github_library->pull(); // <-- 오류(Null) 발생
```

`A class` 에서 `github_library`에 alias 를 걸어 사용했고, 이후에 동작한 `B class` 에서는 alias 를 걸지 않은 상태였다. 그런데 `B class` 에서 load 한 `github_library`를 null 로 인식하고 오류를 발생시켰다. 

파일명을 잘못 작성한건지, 오타를 낸건지, 실수한 부분이 있는 줄 알고 한참을 헤맸다. 그러다 혹시나해서 `B class`에서 `$this->github->pull()` 과 같이 작성하고 동작시켜봤는데, 잘 동작했다.

> ...?

<br>

CodeIgniter 가 기본적으로 싱글톤 패턴으로 동작하기 때문이라고 보기엔 조금 애매했다. 아래와 같은 코드는 동작했기 때문이다.

```php
// A.class

$this->load->library("github_library");

$this->github_library->pull();

...
```

```php
// B.class

$this->load->library("github_library", $constructParameter, "github");

$this->github->pull(); // 정상 동작!
```

<br>

### Codeigniter 에서는 libary 를 어떻게 load 할까?

일단 CodeIgniter 의 global instance 를 출력해봤다. 내용이 정말 많았지만 그 중 `github` 은 (global instance의)property로 잡혀있는데 `github_library` 는 propery 로 잡혀있지 않은 것을 확인할 수 있었다.

> *Codeigniter 에서는 최상단에 하나의 Global Instance 가 있다. 이 object에 library, model, 각종 class 들을 property로 주입(?)시켜 사용하는 개념이다. Global Instance 는 `$this` 변수로 접근할 수 있어서 위의 예시처럼 `$this->github` 과 같이 사용할 수 있는 것이다.*

```php
Index Object
(
    ...

    [github] => github_Library Object
        (
            ...
        )

    ...
)
```

`github_library` property 가 없는 것을 확인하고 codeigniter 에서 library 를 어떻게 load 하는지 바로 확인했다.

<br>

`core/Loader.php` 쪽의 코드를 살펴보면, `library()` 메서드가 있는 것을 확인할 수 있다.

```php
...

public function library($library, $params = NULL, $object_name = NULL)
{
	if (empty($library))
	{
		return $this;
	}
	elseif (is_array($library))
	{
		foreach ($library as $key => $value)
		{
			if (is_int($key))
			{
				$this->library($value, $params);
			}
			else
			{
				$this->library($key, $params, $value);
			}
		}
		return $this;
	}
	if ($params !== NULL && ! is_array($params))
	{
		$params = NULL;
	}

	$this->_ci_load_library($library, $params, $object_name);
	return $this;
}
```

`library()` 메서드에서 다시 `_ci_load_library` 메서드가 동작하는 것을 알 수 있다.

<br>

`_ci_load_library()` 메서드를 살펴보자. 

```php
protected function _ci_load_library($class, $params = NULL, $object_name = NULL)
{
	// Get the class name, and while we're at it trim any slashes.
	// The directory path can be included as part of the class name,
	// but we don't want a leading slash
	$class = str_replace('.php', '', trim($class, '/'));

	// Was the path included with the class name?
	// We look for a slash to determine this
	if (($last_slash = strrpos($class, '/')) !== FALSE)
	{
		// Extract the path
		$subdir = substr($class, 0, ++$last_slash);

		// Get the filename from the path
		$class = substr($class, $last_slash);
	}
	else
	{
		$subdir = '';
	}

	$class = ucfirst($class);

    ...
}
```

> *위의 전체 코드는 [GitHub](https://github.com/bcit-ci/CodeIgniter/blob/develop/system/core/Loader.php) 에서  확인할 수 있습니다.* 

<br>

현재 회사에서는 (버전, 커스터마이징에 의해) 위의 내용과는 약간 다르다. 실제 소스는 아래의 형태와 같다.

```php
protected function _ci_load_library($class, $params = NULL, $object_name = NULL)
{
	// Get the class name, and while we're at it trim any slashes.
	// The directory path can be included as part of the class name,
	// but we don't want a leading slash
	$class = str_replace('.php', '', trim($class, '/'));

	// Was the path included with the class name?
	// We look for a slash to determine this
	$subdir = '';
	if (($last_slash = strrpos($class, '/')) !== FALSE)
	{
		// Extract the path
		$subdir = substr($class, 0, $last_slash + 1);

		// Get the filename from the path
		$class = substr($class, $last_slash + 1);
	}

    ...

    foreach ($this->_ci_library_paths as $path)
    {
        $filepath = $path.'libraries/'.$subdir.$class.'.php';

        if ( ! file_exists($filepath))
        {
            continue;
        }

        // Safety:  Was the class already loaded by a previous call?
        if (in_array($filepath, $this->_ci_loaded_files))
        {
            // Before we deem this to be a duplicate request, let's see
            // if a custom object name is being supplied.  If so, we'll
            // return a new instance of the object
            if ( ! is_null($object_name))
            {
                $CI =& get_instance();
                if ( ! isset($CI->$object_name))
                {
                    return $this->_ci_load_library($class, '', $params, $object_name);
                }
            }

            $is_duplicate = TRUE;
            log_message('debug', $class." class already loaded. Second attempt ignored.");
            return;
        }

        include_once($filepath);
        $this->_ci_loaded_files[] = $filepath;
        return $this->_ci_load_library($class, '', $params, $object_name);
    }

    ...
```

**위의 코드에서 핵심인 부분이 있다.**

```php

$filepath = $path.'libraries/'.$subdir.$class.'.php';

...
if (in_array($filepath, $this->_ci_loaded_files)) {
    // Before we deem this to be a duplicate request, let's see
    // if a custom object name is being supplied.  If so, we'll
    // return a new instance of the object
    if ( ! is_null($object_name))
    {
        $CI =& get_instance();
        if ( ! isset($CI->$object_name))
        {
            return $this->_ci_load_library($class, '', $params, $object_name);
        }
    }
	
    $is_duplicate = TRUE;
    log_message('debug', $class." class already loaded. Second attempt ignored.");
	
    return;
}
```

1. `library의 파일명(경로)`을 기준으로 (`$this->_ci_loaded_files`)이미 로드된 것인지 아닌지 판단한다.
   
2. 여기서 `$object_name` 이 바로 alias 인데, alias 값이 있으면 global instance 에 해당 alias로 propery가 설정되어 있는지 체크하고, 없다면 load 한다. (`$this->_ci_load_library($class, '', $params, $object_name)`)

3. <u>그런데 `$object_name` 이 없으면, object 의 property 를 검사하지 않고 중복된 선언이라며 log 를 남긴고 끝내버린다.</u> (`log_message('debug', $class." class already loaded. Second attempt ignored.");`)

<br>

**그렇기 때문에, 아래와 같은 상황이 발생할 수 있다.**

```php
// [CASE1]
// (1) alias load
// (2) alias 없이 load

// 두 번째로 load 한 Lib_test 는 무시된다.
$this->load->library("Lib_test", null, "test");
$this->load->library("Lib_test");
print_r($this->lib_test->getHi()); // 에러!!


// LOG
DEBUG - 2021-07-19 21:41:21 --> ...
DEBUG - 2021-07-19 21:41:21 --> Lib_test class already loaded. Second attempt ignored.
DEBUG - 2021-07-19 21:41:21 --> ...
```

<br>

```php
// [CASE2]
// (1) alias 없이 load
// (2) alias load

// 두 개의 library 모두 global instance 의 property 로 설정된다.
$this->load->library("Lib_test");
$this->load->library("Lib_test", null, "test");
print_r($this->lib_test->getHi()); // 정상!!

// LOG
DEBUG - 2021-07-19 21:43:24 --> ...
```

<br>

### 어떻게 해야할까

(현재 상황에서) 그럼에도 alias 를 사용하려면 어떻게 해야할까 생각했다.

**`core/Loader.php` 쪽의 코드를 수정하던지, 아니면 `alias` 를 사용하지 말던지.**

1. `core` 쪽의 코드는 말그대로 CodeIgniter 코어 역할을 하는 중요한 부분이다. core 파일의 변경은 프레임워크 전체에 영향을 미친다. 사실  윗 내용에 대한 수정은 큰 수정/영향 없이 수정할 수 있을 것이다.

2. `alias` 를 사용하지 말자는 것은, 그냥 포기하는 것은 아니다. 사실 여태껏 작성되어 있는 코드의 대부분이 alias가 적용되어 있지 않기 때문에 오히려 alias 를 적용한 코드가 통일성을 깨고 있다고 생각했기 때문이다.