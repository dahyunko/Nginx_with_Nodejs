# Nginx

### **nginx란?**

- Web Server
- 비동기 I/O 처리 방식 → 빠른 응답 시간 제공
- 리버스 프록시
    - 프록시 : 인터넷에 접속할 때 보안상의 문제로 직접 통신을 주고 받을 수 없을 때 그 사이에 중계기로서 대리로 통신을 수행하는 기능
        - 포워드 프록시 : 클라이언트와 인터넷 사이의 영역
            - client 정보 요청 → 프워드 프록시  → 서버 전달
            - 정보 노출 x(client IP, 도메인 , URL 접근 제한)
            - 미디어 스트리밍 지원
        - 리버스 프록시 : 인터넷과 백엔드 사이의 영역
            - 로드 밸런싱 : 요청에 따른 웹 서버 분배
            - 캐싱 서버 : 이미지 요청 시 처음에는 서버에서 가져오지만 이후에 요청 시 캐시 서버에서  가져와 client에 전달 → 속도 개선
            - SSL 제공 → HTTPS, 데이터 압축, 비동기 처리

### 과제 정리

1. **nginx 설치 및 실행**
    1. [localhost](http://localhost) 이미 window 서버가 80포트 사용 중
    → nginx의 사용 포트를 [localhost:8070](http://localhost:8070) 으로 수정함 → 실행 완료
        
        ```cpp
        //nginx.conf 포트 수정
        listen       8070;
        server_name  localhost;
        ```
        
    
2. **서버 포트 및 루트 설정**
    - **nginx 실행, 재실행**
        
        ```powershell
        //cmd 이용
        //실행
        C:\Users\82109\nginx-1.24.0\nginx-1.24.0>nginx
        //재실행
        C:\Users\82109\nginx-1.24.0\nginx-1.24.0>nginx -s reload
        ```
        
    - **redirect, rewrite, alias, try_files …**
        
        ```
        //nginx.conf 설정
        
        http {
            ## content type
            include mime.types;
        
            server{
                listen 8080;
                root "C:/Users/82109/workspace/ASAC";
        
                #rewrites
                rewrite ^/number/(\w+) /count/$1;
        
                location ~* /count/[0-9]{
                    #relocation
                    root "C:/Users/82109/workspace/ASAC";
                    try_files /index.html = 404;
                }
        
                location /fruits {
                    #append
                    root "C:/Users/82109/workspace/ASAC";
                }
        
                location /carbs {
                    #fruits 자리에 들어감
                    alias "C:/Users/82109/workspace/ASAC/fruits";
                }
        
                location /vegetables{
                    root "C:/Users/82109/workspace/ASAC";
                    #vegetables에 아래 html이 있는지 확인하고 없으면 404 에러 반환
                    try_files /vegetables/veggies.html /index.html = 404;
                }
        
                location /crops{
                    #redirect
                    #crops로 접속하면 fruits로 우회함
                    return 307 /fruits;
                }
            }
        }
        
        events {
            worker_connections 1024;
        }
        ```
        
    - **reverse proxy(Express + Nginx)**
        - express 서버의 7777포트의 페이지를 8080포트로 접근했을 시 노출
            
            ```
            //nginx.conf 설정
            
            http {
                ## content type
                include mime.types;
            
                #reverse proxy 설정
                #백엔드 upstream
            		#express 서버의 주소 7777
                upstream backend{
                    server 127.0.0.1:7777;
                }
            
                server{
                    listen 8080;
                    root "C:/Users/82109/workspace/ASAC";
            
                    #reverse proxy
                    #express서버의 7777번 포트 서버를
                    #사용자가 8080으로 접속할 시 7777번 소스를 노출 시키는 작업
            				#(express 실행해놓은 상태여야함)
                    location / {
                        proxy_pass http://backend;
                    }
                }
            }
            
            events {
                worker_connections 1024;
            }
            ```
            
    
3. **windows 이용 SSL 인증** 
    1. OpenSSL 다운
    [https://slproweb.com/products/Win32OpenSSL.html](https://slproweb.com/products/Win32OpenSSL.html)
        
        (고급 시스템 설정으로 path와 OPENSSL_CONF 넣어 줘야함)
        → 방법: [https://aspdotnet.tistory.com/2653](https://aspdotnet.tistory.com/2653)
        
    2. cmd창으로 openssl 설치 확인
        
        ```bash
        #설치 확인
        openssl version
        -> OpenSSL 3.1.1 30 May 2023 (Library: OpenSSL 3.1.1 30 May 2023)
        ```
        
    3. localhost.crt, localhost.key 발급 받기
        
        → cmd에 있는 path에 생성 되어 있음, ex) C:\Users
        
        ```bash
        #키 생성
        openssl req -new -newkey rsa:2048 -days 365 -nodes -x509 -keyout localhost.key -out localhost.crt
        
        #아래 내용 입력
        Country Name (2 letter code) [AU]:KR
        State or Province Name (full name) [Some-State]:Seoul
        Locality Name (eg, city) []:Gangnam gu
        Organization Name (eg, company) [Internet Widgits Pty Ltd]:.
        Organizational Unit Name (eg, section) []:Rnd Center
        Common Name (e.g. server FQDN or YOUR name) []:localhost
        Email Address []:example@naver.com
        ```
        
    4. conf 내용 수정 → nginx -s reload 진행
        
        ```
        //nginx.conf 설정
        
        http {
            ## content type
            include mime.types;
            
            server {
                listen 443 ssl;#ssl인증서로 https 연결
        				server_name localhost;
        				
        				#OpenSSL이용하여 개발자 모드로 인증서 받아옴
        				ssl_certificate         C:/Users/path/localhost.crt; 
        				ssl_certificate_key     C:/Users/path/localhost.key; 
        		        
        				ssl_session_cache shared:SSL:10m;
        		        ssl_session_timeout 5m;
        		
        				ssl_ciphers ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA256:ECDHE-RSA-AES256-SHA:HIGH:MEDIUM:!MD5:!aNULL:!EDH:!RC4:!DSS;
        				ssl_prefer_server_ciphers on; 
        		
                location / {
                    root "C:/Users/82109/workspace/ASAC";
                    index  index.html index.htm;
                }
            }
        }
        
        events {
            worker_connections 1024;
        }
        ```
        
        - 결과 화면
          <br>
          <img src="https://github.com/dahyunko/Nginx_with_Nodejs/assets/101400650/9420f5ff-6ac9-4afc-b321-d3769568f0ab"></img>

            
