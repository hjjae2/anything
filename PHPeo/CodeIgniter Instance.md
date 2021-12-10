## CodeIgniter load->library 분석 내용!

### 분석해보자.

아래 명령어에 대해 살펴보자.

`$this->load->library()`

[GitHub](https://github.com/bcit-ci/CodeIgniter/blob/develop/system/core/Loader.php) 이나 Project의 `System` 폴더 내에서 코드를 확인할 수 있다.


```php
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

		// Is this a stock library? There are a few special conditions if so ...
		if (file_exists(BASEPATH.'libraries/'.$subdir.$class.'.php'))
		{
			return $this->_ci_load_stock_library($class, $subdir, $params, $object_name);
		}

		// Safety: Was the class already loaded by a previous call?
		if (class_exists($class, FALSE))
		{
			$property = $object_name;
			if (empty($property))
			{
				$property = strtolower($class);
				isset($this->_ci_varmap[$property]) && $property = $this->_ci_varmap[$property];
			}

			$CI =& get_instance();
			if (isset($CI->$property))
			{
				log_message('debug', $class.' class already loaded. Second attempt ignored.');
				return;
			}

			return $this->_ci_init_library($class, '', $params, $object_name);
		}

		// Let's search for the requested library file and load it.
		foreach ($this->_ci_library_paths as $path)
		{
			// BASEPATH has already been checked for
			if ($path === BASEPATH)
			{
				continue;
			}

			$filepath = $path.'libraries/'.$subdir.$class.'.php';
			// Does the file exist? No? Bummer...
			if ( ! file_exists($filepath))
			{
				continue;
			}

			include_once($filepath);
			return $this->_ci_init_library($class, '', $params, $object_name);
		}

		// One last attempt. Maybe the library is in a subdirectory, but it wasn't specified?
		if ($subdir === '')
		{
			return $this->_ci_load_library($class.'/'.$class, $params, $object_name);
		}

		// If we got this far we were unable to find the requested class.
		log_message('error', 'Unable to load the requested class: '.$class);
		show_error('Unable to load the requested class: '.$class);
	}
```

<br>

현재 회사에서는 위의 내용과는 약간 다르다. (By 버전, 커스터마이징)

실제 소스는 아래의 형태와 같다. 이제 이것들을 실제로 line by line 으로 출력/테스트/검증 해보자.

```php
	protected function _ci_load_class($class, $params = NULL, $object_name = NULL)
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

		print_r("class => " . $class . "<br><br>");
        ...
```

```
common/session :
class => session

encrypt :
class => encrypt

common/Mobiledetect :
class => Mobiledetect

/common/Mobiledetect :
class => Mobiledetect

common/checklogin :
class => checklogin

common/Cookiesecure :
class => Cookiesecure
```

<br>

핵심 메서드인 `_ci_load_class` 전체 내용은 다음과 같다.


```php
	protected function _ci_load_class($class, $params = NULL, $object_name = NULL)
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

		// We'll test for both lowercase and capitalized versions of the file name
		foreach (array(ucfirst($class), strtolower($class)) as $class)
		{
			$subclass = APPPATH.'libraries/'.$subdir.config_item('subclass_prefix').$class.'.php';

			// Is this a class extension request?
			if (file_exists($subclass))
			{
				$baseclass = BASEPATH.'libraries/'.ucfirst($class).'.php';

				if ( ! file_exists($baseclass))
				{
					log_message('error', "Unable to load the requested class: ".$class);
					show_error("Unable to load the requested class: ".$class);
				}

				// Safety:  Was the class already loaded by a previous call?
				if (in_array($subclass, $this->_ci_loaded_files))
				{
					// Before we deem this to be a duplicate request, let's see
					// if a custom object name is being supplied.  If so, we'll
					// return a new instance of the object
					if ( ! is_null($object_name))
					{
						$CI =& get_instance();
						if ( ! isset($CI->$object_name))
						{
							return $this->_ci_init_class($class, config_item('subclass_prefix'), $params, $object_name);
						}
					}

					$is_duplicate = TRUE;
					log_message('debug', $class." class already loaded. Second attempt ignored.");
					return;
				}

				include_once($baseclass);
				include_once($subclass);
				$this->_ci_loaded_files[] = $subclass;

				return $this->_ci_init_class($class, config_item('subclass_prefix'), $params, $object_name);
			}

			// Lets search for the requested library file and load it.
			$is_duplicate = FALSE;
			foreach ($this->_ci_library_paths as $path)
			{
				$filepath = $path.'libraries/'.$subdir.$class.'.php';

				// Does the file exist?  No?  Bummer...
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
							return $this->_ci_init_class($class, '', $params, $object_name);
						}
					}

					$is_duplicate = TRUE;
					log_message('debug', $class." class already loaded. Second attempt ignored.");
					return;
				}

				include_once($filepath);
				$this->_ci_loaded_files[] = $filepath;
				return $this->_ci_init_class($class, '', $params, $object_name);
			}

		} // END FOREACH

		// One last attempt.  Maybe the library is in a subdirectory, but it wasn't specified?
		if ($subdir == '')
		{
			$path = strtolower($class).'/'.$class;
			return $this->_ci_load_class($path, $params);
		}

		// If we got this far we were unable to find the requested class.
		// We do not issue errors if the load call failed due to a duplicate request
		if ($is_duplicate == FALSE)
		{
			log_message('error', "Unable to load the requested class: ".$class);
			show_error("Unable to load the requested class: ".$class);
		}
	}
```

여기서 중요한 부분은 `$subclass` 가 아닌, `$filepath` 부분으로 시작되는 부분이다.

### `$subclass` 부분

`$subclass = APPPATH.'libraries/'.$subdir.config_item('subclass_prefix').$class.'.php';`

(`subclass_prefix = "MY_"`)

즉 library 이되, MY_~~~ 으로 되는 라이브러리를 로드하는 부분이다.

<br>

### `$filepath` 부분

`$filepath = $path.'libraries/'.$subdir.$class.'.php';`

우리가 보통 사용하는 libraries 밑의 파일을 찾는다.

핵심이 되는 부분은 다음과 같다.

```
if (in_array($filepath, $this->_ci_loaded_files))
{
	if ( ! is_null($object_name))
	{
		$CI =& get_instance();
		if ( ! isset($CI->$object_name))
		{
			return $this->_ci_init_class($class, '', $params, $object_name);
		}
	}
	$is_duplicate = TRUE;
	log_message('debug', $class." class already loaded. Second attempt ignored.");
	return;
}
```
`$object_name` 이 선언되어 있다면, CI 객체에서 해당 프로퍼티가 있는지 살피고 없으면 로드해준다.

그 외의 경우(alias 가 없거나, 중복된 alias)는 무시한다.


<br>

### 로그를 확인해보자.

로그에 다음과 같이 찍히는 것을 확인할 수 있다.

```
// 아래와 같이 사용한다면
$this->load->library("Lib_test", null, "test");
$this->load->library("Lib_test");
print_r($this->lib_test->getHi());


DEBUG - 2021-07-19 21:41:21 --> Helper loaded: url_helper
DEBUG - 2021-07-19 21:41:21 --> Model Class Initialized
DEBUG - 2021-07-19 21:41:21 --> Lib_test class already loaded. Second attempt ignored.
DEBUG - 2021-07-19 21:41:21 --> Model Class Initialized
DEBUG - 2021-07-19 21:41:21 --> User Agent Class Initialized
```

![image](https://user-images.githubusercontent.com/35790290/126161269-00bb8676-c53e-47f5-842f-8a455e9008ba.png)

```php
// 아래와 같이 사용한다면,

$this->load->library("Lib_test");
$this->load->library("Lib_test", null, "test");
print_r($this->lib_test->getHi());

DEBUG - 2021-07-19 21:43:24 --> Helper loaded: /hcommon/hosting_helper
DEBUG - 2021-07-19 21:43:24 --> Helper loaded: url_helper
DEBUG - 2021-07-19 21:43:24 --> Model Class Initialized
```

![image](https://user-images.githubusercontent.com/35790290/126161580-533235ff-aa86-46e1-8e8c-fcc861554460.png)

<br>

### 결론

싱글톤으로 Loaded library 를 관리할 때, 파일의 path/name 형태로 Key 값을 관리한다.


```php
$filepath = $path.'libraries/'.$subdir.$class.'.php';
```

alias 가 있다면, CI 객체에 해당 alias의 property 가 있는지 확인한다.

없다면 선언해주고, 있다면 중복된 값이기에 무시한다. (logging)

```
if ( ! is_null($object_name))
{
	$CI =& get_instance();
		if ( ! isset($CI->$object_name))
		{
			return $this->_ci_init_class($class, '', $params, $object_name);
		}
}
```

<br><br>

## [참고] CI 객체

```php
Index Object
(
    [accept_ip:MY_Controller:private] => Array
        (
            [0] => ...
            [1] => ...
            ...
        )

    [gadmin_cookie] => Array
        (
            [islogin] => 
        )

    [exception_path:MY_Controller:private] => Array
        (
            [0] => ...
            [1] => ...
            ...
        )

    [db_h] => 
    [benchmark] => CI_Benchmark Object
        (
            [marker] => Array
                (
                    [total_execution_time_start] => 0.10407300 1626141170
                    [loading_time:_base_classes_start] => 0.10408600 1626141170
                    [loading_time:_base_classes_end] => 0.13011000 1626141170
                    [controller_execution_time_( index / index )_start] => 0.13014600 1626141170
                )

        )

    [hooks] => CI_Hooks Object
        (
            [enabled] => 
            [hooks] => Array
                (
                )

            [in_progress] => 
        )

    [config] => CI_Config Object
        (
            [config] => Array
                (
                    [base_url] => http://leehj-gadmin.gabia.com
                    [index_page] => index.php
                    ...
                )
            [is_loaded] => Array
                (
                    [0] => application/config/config_ssl.php
                    [1] => ...
                    ...
                )

            [_config_paths] => Array
                (
                    [0] => application/
                )

        )

    [log] => CI_Log Object
        (
            [_threshold:protected] => 1
            [_enabled:protected] => 1
            [_levels:protected] => Array
                (
                    [ERROR] => 1
                    [INFO] => 2
                    [DEBUG] => 3
                    [ALL] => 4
                )

            [initialized:protected] => 1
        )

    [exceptions] => CI_Exceptions Object
        (
            [action] => 
            [severity] => 
            [message] => 
            [filename] => 
            [line] => 
            [ob_level] => 1
            [levels] => Array
                (
                    [1] => Error
                    [2] => Warning
                    [4] => Parsing Error
                    [8] => Notice
                    [16] => Core Error
                    [32] => Core Warning
                    [64] => Compile Error
                    [128] => Compile Warning
                    [256] => User Error
                    [512] => User Warning
                    [1024] => User Notice
                    [2048] => Runtime Notice
                )

        )

    [utf8] => CI_Utf8 Object
        (
        )

    [uri] => CI_URI Object
        (
            [keyval] => Array
                (
                )

            [uri_string] => ghosting/ssl
            [segments] => Array
                (
                    [1] => ghosting
                    [2] => ssl
                )

            [rsegments] => Array
                (
                    [1] => index
                    [2] => index
                )

            [config] => CI_Config Object
                (
                    [config] => Array
                        (
                            [base_url] => http://leehj-gadmin.gabia.com
                            [index_page] => ...
                            ...
                        )

                    [is_loaded] => Array
                        (
                            [0] => application/config/config_ssl.php
                            [1] => ...
                            ...
                        )

                    [_config_paths] => Array
                        (
                            [0] => application/
                        )
                )
        )

    [router] => MY_Router Object
        (
            [config] => CI_Config Object
                (
                    [config] => Array
                        (
                            [base_url] => http://leehj-gadmin.gabia.com
                            [index_page] => index.php
                            ...
                        )

                    [is_loaded] => Array
                        (
                            [0] => application/config/config_ssl.php
                            [1] => ...
                            ...
                        )

                    [_config_paths] => Array
                        (
                            [0] => application/
                        )

                )

            [routes] => Array
                (
                    [default_controller] => index
                    [404_override] => 
                    [design/(:any)] => design/index/$1
                    ...
                )

            [error_routes] => Array
                (
                )

            [class] => index
            [method] => index
            [directory] => ghosting/ssl/
            [default_controller] => index
            [uri] => CI_URI Object
                (
                    ...
                )

        )

    [output] => CI_Output Object
        (
            [final_output:protected] => 
            [cache_expiration:protected] => 0
            [headers:protected] => Array
                (
                )

            [mime_types:protected] => Array
                (
                    [hqx] => application/mac-binhex40
                    [cpt] => application/mac-compactpro
                    [csv] => Array
                        (
                            [0] => text/x-comma-separated-values
                            [1] => text/comma-separated-values
                            [2] => application/octet-stream
                            [3] => application/vnd.ms-excel
                            [4] => application/x-csv
                            [5] => text/x-csv
                            [6] => text/csv
                            [7] => application/csv
                            [8] => application/excel
                            [9] => application/vnd.msexcel
                            [10] => text/plain
                        )

                    [bin] => application/macbinary
                    [dms] => application/octet-stream
                    [lha] => application/octet-stream
                    [lzh] => application/octet-stream
                    [exe] => Array
                        (
                            [0] => application/octet-stream
                            [1] => application/x-msdownload
                        )

                    [class] => application/octet-stream
                    [psd] => application/x-photoshop
                    [so] => application/octet-stream
                    [sea] => application/octet-stream
                    [dll] => application/octet-stream
                    [oda] => application/oda
                    [pdf] => Array
                        (
                            [0] => application/pdf
                            [1] => application/x-download
                        )

                    [ai] => application/postscript
                    [eps] => application/postscript
                    [ps] => application/postscript
                    [smi] => application/smil
                    [smil] => application/smil
                    [mif] => application/vnd.mif
                    [xls] => Array
                        (
                            [0] => application/excel
                            [1] => application/vnd.ms-excel
                            [2] => application/msexcel
                        )

                    [ppt] => Array
                        (
                            [0] => application/powerpoint
                            [1] => application/vnd.ms-powerpoint
                        )

                    [wbxml] => application/wbxml
                    [wmlc] => application/wmlc
                    [dcr] => application/x-director
                    [dir] => application/x-director
                    [dxr] => application/x-director
                    [dvi] => application/x-dvi
                    [gtar] => application/x-gtar
                    [gz] => application/x-gzip
                    [php] => application/x-httpd-php
                    [php4] => application/x-httpd-php
                    [php3] => application/x-httpd-php
                    [phtml] => application/x-httpd-php
                    [phps] => application/x-httpd-php-source
                    [js] => application/x-javascript
                    [swf] => application/x-shockwave-flash
                    [sit] => application/x-stuffit
                    [tar] => application/x-tar
                    [tgz] => Array
                        (
                            [0] => application/x-tar
                            [1] => application/x-gzip-compressed
                        )

                    [xhtml] => application/xhtml+xml
                    [xht] => application/xhtml+xml
                    [zip] => Array
                        (
                            [0] => application/x-zip
                            [1] => application/zip
                            [2] => application/x-zip-compressed
                        )

                    [mid] => audio/midi
                    [midi] => audio/midi
                    [mpga] => audio/mpeg
                    [mp2] => audio/mpeg
                    [mp3] => Array
                        (
                            [0] => audio/mpeg
                            [1] => audio/mpg
                            [2] => audio/mpeg3
                            [3] => audio/mp3
                        )

                    [aif] => audio/x-aiff
                    [aiff] => audio/x-aiff
                    [aifc] => audio/x-aiff
                    [ram] => audio/x-pn-realaudio
                    [rm] => audio/x-pn-realaudio
                    [rpm] => audio/x-pn-realaudio-plugin
                    [ra] => audio/x-realaudio
                    [rv] => video/vnd.rn-realvideo
                    [wav] => Array
                        (
                            [0] => audio/x-wav
                            [1] => audio/wave
                            [2] => audio/wav
                        )

                    [bmp] => Array
                        (
                            [0] => image/bmp
                            [1] => image/x-windows-bmp
                        )

                    [gif] => image/gif
                    [jpeg] => Array
                        (
                            [0] => image/jpeg
                            [1] => image/pjpeg
                        )

                    [jpg] => Array
                        (
                            [0] => image/jpeg
                            [1] => image/pjpeg
                        )

                    [jpe] => Array
                        (
                            [0] => image/jpeg
                            [1] => image/pjpeg
                        )

                    [png] => Array
                        (
                            [0] => image/png
                            [1] => image/x-png
                        )

                    [tiff] => image/tiff
                    [tif] => image/tiff
                    [css] => text/css
                    [html] => text/html
                    [htm] => text/html
                    [shtml] => text/html
                    [txt] => text/plain
                    [text] => text/plain
                    [log] => Array
                        (
                            [0] => text/plain
                            [1] => text/x-log
                        )

                    [rtx] => text/richtext
                    [rtf] => text/rtf
                    [xml] => text/xml
                    [xsl] => text/xml
                    [mpeg] => video/mpeg
                    [mpg] => video/mpeg
                    [mpe] => video/mpeg
                    [qt] => video/quicktime
                    [mov] => video/quicktime
                    [avi] => video/x-msvideo
                    [movie] => video/x-sgi-movie
                    [doc] => application/msword
                    [docx] => Array
                        (
                            [0] => application/vnd.openxmlformats-officedocument.wordprocessingml.document
                            [1] => application/zip
                        )

                    [xlsx] => Array
                        (
                            [0] => application/vnd.openxmlformats-officedocument.spreadsheetml.sheet
                            [1] => application/zip
                        )

                    [word] => Array
                        (
                            [0] => application/msword
                            [1] => application/octet-stream
                        )

                    [xl] => application/excel
                    [eml] => message/rfc822
                    [json] => Array
                        (
                            [0] => application/json
                            [1] => text/json
                        )

                )

            [enable_profiler:protected] => 
            [_zlib_oc:protected] => 
            [_profiler_sections:protected] => Array
                (
                )

            [parse_exec_vars:protected] => 1
        )

    [security] => CI_Security Object
        (
            [_xss_hash:protected] => 
            [_csrf_hash:protected] => 
            [_csrf_expire:protected] => 7200
            [_csrf_token_name:protected] => ci_csrf_token
            [_csrf_cookie_name:protected] => ci_csrf_token
            [_never_allowed_str:protected] => Array
                (
                    [document.cookie] => [removed]
                    [document.write] => [removed]
                    [.parentNode] => [removed]
                    [.innerHTML] => [removed]
                    [window.location] => [removed]
                    [-moz-binding] => [removed]
                    [] => -->
                    [ <![CDATA[
                    [] => <comment>
                )

            [_never_allowed_regex:protected] => Array
                (
                    [0] => javascript\s*:
                    [1] => expression\s*(\(|&\#40;)
                    [2] => vbscript\s*:
                    [3] => Redirect\s+302
                    [4] => (["'])?data\s*:[^\1]*?base64[^\1]*?,[^\1]*?\1?
                )

        )

    [input] => CI_Input Object
        (
            [ip_address] => 10.25.13.34
            [user_agent] => Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36
            [_allow_get_array] => 1
            [_standardize_newlines] => 1
            [_enable_xss] => 
            [_enable_csrf] => 
            [headers:protected] => Array
                (
                )

            [security] => CI_Security Object
                (
                    [_xss_hash:protected] => 
                    [_csrf_hash:protected] => 
                    [_csrf_expire:protected] => 7200
                    [_csrf_token_name:protected] => ci_csrf_token
                    [_csrf_cookie_name:protected] => ci_csrf_token
                    [_never_allowed_str:protected] => Array
                        (
                            [document.cookie] => [removed]
                            [document.write] => [removed]
                            [.parentNode] => [removed]
                            [.innerHTML] => [removed]
                            [window.location] => [removed]
                            [-moz-binding] => [removed]
                            [] => -->
                            [ <![CDATA[
                            [] => <comment>
                        )

                    [_never_allowed_regex:protected] => Array
                        (
                            [0] => javascript\s*:
                            [1] => expression\s*(\(|&\#40;)
                            [2] => vbscript\s*:
                            [3] => Redirect\s+302
                            [4] => (["'])?data\s*:[^\1]*?base64[^\1]*?,[^\1]*?\1?
                        )

                )

            [uni] => CI_Utf8 Object
                (
                )

        )

    [lang] => CI_Lang Object
        (
            [language] => Array
                (
                    [error_204] => 데이터가 없습니다
                    [error_400] => 잘못된 요청입니다. 입력값들을 다시 확인하세요
                    [error_404] => 존재하지 않는 페이지로의 요청입니다
                    [error_405] => 정상적인 요청이 아닙니다
                    [error_500] => 일시적 서버 에러입니다. 잠시후 다시 시도하세요
                )

            [is_loaded] => Array
                (
                    [0] => error_msg_lang.php
                )

        )

    [load] => MY_Loader Object
        (
            [_ci_ob_level:protected] => 1
            [_ci_view_paths:protected] => Array
                (
                    [application/views/] => 1
                )

            [_ci_library_paths:protected] => Array
                (
                    [0] => application/
                    [1] => /home/guser/gm2102943/gabia-framework/codeigniter/base/system/
                )

            [_ci_model_paths:protected] => Array
                (
                    [0] => application/
                )

            [_ci_helper_paths:protected] => Array
                (
                    [0] => application/
                    [1] => /home/guser/gm2102943/gabia-framework/codeigniter/base/system/
                )

            [_base_classes:protected] => Array
                (
                    [benchmark] => Benchmark
                    [hooks] => Hooks
                    [config] => Config
                    [log] => Log
                    [exceptions] => Exceptions
                    [utf8] => Utf8
                    [uri] => URI
                    [router] => Router
                    [output] => Output
                    [security] => Security
                    [input] => Input
                    [lang] => Lang
                    [loader] => Loader
                )

            [_ci_cached_vars:protected] => Array
                (
                )

            [_ci_classes:protected] => Array
                (
                    [gsession] => gsession
                    [encrypt] => encrypt
                    [gabialib] => gabialib
                    [parser] => parser
                    [form_validation] => form_validation
                    [lib_login] => lib_login
                    [cookiesecure] => cookiesecure
                    [lib_ssl] => sslssl
                    [gabiaapiclient] => gapi
                    [lib_kica] => kicaLibrary
                    [lib_ucert] => ucertLibrary
                    [lib_setting_process] => settingProcessLibrary
                )

            [_ci_loaded_files:protected] => Array
                (
                    [0] => application/libraries/Gsession.php
                    [1] => /home/guser/gm2102943/gabia-framework/codeigniter/base/system/libraries/Encrypt.php
                    [2] => application/libraries/Gabialib.php
                    [3] => /home/guser/gm2102943/gabia-framework/codeigniter/base/system/libraries/Parser.php
                    [4] => /home/guser/gm2102943/gabia-framework/codeigniter/base/system/libraries/Form_validation.php
                    [5] => application/libraries/Lib_login.php
                    [6] => application/libraries/common/Cookiesecure.php
                    [7] => application/libraries/ssl/Lib_ssl.php
                    [8] => application/libraries/common/gabiaapiclient.php
                    [9] => application/libraries/ssl/Lib_kica.php
                    [10] => application/libraries/ssl/Lib_ucert.php
                    [11] => application/libraries/ssl/Lib_setting_process.php
                )

            [_ci_models:protected] => Array
                (
                    [0] => m_setting
                )

            [_ci_helpers:protected] => Array
                (
                    [exceptionhandler_helper] => 1
                    [language_helper] => 1
                    [string_helper] => 1
                    [form_helper] => 1
                    [url_helper] => 1
                    [gadmin_helper] => 1
                    [common/script_helper] => 1
                    [common/hosting_helper] => 1
                    [script_helper] => 1
                )

            [_ci_varmap:protected] => Array
                (
                    [unit_test] => unit
                    [user_agent] => agent
                )

        )

    [db] => CI_DB_mysqli_driver Object
        (
        )

    [gsession] => Gsession Object
        (
            [sess_encrypt_cookie] => 
            [sess_use_database] => 1
            [sess_table_name] => ci_sessions
            [sess_expiration] => 36000
            [sess_expire_on_close] => 1
            [sess_match_ip] => 1
            [sess_match_useragent] => 1
            [sess_cookie_name] => adminsession
            [cookie_prefix] => 
            [cookie_path] => /
            [cookie_domain] => gabia.com
            ...
        )

    [encrypt] => CI_Encrypt Object
        (
            [CI] => Index Object
            [encryption_key] => 
            [_hash_type] => sha1
            [_mcrypt_exists] => 1
            [_mcrypt_cipher] => 
            [_mcrypt_mode] => 
        )

    [gabialib] => Gabialib Object
        (
        )

    [parser] => CI_Parser Object
        (
            [l_delim] => {
            [r_delim] => }
            [object] => 
        )

    [form_validation] => CI_Form_validation Object
        (
            [CI:protected] => Index Object
            [_field_data:protected] => Array
                (
                )

            [_config_rules:protected] => Array
                (
                )

            [_error_array:protected] => Array
                (
                )

            [_error_messages:protected] => Array
                (
                )

            [_error_prefix:protected] => 

            [_error_suffix:protected] => 


            [error_string:protected] => 
            [_safe_form_data:protected] => 
        )

    [lib_login] => Lib_login Object
        (
            [_ci:protected] => Index Object
            [gadmin_cookie:protected] => 
            [today:Lib_login:private] => 2021-07-13
            [cookie_expire:Lib_login:private] => 0
        )

    [cookiesecure] => Cookiesecure Object
        (
            [_hex16_hash_key:protected] => gabiataesachi
        )

    [work_name] => 
    [work_employee] => 
    [work_ip] => 
    [gdb] => CI_DB_gabiaoci8_driver Object
        (
            ...
        )

    [cdb] => CI_DB_gabiaoci8_driver Object
        (
            ...
        )

    [m_setting] => M_setting Object
        (
            [_CI:protected] => Index Object
            ...
        )

    [gapi] => gabiaApiClient Object
        (
            [api_host:protected] => leehj.gapi.gabia.com
            [auth_uri:protected] => /oauth/token
            [valid_uri:protected] => /oauth/validation
            [m_szResultCode:protected] => 200
            [m_szResultMessage:protected] => 
            [m_bHttps:protected] => 
            [m_nPort:protected] => 80
            [m_szToken:protected] => YjUzOWYyNWEzNDJkMjIxZjgzYTdkYzA3MWNlY2E3
            [m_szId:protected] => oracle
            [m_szKey:protected] => admin
            [m_nExpire:protected] => 3600
            [m_szCreateOn:protected] => 2021-07-13 10:01:00
            [m_szClientCreateTime:protected] => 2021-07-13 10:47:39
            [headers:protected] => 
            [m_szData:gabiaApiClient:private] => Array
                (
                    [carve_code] => sslhosting
                )

            [m_szAcceptType:gabiaApiClient:private] => json
            [m_oData:gabiaApiClient:private] => {"code":"S","data":{"TD_N":{"name":"Thawte Code Signing","retail_price":"180000","sale_status":"Y","carve_code":"sslhosting","gtype":"TD_N","ssl_level":null,"ssl_brand":null,"kind":"single","license_cnt":"1"},"TC_N":{"name":"Thawte Wildcard","retail_price":"820000","sale_status":"Y","carve_code":"sslhosting","gtype":"TC_N","ssl_level":null,"ssl_brand":null,"kind":"wild","license_cnt":"1"},"TW_N":{"name":"Thawte Webserver","retail_price":"180000","sale_status":"Y","carve_code":"sslhosting","gtype":"TW_N","ssl_level":null,"ssl_brand":null,"kind":"single","license_cnt":"1"},"VS_N":{"name":"Secure Server","retail_price":"360000","sale_status":"Y","carve_code":"sslhosting","gtype":"VS_N","ssl_level":null,"ssl_brand":null,"kind":"single","license_cnt":"1"},"VG_N":{"name":"Global Server","retail_price":"600000","sale_status":"Y","carve_code":"sslhosting","gtype":"VG_N","ssl_level":null,"ssl_brand":null,"kind":"single","license_cnt":"1"},"GD":{"name":"Global Code Signing","retail_price":"330000","sale_status":"Y","carve_code":"sslhosting","gtype":"GD","ssl_level":null,"ssl_brand":null,"kind":"single","license_cnt":"1"},"GE":{"name":"Extended SSL","retail_price":"1104000","sale_status":"Y","carve_code":"sslhosting","gtype":"GE","ssl_level":"EV","ssl_brand":"GLOBALSIGN","kind":"single","license_cnt":"1"},"GDW":{"name":"Domain Wildcard","retail_price":"660000","sale_status":"Y","carve_code":"sslhosting","gtype":"GDW","ssl_level":"DV","ssl_brand":"GLOBALSIGN","kind":"wild","license_cnt":"1"},"M":{"name":"Domain SSL","retail_price":"200000","sale_status":"Y","carve_code":"sslhosting","gtype":"M","ssl_level":"DV","ssl_brand":"GLOBALSIGN","kind":"single","license_cnt":"1"},"A":{"name":"AlphaSign","retail_price":"40000","sale_status":"Y","carve_code":"sslhosting","gtype":"A","ssl_level":"DV","ssl_brand":"GLOBALSIGN","kind":"single","license_cnt":"1"},"CCS":{"name":"Sectigo Code Signing","retail_price":"140000","sale_status":"Y","carve_code":"sslhosting","gtype":"CCS","ssl_level":"CODE","ssl_brand":"SECTIGO","kind":"single","license_cnt":"1"},"CEV":{"name":"Sectigo EV","retail_price":"1200000","sale_status":"Y","carve_code":"sslhosting","gtype":"CEV","ssl_level":"EV","ssl_brand":"SECTIGO","kind":"single","license_cnt":"1"},"CW":{"name":"Sectigo Wildcard","retail_price":"800000","sale_status":"Y","carve_code":"sslhosting","gtype":"CW","ssl_level":"OV","ssl_brand":"SECTIGO","kind":"wild","license_cnt":"1"},"CM":{"name":"Sectigo Multi","retail_price":"280000","sale_status":"Y","carve_code":"sslhosting","gtype":"CM","ssl_level":"DV","ssl_brand":"SECTIGO","kind":"multi","license_cnt":"3"},"CR":{"name":"Sectigo Premium","retail_price":"200000","sale_status":"Y","carve_code":"sslhosting","gtype":"CR","ssl_level":"OV","ssl_brand":"SECTIGO","kind":"single","license_cnt":"1"},"CP":{"name":"Sectigo Pro","retail_price":"180000","sale_status":"Y","carve_code":"sslhosting","gtype":"CP","ssl_level":"OV","ssl_brand":"SECTIGO","kind":"single","license_cnt":"1"},"CG":{"name":"Sectigo Global","retail_price":"90000","sale_status":"Y","carve_code":"sslhosting","gtype":"CG","ssl_level":"OV","ssl_brand":"SECTIGO","kind":"single","license_cnt":"1"},"CB":{"name":"Sectigo Basic","retail_price":"40000","sale_status":"Y","carve_code":"sslhosting","gtype":"CB","ssl_level":"DV","ssl_brand":"SECTIGO","kind":"single","license_cnt":"1"},"GM":{"name":"GlobalSign IP","retail_price":"450000","sale_status":"Y","carve_code":"sslhosting","gtype":"GM","ssl_level":"OV","ssl_brand":"GLOBALSIGN","kind":"single","license_cnt":"1"},"DGS":{"name":"Digicert Standard","retail_price":"270000","sale_status":"Y","carve_code":"sslhosting","gtype":"DGS","ssl_level":"OV","ssl_brand":"DIGICERT","kind":"single","license_cnt":"1"},"DGM":{"name":"Digicert Multi","retail_price":"810000","sale_status":"Y","carve_code":"sslhosting","gtype":"DGM","ssl_level":"OV","ssl_brand":"DIGICERT","kind":"multi","license_cnt":"3"},"DGE":{"name":"Digicert EV","retail_price":"880000","sale_status":"Y","carve_code":"sslhosting","gtype":"DGE","ssl_level":"EV","ssl_brand":"DIGICERT","kind":"single","license_cnt":"1"}}}
            [m_nLoginTryCount:gabiaApiClient:private] => 0
            [m_requestUrl:gabiaApiClient:private] => ghosting/api_goods/info?carve_code=sslhosting
            [m_Grant:gabiaApiClient:private] => client_credentials
            [tokenReissueCnt:gabiaApiClient:private] => 1
            [maxTokenReissueCnt:gabiaApiClient:private] => 5
            [maxReAuthCnt:gabiaApiClient:private] => 2
            [reAuthCnt:gabiaApiClient:private] => 1
            [m_maxTimeout:protected] => 60
        )

    [sslLibrary] => Lib_ssl Object
        (
            [HTTP_SUCCESS_CODE:Lib_ssl:private] => 200
            [LABEL_TYPE_ID_SSL_LEVEL:Lib_ssl:private] => 18
            [LOG_PATH:Lib_ssl:private] => /tmp/ssl/setting/ssl/SSL_20210713.log
            [ERROR_MESSAGE:Lib_ssl:private] => 
            [SECTIGO_GTYPES:Lib_ssl:private] => Array
                (
                    ...
                )

            [GLOBALSIGN_GTYPES:Lib_ssl:private] => Array
                (
                    ...
                )

            [DIGICERT_GTYPES:Lib_ssl:private] => Array
                (
                    ...
                )

            [WILDCARD_GTYPES:Lib_ssl:private] => Array
                (
                    ...
                )

            [_ci] => Index Object
            ...
        )

    [sslssl] => Lib_ssl Object
        (
            [HTTP_SUCCESS_CODE:Lib_ssl:private] => 200
            [LABEL_TYPE_ID_SSL_LEVEL:Lib_ssl:private] => 18
            [LOG_PATH:Lib_ssl:private] => /tmp/ssl/setting/ssl/SSL_20210713.log
            [ERROR_MESSAGE:Lib_ssl:private] => 
            [SECTIGO_GTYPES:Lib_ssl:private] => Array
                (
                    [CCS] => Array
                        (
                            [name] => Sectigo Code Signing
                            [retail_price] => 140000
                            [sale_status] => Y
                            [carve_code] => sslhosting
                            [gtype] => CCS
                            [ssl_level] => CODE
                            [ssl_brand] => SECTIGO
                            [kind] => single
                            [license_cnt] => 1
                        )
                    ...
                )

            [GLOBALSIGN_GTYPES:Lib_ssl:private] => Array
                (
                    ...
                )

            [DIGICERT_GTYPES:Lib_ssl:private] => Array
                (
                    ...
                )

            ...

            [_ci] => Index Object

            ...
        )

    [kicaLibrary] => Lib_kica Object
        (
            [HTTP_SUCCESS_CODE:Lib_kica:private] => 200
            [KICA_SUCCESS_CODE:Lib_kica:private] => 0000
            [BASE_URI:Lib_kica:private] => https://www.kicassl.com
            [REQUEST_CERT_URI:Lib_kica:private] => /gw/api/apiResellerGateWay.sg
            [REQUEST_CERT_COS_URI:Lib_kica:private] => /gw/api/apiResellerGateWayCos.sg
            [KICA_CLIENT:Lib_kica:private] => GuzzleHttp\Client Object
                (
                    ...
                )
            [LOG_PATH:Lib_kica:private] => /tmp/ssl/setting/kica/KICA_DEFAULT_20210713.log
            [ERROR_MESSAGE:Lib_kica:private] => 
            [_ci] => Index Object
            [GET_HASH_URI] => /gw/api/apiResellerGetHash.sg
            ...
        )

    [ucertLibrary] => Lib_ucert Object
        (
            [UCERT_URL:Lib_ucert:private] => https://realapi.ucert.co.kr
            [CLIENT_OPT:Lib_ucert:private] => Array
                (
                    [headers] => Array
                        (
                            [content-type] => application/json
                        )

                    [timeout] => 30
                )

            [ssl_type:Lib_ucert:private] => DV
            [UCERT_GOODS:Lib_ucert:private] => Array
                (
                    [GDW] => Array
                        (
                            [type] => DV
                            [name] => Glo.dom.wild
                        )
                        ...
                )

            [logPath:Lib_ucert:private] => /tmp/ssl/setting/ucert/
            [logName:Lib_ucert:private] => /tmp/ssl/setting/ucert/20210713.txt
            [periodMonth:Lib_ucert:private] => 12
            [__message:Lib_ucert:private] => 
            [ssl_pwd:Lib_ucert:private] => NOPASS
            [orderKind:Lib_ucert:private] => 
            [_el] => Index Object
            [db] => CI_DB_gabiaoci8_driver Object
                (
                    ...
                )
            [partnerId] => gabiainc
            [partnerKey] => gabia4370
            [api] => GuzzleHttp\Client Object
                (
                    ...
                )
            [authToken] => Array
                (
                    ...
                )

        )

    [settingProcessLibrary] => Lib_setting_process Object
        (
            [_ci] => Index Object
            ...
        )

)


```

```
$this->load->library(['ssl/Lib_ssl','ssl/Lib_kica', 'ssl/Lib_ucert', 'ssl/Lib_setting_process']);
$this->load->library("ssl/Lib_ssl", null, "sslssl2");
print_r($this->lib_ssl->isMultiType("CM"));
print_r($this->sslssl2->isMultiType("CM"));
```
