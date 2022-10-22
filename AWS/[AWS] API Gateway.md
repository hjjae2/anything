### What is Amazon API Gateway?

> Amazon API Gateway is an AWS service for creating, publishing, maintaining, monitoring, and securing REST, HTTP, and WebSocket APIs at any scale.
>
> **API developers can create APIs that access AWS or other web services, as well as data stored in the AWS Cloud.** 
> 
> As an API Gateway API developer, you can create APIs for use in your own client applications. 
> Or you can make your APIs available to third-party app developers.

<br><br>

### Custom Domain Names

API Gateway 는 기본적으로 아래 형태와 같은 HOSTNAME 을 갖는다.
```sh
https://api-id.execute-api.region.amazonaws.com/stage

# https://ab123cdef4.execute-api.ap-northeast-2.amazonaws.com/my-stage/my-api-path
```

`Custom Domain Names` 를 사용할 수 있다.

물론 정식으로 등록된, 소유하고 있는 domain 만 사용할 수 있다.

> *You must have a registered internet domain name in order to set up custom domain names for your APIs. If needed, you can register an internet domain using Amazon Route 53 or using a third-party domain registrar of your choice. An API's custom domain name can be the name of a subdomain or the root domain (also known as "zone apex") of a registered internet domain.*
> 
> *After a custom domain name is created in API Gateway, you must create or update your DNS provider's resource record to map to your API endpoint. Without such a mapping, API requests bound for the custom domain name cannot reach API Gateway.*

> *With custom domain names, you can set up your API's hostname, and choose a base path (for example, myservice) to map the alternative URL to your API. For example, a more user-friendly API base URL can become: `https://api.example.com/myservice`*
>
> *If you don't set any base mapping under a custom domain name, the resulting API's base URL is the same as the custom domain (for example, https://api.example.com). In this case, the custom domain name can't support more than one API.*

<br><br>

### With AWS Lambda

> https://docs.aws.amazon.com/lambda/latest/dg/services-apigateway.html

AWS Lambda Function을 호출하는 형태로 사용할 수도 있다.

> *You can create a web API with an HTTP endpoint for your Lambda function by using Amazon API Gateway.*

### 참고

- https://docs.aws.amazon.com/apigateway/latest/developerguide/welcome.html
- https://docs.aws.amazon.com/apigateway/latest/developerguide/how-to-custom-domains.html
- https://docs.aws.amazon.com/lambda/latest/dg/services-apigateway.html