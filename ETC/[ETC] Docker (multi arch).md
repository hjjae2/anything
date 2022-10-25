### [Multi-platform images](https://docs.docker.com/build/building/multi-platform/)

도커 이미지는 여러 플랫폼(아키텍처)를 지원할 수 있다. 즉 하나의 단일 이미지는 여러 개의 아키텍처, OS 환경에서 동작할 수 있다.

(멀티 아키텍처의 이미지는) 이미지가 실행될 때, 자동적으로 맞는 OS, 아키텍처를 선택하여 구동된다.

> *Docker images can support multiple platforms, which means that a single image may contain variants for different architectures, and sometimes for different operating systems, such as Windows.*

> *When running an image with multi-platform support, docker automatically selects the image that matches your OS and architecture.*


<br>

### `docker build`

Docker 엔진은 client-server 아키텍처를 사용한다. (...중략) 이 CLI는 도커 엔진에 build 를 실행하라고 요청한다.

> *Engine uses a client-server architecture and is composed of multiple components and tools. The most common method of executing a build is by issuing a docker build command. The CLI sends the request to Docker Engine which, in turn, executes your build.*

<br>

### `docker buildx`

새로운 명령어 docker buildx 는 docker 커맨드를 확장하는 플러그인(CLI)이다.

> *The new client Docker Buildx, is a CLI plugin that extends the docker command with the full support of the features provided by BuildKit builder toolkit.*

`docker buildx build`는 `docker build`와 동일하게 사용할 수 있다. 기존의 build 기능에 추가적인 기능을 더 사용할 수 있다고 보면 된다. **이때 플랫폼(아키텍처)도 지정할 수 있다.**

> *docker buildx build provides the same user experience as docker build with many new features like creating scoped builder instances, building against multiple nodes concurrently, outputs configuration, inline build caching, and specifying target platform. In addition, Buildx also supports new features that are not yet available for regular docker build like building manifest lists, distributed caching, and exporting build results to OCI image tarballs.*

상세한 내용은 아래를 참고하자.

1. [Overview of Docker Build](https://docs.docker.com/build/)
2. [Multi-platform images](https://docs.docker.com/build/building/multi-platform/)