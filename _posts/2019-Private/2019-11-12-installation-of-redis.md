---
layout: post
title: "[DB]  Redis의 설치"
date: 2019-11-12
excerpt: "Redis를 설치해본다."
tag:
- database
comments: true
---

# Redis의 설치

## Redis란?

![redis logo](https://www.drupal.org/files/styles/grid-3-2x/public/project-images/redis-logo.png?itok=glSfTKCn)

Redis(REmote DIctionary System)는 BSD 라이선스를 기반으로 하는 오픈소스로 인 메모리(In-memory) 기반이다. 이는 다양한 자료 구조를 지원하면서 동시에 인메모리 방식으로 데이터베이스, 캐시 또는 메시지 전달자로 사용된다[^1]. 이러한 특징으로 인해서 Redis는 인스타그램, 네이버 재팬의 LINE 메신저 서비스, 스택 오버플로우(StackOverflow), 블리자드 등 여러 소셜 서비스에서 널리 사용되고 있다[^2].

## Redis의 설치

Redis를 사용하기 위해 package 매니저로 다운을 받는 방법도 있지만, 본 글에서는 소스 코드를 통해서 받는 방법을 설명하도록 한다. 만약 본인이 WSL(Windows Subsystem for Linux)에서 redis를 돌리고자 한다면 [이 글]( https://medium.com/@RedisLabs/windows-subsystem-for-linux-wsl-10e3ca4d434e )을 참고 해주길 바란다.

```bash
wget http://download.redis.io/redis-stable.tar.gz
tar xvzf redis-stable.tar.gz
```

여기까지 했다면, 이제는 빌드를 할 차례이다.

```bash
sudo apt install tcl tk
cd redis-stable
make
```

만약 문제없이 빌드가 되었다면, 빌드 내용에 대해서 테스트를 진행하도록 한다.

```bash
make test
```

테스트가 완료되었다고 뜨면 아래의 일을 수행하도록 한다.

```bash
sudo make install
```

그리고 `redis-server`를 통해 실행시키고 `redis-cli`를 통해서 `redis`에 정상적으로 접근할 수 있는지를 확인하도록 한다.

# Redis 모듈 설치

## Python 에서의 Redis

Python에서 redis를 실행시키기 위해서는 먼저 `pip install redis`를 통해서 Python용 redis 모듈을 설치하도록 한다.

```bash
sudo pip3 install redis
```

이 다음에 python을 실행시키고 간단한 프로그램을 돌려보도록 한다[^3].(이름을 `redis.py`로 하지 않도록 한다. 그렇게 하는 경우에는 `import redis`가 재귀적으로 파일을 호출하여 `StrictRedis`를 찾을 수 없다고 뜰 것이다[^4].)

```python
# filename: redis-sample.py
import redis

try:
    conn_info = {
        'host': 'localhost',
        'port': 6379,
        'db': 0,
    }
    conn = redis.StrictRedis(
        host = conn_info['host'],
    	port = conn_info['port'],
    	db = conn_info['db'])
    print('set result: {}'.format(conn.set('value', 1)))
    print('get result: {}'.format(conn.get('value')))
    print('delete result: {}'.format(conn.delete('value')))
    print('get result: {}'.format(conn.get('value')))
except Exception as e:
    print('Error: {}'.format(e))
```

이 다음에 `python3 redis-sample.py`를 수행하면 다음과 같은 결과를 얻을 수 있다.

```bash
set result: True
get result: b'1'
delete result: 1
get result: None
```



## C/C++ 에서의 Redis

Redis를 사용하는 데에는 무수히 많은 라이브러리들이 존재한다. 하지만 C++의 경우에는 추천하는 redis 라이브러리가 없는 것을 알 수 있다[^5]. 그래서 C++에서 redis를 개발할 때에는 C 라이브러리인 [hiredis](https://github.com/redis/hiredis )를 사용하는 것을 추천한다.

hiredis의 설치 방법은 간단하다. 먼저 Redis 설치 방법과 동일하게 웹에서 소스 코드를 다운 받도록 한다.

```bash
wget https://github.com/redis/hiredis/archive/v0.14.0.tar.gz
tar -xvzf v0.14.0.tar.gz
cd hiredis-0.14.0
```

다운 받은 후에는 빌드를 하고 설치를 진행한다.

```bash
sudo make && sudo make install
```

설치가 완료되었으면 아래와 같은 코드를 작성한다[^6][^7].

```c
// filename: redis-sample.c
#include <stdio.h>
#include <hiredis/hiredis.h>

int main(void) {
        const char *host = "localhost";
        const int port = 6379;
        struct timeval timeout = {1, 500000}; // 1.5 seconds

        redisReply *reply;
        redisContext *conn = redisConnectWithTimeout(host, port, timeout);

        reply = redisCommand(conn, "SET %s %d", "value", 1);
        printf("set result: %s\n", reply->str);
        freeReplyObject(reply);

        reply = redisCommand(conn, "GET %s", "value");
        printf("set result: %s\n", reply->str);
        freeReplyObject(reply);

        reply = redisCommand(conn, "DEL %s", "value");
        printf("set result: %s\n", reply->str);
        freeReplyObject(reply);

        reply = redisCommand(conn, "GET %s", "value");
        printf("set result: %s\n", reply->str);
        freeReplyObject(reply);

        redisFree(conn);
        return 0;
}
```

코드를 작성을 했다면, 다음 명령을 통해서 컴파일하고 실행해보도록 한다. 만약에 `<hiredis/hiredis.h>`와 같이 사용하지 않고, `<hiredis.h>`와 같이 사용하고자 한다면 `-I/usr/local/include/hiredis`를 인자 값으로 추가해주도록 한다.

```bash
gcc redis-sample.c -L/usr/local/lib -lhiredis
./a.out
```

실행을 하면 다음과 같은 결과가 나오게 된다.

```bash
set result: OK
set result: 1
set result: (null)
set result: (null)
```

앞서 짠 python의 코드를 동일하게 수행하는 코드를 작성한 것이다.

# 결론

이러한 방식을 통하여 **Redis**와 그에 연관된 라이브러리를 사용할 수 있는 것을 알 수 있다.

참고 자료
---

[^1]: https://redis.io/
[^2]:  https://www.kdata.or.kr/info/info_04_view.html?field=&keyword=&type=techreport&page=38&dbnum=176019&mode=detail&type=techreport 
[^3]:  https://bstar36.tistory.com/351 
[^4]: https://github.com/andymccurdy/redis-py/issues/533
[^5]:  https://redis.io/clients 
[^6]:  https://github.com/redis/hiredis/blob/master/examples/example.c 
[^7]:  http://yular.github.io/2017/01/28/C-Redis-QuickStart/ 