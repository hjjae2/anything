## FTP active vs passive

FTP 는 2개의 포트를 사용한다.
 - Command 포트: 주로 21번 포트를 사용하며, 연결 시에 사용되는 포트
 - Data 포트: 주로 20번 포트를 사용하며, 데이터 전송 시에 사용되는 포트

 
**Active 모드**

1. Client -> Server: 연결 시도 (21번 포트)
2. Server -> Client: OK 응답
3. Server -> Client: Data 채널 연결 요청 <--- active
4. Client -> Server: OK 응답


흔히, Client -> Server 로 연결을 시도하는 방식과 달리, Server -> Client 로 연결을 시도한다. (Active)
(20번 포트에 대한 Server의 아웃바운드 규칙, Client의 인바운드 규칙의 설정이 필요하다.)


**Passive 모드**

1. Client -> Server: 연결 시도 (21번 포트)
2. Server -> Client: OK 응답
3. Client -> Server: Data 채널 연결 요청 <--- passive
4. Server -> Client: OK 응답

20번 포트를 사용하지 않고, 잘 알려진 포트(1024번) 이후의 임의의 포트를 사용한다.
이때, 데이터 채널 포트의 범위를 지정하지 않으면 모든 포트에 대해 허용해야 하는 문제가 있다. (임의로 설정되기 때문에)
