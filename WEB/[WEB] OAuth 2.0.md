## Grant Type

|방식|사용자 인증 과정 개입|설명|
|:-:|:-:|-|
|authorization_code|O|* 인증 과정에서 Authorization Code 교환 후 access token 을 받고, 사용한다.|
|implicit|O|* 인증 과정에서 Authorization Code 의 교환 없이, 즉시 access token 을 받고, 사용한다.|
|client_credentials|X|* 인증 과정에서 client credential 정보(ex : client id, client secret)를 통해 access token 을 받고, 사용한다.|
|password <br> (Resource Owner Password Credentials, Password Credentials)|X|* 인증 과정에서 client credential 정보(ex : client id, client secret, id, password)를 통해 access token 을 받고, 사용한다.|


<br><br>

## + client_credentials

Client(= `3rd party` or `resource owner`)는 이미 토큰을 발행할 수 있는 주체이다.



> *" 대부분의 토큰 발행 유형이 Resource Owner가 시발점이 되는데 반해서, 해당 방법은 명칭에서부터 알 수 있듯이 Client가 주권을 쥐고 있습니다. 여타 방법들에서의 허가(approve)에 대한 동의를 Client가 이미 도장 쾅쾅 찍은 상태라고 보시면 됩니다. 즉, 서비스(Client)는 이미 그 자체만으로도 토큰 발행을 요청할 수 있는 주체가 되어있고 토큰을 요청하기만하면 Authorization Server는 토큰을 발행해줍니다. "*
> 
> 출처: https://blinders.tistory.com/65 [글쓰는 개발자:티스토리]