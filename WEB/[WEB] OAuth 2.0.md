## Grant Type

|방식|사용자 인증 과정 개입|설명|
|:-:|:-:|-|
|authorization_code|O|* 클라이언트 = 3rd party (웹 애플리케이션) <br><br> * 사용자 인증 후 Authorization Code 교환 후 access token 을 받고, 사용한다.|
|implicit|O|* 클라이언트 = 3rd party (웹 애플리케이션) <br><br> * 사용자 인증 후 Authorization Code 의 교환 없이, 즉시 access token 을 받고, 사용한다.|
|password|X|* 클라이언트 = 3rd party (웹 애플리케이션) <br><br> * 3rd party 가 사용자의 credential 정보(id, password)를 가지고 있다. 이 정보를 통해 access token 을 받고, 사용한다.|
|client_credentials|X|* 클라이언트 = 사용자(resource owner) <br><br> * 사용자가 credential 정보(ex : client id, client secret 혹은 id, password 등)를 통해 access token 을 받고, 사용한다.|
