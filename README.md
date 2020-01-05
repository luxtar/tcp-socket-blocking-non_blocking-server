# tcp-socket-blocking-non_blocking-server

TCP 소켓의 Blocking 모드는 소켓이 블록 되는 것을 의미한다.  
소켓이 블록 상태이면 블록이 풀리기 전까진 다음 처리를 진행할 수 없다.  
애플리케이션이 싱글스레드 모델이라면 블록 상태에서 문제가 생겼을 경우 다음 처리를 진행할 수 없기 때문에 문제가 될 수 있다.

Non-Blocking 모드는 Blocking 소켓의 단점을 보완하기 위해 등장한 개념으로 소켓이 블록 되지 않고 즉시 소켓의 상태를 반환하게 된다.  
이렇게 되면 소켓의 상태를 주기적으로 체크하고 다른 처리를 진행할 수 있게 된다.

기본적으로 소켓은 블로킹 모드로 동작하는데 fcntl 함수를 이용하면 지정한 fd를 Non-Blocking 소켓으로 변경할 수 있다.  
다음은 fcntl로 소켓 fd를 Non-Blocking 모드로 설정하는 방법이다.

```
#include <unistd.h>
#include <fcntl.h>

int flag = fcntl(sock_fd, F_GETFL, 0);
fcntl(sock_fd, F_SETFL, flag | O_NONBLOCK);
```

이해를 돕기 위해 블로킹 모드 별 서버를 만들어보았다.

소스 :

### Blocking 소켓 서버

앞서 설명했듯이 소켓은 기본적으로 블로킹 모드로 동작한다고 했다.  
블로킹 소켓 서버는 fcntl를 사용하지 않는다.  
예제 소스는 클라이언트를 accept 하고 데이터를 read 한 후에 other\_routine을 실행하는 서버이다.

**출력 결과**

```
----- accept wait
----- accept client

----- read wait
----- read : A
----- Other routine processing

----- read wait
----- read : B
----- Other routine processing

----- read wait
----- read : C
----- Other routine processing

----- read wait
```

read에서 블록이 되므로 데이터를 수신했을 경우에만 other\_routine이 실행되는 결과를 볼 수 있다.

### Non-Blocking 소켓 서버

논 블로킹 소켓 서버에는 클라이언트 소켓만 논 블로킹 모드로 설정했다.  
따라서 accept는 블로킹 모드로 동작하고 데이터를 read 하는 부분만 논 블로킹 모드로 동작한다.

**출력 결과**

```
----- accept wait
----- accept client

----- read wait
----- read EAGAIN
----- Other rutine processing...

----- read wait
----- read EAGAIN
----- Other rutine processing...

----- read wait
----- read EAGAIN
----- Other rutine processing...

----- read wait
----- read EAGAIN
----- Other rutine processing...

----- read wait
----- read : A
----- Other rutine processing...

----- read wait
----- read EAGAIN
----- Other rutine processing...

----- read wait
----- read EAGAIN
----- Other rutine processing...

----- read wait
----- read EAGAIN
----- Other rutine processing...

----- read wait
----- read EAGAIN
----- Other rutine processing...

----- read wait
----- read : B
----- Other rutine processing...

----- read wait
----- read EAGAIN
----- Other rutine processing...

----- read wait
----- read EAGAIN
----- Other rutine processing...

----- read wait
----- read EAGAIN
----- Other rutine processing...

----- read wait
----- read EAGAIN
----- Other rutine processing...

----- read wait
----- read : C
----- Other rutine processing...

----- read wait
```

read에서 블록이 되지 않으므로 데이터를 수신 여부와 상관없이 other\_routine이 계속해서 실행되는 결과를 볼 수 있다.
