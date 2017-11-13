## 배포하기

- #### Refer

  - [Nginx, uWSGI 배포-WIKIDOCS](https://wikidocs.net/6611)
  - [배포하기-digital ocean](https://www.digitalocean.com/community/tutorials/how-to-serve-django-applications-with-uwsgi-and-nginx-on-ubuntu-16-04)
  - [Nginx 보안(우분투에서 암호화하기)](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-16-04)
  - [리버스 프록시란?](https://ko.wikipedia.org/wiki/%EB%A6%AC%EB%B2%84%EC%8A%A4_%ED%94%84%EB%A1%9D%EC%8B%9C)

- #### 시스템 환경

  - ##### Ubuntu 16.04

  - ##### Django 1.11

  - ##### nginx

  - ##### uwsgi

- #### Step

  1. ##### 기본 스크립트 설정 및 가상환경 준비 

     ```
     # 기본적인 명령어 alias 등록
     $ echo "alias python='python3'" >> ~/.bashrc
     $ echo "alias pip='pip3'" >> ~/.bashrc
     $ source ~/.bashrc # 쉘 스크립트 적용

     # 파이썬 가상환경 패키지 설치
     $ sudo apt-get update
     $ sudo apt-get install python3-pip
     $ sudo apt-get install python3-venv
     $ sudo pip3 install --upgrate pip

     # 가상환경 준비
     $ python -m venv [venv_name]
     ```

  2. ##### 장고 프로젝트 준비

     - python package 설치, 셋팅 파일 분기 등

  3. ##### uWSGI 설치

     ```
     # uWSGI 설치
     $ sudo apt-get install python3-dev
     $ pip install uwsgi

     # uWSGI 서버 테스트
     $ uwsgi --http :8080 --home /home/[user]/[env] --chdir /home/[user]/[project] -w [project].wsgi
     ```

  4. ##### uWSGI 옵션 파일(.ini) 추가 

     ```
     $ sudo mkdir -p /etc/uwsgi/sites
     $ sudo vi /etc/uwsgi/sites/[project].ini

     # dir: /etc/uwsgi/sites/[project].ini -> 한 서버에 여러 Django 앱을 둘 경우 
     # dir: /home/[user]/run/uwsgi/[project].ini -> 하나만 있을 경우 run 폴더에 바로 설정해도 무관 

     [uwsgi]
     project = firstsite
     uid = sammy
     base = /home/%(uid)

     chdir = %(base)/%(project)
     home = %(base)/Env/%(project)
     module = %(project).wsgi:application

     master = true
     processes = 5

     socket = /run/uwsgi/%(project).sock
     chown-socket = %(uid):www-data
     chmod-socket = 660
     vacuum = true
     ```

  5. ##### uWSGI에 대한 서비스 스크립트 생성

     `--emperor` 옵션으로 `uwsgi.ini` 파일이 들어있는 디렉토리를 지정한다.

     ```
     $ sudo vi /etc/systemd/system/uwsgi.service

     [Unit]
     Description=uWSGI Emperor service

     [Service]
     ExecStartPre=/bin/bash -c 'mkdir -p /run/uwsgi; chown sammy:www-data /run/uwsgi'
     ExecStart=/home/[user]/[venv]/bin/uwsgi --emperor /etc/uwsgi/sites
     Restart=always
     KillSignal=SIGQUIT
     Type=notify
     NotifyAccess=all

     [Install]
     WantedBy=multi-user.target
     ```

  6. ##### uWSGI 서비스 등록 및 구동 확인

     ```
     $ sudo systemctl start uwsgi
     $ sudo systemctl enable uwsgi
     $ systemctl status uwsgi
     ```

  7. ##### nginx 설정

     1. nginx를 socket 서버로 설치하고 구성한다.

     2. Server-block 설정 파일 추가

        ```
        $ sudo apt-get install nginx
        $ sudo vi /etc/nginx/sites-available/[project]

        server {
            listen 80;
            server_name firstsite.com www.firstsite.com; # 서버 도메인 or ip 추가 

            location = /favicon.ico { access_log off; log_not_found off; }
            location /static/ {
                root /home/[user]/[project];
            }

            location / {
                include         uwsgi_params;
                uwsgi_pass      unix:/run/uwsgi/firstsite.sock;
            }
        }
        ```

     3. 설정 파일 심볼릭 링크 연결

        ```
        $ sudo ln -s /etc/nginx/sites-available/[project] /etc/nginx/sites-enabled
        ```

     4. 구문 오류 체크 후 nginx 재시작

        ```
        $ sudo nginx -t
        $ sudo systemctl restart nginx
        $ sudo systemctl enable nginx
        ```

     5. 방화벽 액세스 허용

        ```
        $ sudo ufw allow 'Nginx Full'
        ```

  8. ##### SSL/ TLS를 사용 트래픽 보호하기.

- #### Error log Check

  - syslog 파일 위치: /var/log/syslog
  - $ sudo systemctl status uwsgi
  - $ sudo tail -F /var/log/nginx/error.log
  - $ sudo journalctl -u uwsgi

- #### Trouble shotting

  - ##### systemctl error시

    - $ sudo systemctl status uwsgi 로 에러 확인 후 fix

    ```
    Failed to start uwsgi.service: Unit uwsgi.service is not loaded properly: Invalid argument.
    See system logs and 'systemctl status uwsgi.service' for details.
    ```

  - ##### 502 bad gateway error시

    - $ sudo tail -F /var/log/nginx/error.log 로 에러 확인

    ```
    connect() to unix:/run/uwsgi/firstsite.sock failed (2: No such file or directory)
    ```

- #### 알아야 할 것!