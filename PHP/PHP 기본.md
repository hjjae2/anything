## PHP 기본

PHP(Php Hypertext Preprocessor) 는 server-side html-embedded 스크립트 언어("서버에서 실행되며 HTML 을 포함하는 스크립트 언어")이다.
- html 내용을 php 확장자로 저장하여도 아무런 지장이 없다.

1개의 프로세스를 생성한 후, 그 안에서 여러 개의 쓰레드를 생성하여 응답/처리하는 방식이다.

**PHP 를 이해하기 위해서 알아야 할 가장 큰 것은, 'PHP 는 1개가 아니다' 라는 것이다.**

[image](https://carlalexander.ca/uploads/2017/06/php-request-diagram.png)

<br>

### PHP 는 스크립트(Script)다.

PHP 로 웹 서비스를 할 때, 웹서버는 단순히 중개 역할을 해준다. 예를 들어, PHP 파일에 대한 요청이 오면 이 요청을 PHP Interpreter 에게 넘겨주고, 응답을 받아 브라우저에게 최종적으로 응답하는 것이다.

이러한 특징은 다른 언어(Java, Python, .Net)과의 차이점이기도 하다. 아래의 그림과 같이 Java, Python, .Net의 경우에는 Application Server (이하 WAS) 안에서 동작한다. 즉 WAS 가 필요하다. (PHP는 필요없다.)

[image](https://carlalexander.ca/uploads/2017/06/application-server-request-diagram.png)

PHP는 수정하게되면 곧바로 웹상에서 확인할 수 있다.
반면에 Java, Python 과 같은 언어는 (그럴 수도 있지만) WAS 를 재시작해야한다. (컴파일 언어니까..?) (Python 은 인터프리터 언어가 아닌가..)

### PHP는 웹 모듈이다.

즉, 웹서버와 같이 동작해야 사용할 수 있다. (혼자서 동작하지 않는다.) 

<br>

### PHP 의 처리 순서

1. client 가 http://domain.com/index.php 를 입력한다.
2. 웹서버는 해당 파일을 찾은 후 확장자를 검사/비교한다. (예를 들어, html|htm 이면 해당 파일을 그대로 브라우저로 보내준다.)
3. Php 엔진(Php interpreter) 가 처리해야할 파일(php|php3|inc 등의 확장자)라면, (웹서버에 장착되어 있는) php 엔진(php interpreter)에 보낸다.
4. php 엔진은 php 소스를 해석하여 html 코드로 만들고 웹서버로 응답한다.
5. 웹서버는 (응답받은) html 파일을 브라우저로 보내준다.

<br>

### \* CGI 란?

CGI 는 웹서버와 분리되어 있다. 요청 당 프로세스가 생성된다. 즉, N 개의 요청이라면 N 개의 프로세스가 생성되고 소멸된다.

웹서버로 요청이 들어오면 CGI 프로그램을 실행하여 1개의 프로세스를 생성한다. 로직이 처리되고 그 결과를 웹서버로 응답하고, 프로세스는 종료된다.

<br>

### Architecture

1. 브라우저에서 URL 요청
2. URL translation (URL -> IP -> 서버 찾음)
3. Header parsing, ... (Access-Control, Check MIME, Calls Handler)
4. PHP Script 실행 (tokenizing/lexing -> parsing -> compile -> optimize -> cache -> execute(zend engine))

<br><br>

> Reference
> 1. [PHP 개요 (웹 시스템, CGI, WAS 원리)](https://dev-youngjun.tistory.com/67)
> 2. [How does a PHP application work?](https://carlalexander.ca/php-application/)
> 3. [How does PHP works and what is its architecture?](https://stackoverflow.com/questions/24797539/how-does-php-works-and-what-is-its-architecture/46507205)
> 4. [How php interperter works](https://www.droptica.com/blog/how-php-interpreter-works/)