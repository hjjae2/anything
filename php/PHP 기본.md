## PHP 기본

PHP(Php Hypertext Preprocessor) 는 server-side html-embedded 스크립트 언어("서버에서 실행되며 HTML 을 포함하는 스크립트 언어")이다.
- html 내용을 php 확장자로 저장하여도 아무런 지장이 없다.

1개의 프로세스를 생성한 후, 그 안에서 여러 개의 쓰레드를 생성하여 응답/처리하는 방식이다.

<br>

### PHP는 웹 모듈이다.

즉, 웹서버와 같이 동작해야 사용할 수 있다. (혼자서 동작하지 않는다.) 

<br>

### PHP 의 처리 순서

1. client 가 http://domain.com/index.php 를 입력한다.
2. 웹서버는 해당 파일을 찾은 후 확장자를 검사/비교한다.
3. 예를 들어, html|htm 이면 해당 파일을 그대로 브라우저로 보내준다.
4. php|php3|inc 등의 확장자라면, 웹서버에 장착되어 있는 php 엔진에 보낸다.
5. php 엔진은 php 소스를 해석하여 html 코드로 만들고 웹서버로 응답한다.
6. 웹서버는 (응답받은) html 파일을 브라우저로 보내준다.

<br>