---
layout: post
title: "CodeIgniter :: xss_clean() 살펴보기"
author: "leehyunjae"
tags: ["codeigniter", "php"]
---

> CodeIgniter xss_clean() 메서드 살펴보기

<br>

### 개요

오픈소스에서 제공하는 `xss_clean()` 메서드를 사용하고 있는데, 오탐으로 인해 사용자가 입력한 데이터가 변경되어 문제가 발생했다.

<br>

문제가 발생한 문자열은 아래와 같다.

**원본 데이터**

```text
...
HgrSLy0VBO71Kjbr3Co7h76WU0QB1rVT61QGGT5/M/9pPf/muYF3eSdTkxDOZkXZT8vFUwbZiUyX
BGVvMC1epaCn329BFB4G3B+gJPa5k2OjlfogcT0jYWA0sUe1O/7bmW6sEfMwoTqh/VbGQLc/Eawz
mB8+Oxgvkhri175eT62jHhCKyseBKZU4JvyOLNzyMh4g3UU6TkuzKdLdc2Lmk4uEDT5qOs2Kkqnf
KuJyrG3L5oNFuRw=
-----END NEW CERTIFICATE REQUEST-----
```

**변경된 데이터**

```text
...
HgrSLy0VBO71Kjbr3Co7h76WU0QB1rVT61QGGT5/M/9pPf/muYF3eSdTkxDOZkXZT8vFUwbZiUyX
BGVvMC1epaCn329BFB4G3B+gJPa5k2OjlfogcT0jYWA0sUe1O/7bmW6sEfMwoTqh/VbGQLc/Eawz
mB8+Oxgvkhri175eT62jHhCKyseBKZU4JvyOLNzyMh4g3UU6TkuzKdLdc2Lmk4uEDT5qOs2Kkqnf KuJyrG3L5 NEW CERTIFICATE REQUEST-----
```

<br><br>

### xss_clean() 살펴보기

`xss_clean` 메서드가 어떻게 구현되어 있는지 살펴보기 위해 상속하고 있는 클래스를 따라가본다.

> `A Class ---> MYRest_Controller ---> REST_Controller ---> CI_Controller`

```php
class A extends MYRest_Controller {

    ...

    public function method1()
    {
	    try {
		    $data = $this->post('data', true, true);

            ...

```

```php
class MYRest_Controller {

    ...

    public function post($key = NULL, $xss_clean = TRUE, $required = FALSE, $type = FALSE){
		$rs = parent::post($key, $xss_clean);

        ...
	}
}
```

```php
class REST_Controller {

    ...
    
    public function post($key = NULL, $xss_clean = TRUE)
	{
		if ($key === NULL)
		{
			return $this->_post_args;
		}

		return array_key_exists($key, $this->_post_args) ? $this->_xss_clean($this->_post_args[$key], $xss_clean) : FALSE;
	}

    ...
}
```

```php
class REST_Controller {

    ...

    protected function _xss_clean($val, $process)
	{
		if (CI_VERSION < 2)
		{
			return $process ? $this->input->xss_clean($val) : $val;
		}

		return $process ? $this->security->xss_clean($val) : $val; // <-- 여기
	}

    ...
}
```

끝가지 올라가보면, security 클래스(라이브러리)의 xss_clean 메서드가 실행되는 것을 알 수 있다.

<br>


```php
class Security {

    ...

    public function xss_clean($str, $is_image = FALSE)
	{
		...

		// Remove evil attributes such as style, onclick and xmlns
		$str = $this->_remove_evil_attributes($str, $is_image);

		...

		return $str;
	}

    ...
}
```

실제로는 내용이 무척 긴 데, Z살펴봐야할 부분은 `_remove_evil_attributes()` 부분이다.

```php
$str = $this->_remove_evil_attributes($str, $is_image);
```

<br>

```php
class Security {

    ...

    protected function _remove_evil_attributes($str, $is_image)
	{
		// All javascript event handlers (e.g. onload, onclick, onmouseover), style, and xmlns
		$evil_attributes = array('on\w*', 'style', 'xmlns', 'formaction');

        ...

		do {
			$count = 0;
			$attribs = array();

			// find occurrences of illegal attribute strings with quotes (042 and 047 are octal quotes)
			preg_match_all('/('.implode('|', $evil_attributes).')\s*=\s*(\042|\047)([^\\2]*?)(\\2)/is', $str, $matches, PREG_SET_ORDER);

			foreach ($matches as $attr)
			{
				$attribs[] = preg_quote($attr[0], '/');
			}

			// find occurrences of illegal attribute strings without quotes
			preg_match_all('/('.implode('|', $evil_attributes).')\s*=\s*([^\s>]*)/is', $str, $matches, PREG_SET_ORDER);

			foreach ($matches as $attr)
			{
				print_r($attr[0]);
				$attribs[] = preg_quote($attr[0], '/');
			}

			// replace illegal attribute strings that are inside an html tag
			if (count($attribs) > 0)
			{
				$str = preg_replace('/(<?)(\/?[^><]+?)([^A-Za-z<>\-])(.*?)('.implode('|', $attribs).')(.*?)([\s><]?)([><]*)/i', '$1$2 $4$6$7$8', $str, -1, $count);
			}

		} while ($count);

		return $str;
	}

    ...
}
```

여기서도 핵심 부분은 아래 부분이다.

```php
preg_match_all('/('.implode('|', $evil_attributes).')\s*=\s*([^\s>]*)/is', $str, $matches, PREG_SET_ORDER);
```

<br>

위의 코드를 조금 간단하게 살펴보면 아래와 같다.

```php
preg_match((on\w*)\s*=\s*([^\s>]*), $str);
```

<br>

더 간략하게 하면 아래와 같고, 이 부분에 걸린 것을 알 수 있다.

```
`(on\w*)\s*=\s*([^\s>]*)`
= `(on\w*)=\s*([^\s]*)`
= `(on\w*)=\s.*`
```

<img src="/images/work/2021-08-22/xss_clean_regex_pattern.png" width="50%" height="50%" alt="xss_clean_regex_pattern">

