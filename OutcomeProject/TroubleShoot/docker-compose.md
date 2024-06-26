## SpringBoot 애플리케이션과 웹 애플리케이션 포트충돌

- spring boot 애플리케이션을 이미 올려 놓은 상태에서 nginx로 설정한 web 애플리케이션과 포트번호가 맞지 않았다.
  - Nginx를 사용하여 각각 다른 경로로 트래픽을 분배하면 됩니다.
  - 예를 들어, /api로 들어오는 요청은 Spring Boot 애플리케이션으로 보내고, 나머지 요청은 웹 애플리케이션으로 보냅니다.
  ```docker-compose.yml
      version: '3'
      services:

          // 생략
      
        spring-boot-app:
          container_name: spring-boot-container
          image: outcomespring:1.0
          ports:
            - "8080:8080"
          depends_on:
            - mysql
          environment:
            SPRING_DATASOURCE_URL: jdbc:mysql://mysql:3306/outcome
            SPRING_DATASOURCE_USERNAME: iri
            SPRING_DATASOURCE_PASSWORD: ****
      
        web:
          container_name: web
          image: iri051/web-nginx
          expose:
            - "80"    // 기존 8080
      
        nginx:
          container_name: nginx
          image: nginx:latest
          ports:
            - "80:80"
          depends_on:
            - spring-boot-app
            - web
          volumes:
            - ./nginx.conf:/etc/nginx/nginx.conf

  ```
  ```nginx
  events {
    worker_connections 1024;
  }
  
  http {
    upstream spring_boot {
        server spring-boot-container:8080;
      }

    upstream web {
        server web:80;
    }

    server {
        listen 80;

        location /api/ {
            proxy_pass http://spring_boot/;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        location / {
            proxy_pass http://web/;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
  }
  ```
- docker-compose.yml:
  - spring-boot-app은 8080 포트에서 실행됩니다.
  - web은 기본적으로 80 포트에서 실행되도록 설정됩니다.
  - nginx는 두 서비스를 각각 다른 경로로 분배합니다.
- nginx.conf:
  - /api/로 들어오는 요청은 spring-boot-container의 8080 포트로 전달됩니다.
  - /로 들어오는 나머지 모든 요청은 web 컨테이너의 80 포트로 전달됩니다.
- 이 설정을 사용하면 Nginx가 리버스 프록시 역할을 하여 Spring Boot 애플리케이션과 웹 애플리케이션이 동일한 포트(예: 80 포트)에서 동시에 서비스를 제공할 수 있습니다. 
/api/ 경로는 Spring Boot 애플리케이션으로, 그 외의 경로는 웹 애플리케이션으로 라우팅됩니다.


### nginx events 추가설명
Nginx 설정 파일의 events 블록은 Nginx의 이벤트 모델과 관련된 설정을 포함합니다. 이 블록 내의 설정은 Nginx 서버가 클라이언트 요청을 처리하는 방식을 제어합니다.
```nginx
events {
    worker_connections 1024;
}
```

위 설정에서 events 블록의 worker_connections 지시문은 각 워커 프로세스가 동시에 처리할 수 있는 최대 연결 수를 설정합니다. 여기서 1024는 각 워커 프로세스가 동시에 처리할 수 있는 최대 연결 수가 1024개임을 의미합니다.

#### events 블록
- Nginx는 이벤트 기반의 비동기 서버로, 효율적인 네트워크 I/O를 처리하기 위해 이벤트 모델을 사용합니다.
- events 블록은 이러한 이벤트 모델에 대한 설정을 정의하는 곳입니다.
#### worker_connections
- worker_connections는 각 워커 프로세스가 동시에 처리할 수 있는 최대 연결 수를 설정합니다.
- 예를 들어, worker_connections 1024;로 설정하면, 각 워커 프로세스는 최대 1024개의 연결을 동시에 처리할 수 있습니다.
- 만약 Nginx가 4개의 워커 프로세스를 실행 중이라면, 전체적으로 최대 4096개의 연결을 동시에 처리할 수 있습니다 (4 * 1024).
#### 예시 설명
- worker_connections 1024;로 설정된 경우, 각 워커 프로세스는 최대 1024개의 클라이언트 연결을 처리할 수 있습니다. 이는 높은 트래픽을 처리하기 위해 중요한 설정입니다.
- 이 값은 서버의 성능, 메모리 사용량, 그리고 예상되는 트래픽에 따라 조정할 수 있습니다.</br>
</br>따라서, Nginx 서버가 높은 트래픽을 효과적으로 처리하도록 설정하기 위해 events 블록과 worker_connections 값을 적절하게 설정하는 것이 중요합니다.





