---
layout: post
title:  "Nginx Shutdown 상태 검사"
date:   2020-03-20
tags: [Blog]
---

### Nginx

Nginx는 `nginx` 명령어를 통해 켜고, `nginx -s quit` 명령어를 통해 끄게 됩니다. Upstream과 같은 로드 밸런싱 (Load Balancing) 기능을 사용하면 같은 어플리케이션을 서빙하는 여러 서버로 요청을 분산할 수 있습니다.

`nginx.conf` (혹은 내부적으로 사용하는 추가적인 호스트 설정 파일)의 upstream 헤더 안에 서버 이름을 추가/제거하는 방식으로, 정말 간단하게 로드 밸런싱을 구축할 수 있는데, 이 때문에 발생하는 문제가 있었습니다. 아주 간단한 처리를 통해 해결할 수 있는 문제지만, 이번 기회에 nginx의 on/off 메커니즘에 대해 좀 더 자세히 알아볼 수 있었습니다.

### 문제

Nginx를 웹서버로 사용하는 서버에 애플리케이션을 배포할 때, 보통 다음과 같은 절차를 거칩니다:
1. Nginx를 내린다. (upstream에서 제외해서 접속할 수 없도록 하기 위해)
2. 애플리케이션을 내린다 (e.g., Tomcat)
3. 새로운 패키지를 배포한다.
4. 애플리케이션을 켠다.
5. Nginx를 켜고, 제대로 동작하는지 모니터링한다.

여기서 Nginx를 내리는 동작에는 2가지 선택이 있는데


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
