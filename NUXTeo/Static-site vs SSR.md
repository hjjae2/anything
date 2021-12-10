## Static-site vs SSR

### Static-site

빌드 시에 완벽한 HMTL 페이지를 생성한다.

이 페이지들은 웹서버를 통해 전달된다.

페이지의 내용을 변경하기 위해서는 재빌드 해야 한다.

**속도**, **배포**, **보안** 측면에서 장점이 있다.

**즉 빌드 시에 페이지를 완성시키고, 클라이언트의 요청이 들어오면 완성된 페이지를 전달하기만 하면 된다.**

<br>

### SSR

SSR 은 웹페이지를 브라우저에서 렌더링하지 않고 서버에서 렌더링하여 전달하는 것이다.

**콘텐츠 변경(최신 유지)** 등의 장점이 있다.

**즉 클라이언트의 요청이 들어오면, 서버 측에서 렌더링을 진행하고 전달한다.**

<br>

> Reference
> 1. https://www.smashingmagazine.com/2020/07/differences-static-generated-sites-server-side-rendered-apps/