---
layout: post
title:  "Nginx Upstream과 Shutdown 상태 검사"
date:   2020-03-20
tags: [Blog]
---

### Nginx

Nginx는 `nginx` 명령어를 통해 켜고, `nginx -s quit` 명령어를 통해 끄게 됩니다. Upstream과 같은 로드 밸런싱 (Load Balancing) 기능을 사용하면 한 서버에서 다른 여러 서버로 요청을 분산할 수 있습니다.

예를 들어, 같은 어플리케이션을 서빙하는 3대의 서버가 있다고 가정하면, nginx 설정 파일에 다음과 같은 처리를 해주면 간단한 로드 밸런싱이 처리됩니다:

```
http {
    upstream myapp1 {
        server srv1.example.com;
        server srv2.example.com;
        server srv3.example.com;
    }

    server {
        listen 80;

        location / {
            proxy_pass http://myapp1;
        }
    }
}
```

`myapp1`이라는 공통된 어플리케이션을 서빙하는 서버 3대 (`srv1`, `srv2`, `srv3`)에 기본으로 라운드 로빈 (round robin) 방식의 로드 밸런싱이 적용되고,



- `${NGINX} -t` 를 통해 nginx 상태 체크 가능
    - 정상으로 구동되고 있는 경우:

    ```
    nginx: the configuration file /home1/irteam/apps/nginx/conf/nginx.conf syntax is ok
    nginx: configuration file /home1/irteam/apps/nginx/conf/nginx.conf test is successful
    ```

    - 하지만 이건 nginx.conf가 정상인지 테스트하는 용도일 뿐, nginx가 제대로 shutdown 되었는지, nginx 프로세스가 아직 떠있는지 체크하지 못함.

- 조사를 해보니 `nginx.pid`라는 파일이 있는데 nginx가 켜져있을 때만 nginx의 master process pid를 담으며, nginx가 꺼지면 이 파일도 삭제되도록 동작함.
    - MacOS에서는 Homebrew를 통해 설치하였기 때문에 default path인 `/usr/local/var/run`에 `nginx.pid`가 존재함. (일반 Linux에는 `/var/run`에 존재하는 것으로 보이는데 정확한 위치는 `sudo find / -name "nginx.pid"`로 찾아보면 됨)

    ```console
    $ cat /usr/local/var/run/nginx.pid
    75097
    $ ps -ef | grep nginx
      501 75097     1   0 10 320  ??         0:00.00 nginx: master process nginx
      501 75098 75097   0 10 320  ??         0:00.02 nginx: worker process
      501 85164 15709   0  9:28PM ttys003    0:00.00 grep nginx
    ```

- 인프라에서 제공하는 nginx는 커스텀이라 ${NGINX}/logs 디렉토리에 `nginx.pid` 파일이 있고, 이 파일의 유무를 체크하여 nginx가 제대로 꺼졌는지 체크하면 될 것 같음.

```
NGINX_PID=/home1/irteam/apps/nginx/logs/nginx.pid
if [ -e ${NGINX_PID ]; then
    echo "Nginx seems to be still alive. Exiting..."
    exit 1;
fi
```

### Reference

1. [StackOverflow: Understanding Spring @Autowired usage][1]
2. [Spring RequestParam][2]

[1]:https://stackoverflow.com/a/19419296
[2]:https://www.baeldung.com/spring-request-param
