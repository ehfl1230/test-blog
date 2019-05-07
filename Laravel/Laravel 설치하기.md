CentOS7에 라라벨 설치를 해보도록 하겠습니다.
다음 글은 다음 문서를 번역하였습니다.
https://devops.profitbricks.com/tutorials/install-the-laravel-php-framework-on-centos-7/

# PHP와 Nginx 설치하기
php7을 사용할 것이기 때문에 가능한 패키지를 확득하기 위해서 EPEL과 Webtatic 저장소를 추가해줍니다.
rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
먼저 위의 두 명령어를 입력한 후, 성공하면 다음을 실행시키세요.

<code>
yum check-update
</code>

그리고 나서 다음 명령어를 실행합니다.

yum search php71

다음 두 패키지가 리턴되어야 합니다.

mod_php71w.x86_64 : PHP module for the Apache HTTP Server
php71w-fpm.x86_64 : PHP FastCGI Process Manager

우리는 웹서버로 nginx를 사용하고, php를 다루기 위해서 fpm을 사용할 것입니다. 우리는 이미 Webtatic repo를 구성했기 때문에, 우리는 업데이트된 버전의 nginx를 획득할 수 있습니다. 우리는 Laravel을 실행시키기 위해서 다음의 필요한 컴포넌트들을 설치해줍니다.

yum install nginx1w php71w-fpm php71w-pdo php71w-mbstring php71w-xml php71w-common php71w-cli

openssl과 tokenizer PHP extentions은 php71w-common에 있어 별도로 설치하지 않았습니다. php71w-cli를 사용하면 커맨드 라인에서 PHP로 작업할 수 있어 좀 더 쉽게 설정을 할 수 있습니다.

# Nginx와 PHP-FPM 설정하기
firewall-cmd --add-port=80/tcp
firewall-cmd --permanent --add-port=80/tcp

nginx를 실행시키기 위해서 80번 포트를 오픈해주어야 합니다. 위의 명령어를 입력하여 오픈해주세요.
* 최소설치로 설치하여 firewall-cmd 명령어가 없다면,  yum install firewalld로 설치후  systemctl start firewalld로 활성화 시켜주세요!

systemctl start nginx

이제 기본 설정으로 시작을 합니다.

systemctl enable nginx

만약에 서버가 부팅됐을 때 nginx가 실행되게 하고 싶다면 위의 명령어를 입력해 주세요. 

이제 서버 IP로 접속하면 nginx 페이지를 볼 수 있습니다.

systemctl start php-fpm
기본 정적 컨텐츠를 제공하기 위해 php-fpm을 실행시킵니다.  
 
systemctl enable php-fpm
만약에 서버가 부팅됐을 때 php-fpm가 실행되게 하고 싶다면 위의 명령어를 입력해 주세요.

기본적으로 php-fpm은 9000번 포트를 사용합니다. 

/etc/nginx/nginx.conf 파일에 49라인 server 블록에 아래 내용을 추가해주세요.



  location ~ [^/]\.php(/|$) {  
                fastcgi_split_path_info ^(.+?\.php)(/.*)$;  
                if (!-f $document_root$fastcgi_script_name) {  
                    return 404;  
                }
                # Mitigate https://httpoxy.org/ vulnerabilities
                fastcgi_param HTTP_PROXY "";

                fastcgi_pass 127.0.0.1:9000;
                fastcgi_index index.php;

                # include the fastcgi_param setting
                include fastcgi_params;

                # SCRIPT_FILENAME parameter is used for PHP FPM determining
                #  the script name. If it is not set in fastcgi_params file,
                # i.e. /etc/nginx/fastcgi_params or in the parent contexts,
                # please comment off following line:
                 fastcgi_param  SCRIPT_FILENAME   $document_root$fastcgi_script_name;
              (기본적으로 위의 문장은 주석처리 되어 있습니다. 주석 처리 되어 있으면 PHP output이 보이지 않습니다.)
        }


추가한 후 
systemctl restart nginx
systemctl restart php-fpm


재시작해주세요.

vi /usr/share/nginx/html/test.php

아래 파일을 열러

<html>
  <head><title>Test Vars</title></head>
  <body>
  <pre>

      <?php

        var_export($_SERVER)

      ?>

    </pre>
</body>
</html>
입력하고 
http://YOUR_SERVER_IP_ADDRESS/test.php

에 접속하면 서버 정보를 볼 수 있습니다.


# Composer와 Laravel 설치하기
composer를 설치하여 Laravel을 사용해 보겠습니다. 저는 홈디렉토리에서 아래 명령을 실행시키겠습니다. (cd ~) 아래 명령문을 실행하면 자동으로 다운로드가 되고 composer가 설치됩니다.



php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"

php -r "if (hash_file('SHA384', 'composer-setup.php') === '544e09ee996cdf60ece3804abc52599c22b1f40f4323403c44d44fdfdd586475ca9813a858088ffbc1f233e9b180f061') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"


위의 문장을 실행시켰을 때, "Installer verified"가 나와야 합니다. 만약 여기까지 되었다면, 다음을 실행시키세요.


 php composer-setup.php

위의 문장을 실행시키면 composer가 설치됩니다.


rm composer-setup.php



설치 스크립트를 삭제하세요.



php composer.phar



이제 우리는 위의 명령어로 실행시킬 수 있습니다.



편의를 위해서 우리는 composer.phar 파일을 $PATH 환경변수에 포함되어 있는 디렉토리로 복사를 하겠습니다.



 cp composer.phar /usr/bin/composer



혹은 심볼링 링크를 걸어도 됩니다.



ln -s /home/jiransoft/composer /usr/bin/composer.phar



이제 composer만 입력해도 실행이 됩니다.



이제 nginx 문서 루트로 이동하세요.



cd /usr/share/nginx/html



Laravel 프레임워크를 사용하여 새 testapp 디렉토리에 설치하려면 다음을 실행하세요.

composer create-project laravel/laravel testapp

위의 명령어로 프로젝트를 생성할 수 있습니다. 위의 명령어가 완료되면 nginx 문서를 수정해야합니다.

 vi /etc/nginx/nginx.conf

nginx..conf문서를 열어 52라인쯤에 있는

root    /usr/share/nginx/html;
를 
root    /usr/share/nginx/html/testapp/public;
으로 변경시켜 주세요.

이제 nginx를 재시작합니다.
nginx -t
systemctl restart nginx

# Ownership과 SELinex 오류 해결하기

SELinex는 CentOS7 이미지에 기본적으로 활성화 되어 있습니다. 
다음을 실행함으로써, 활성화 되어 있는지를 확인할 수 있습니다.

sestatus
만약 
SELinux status: enabled, Current mode: enforcing이면 다음 단계를 따라야 합니다. 만약 SELiux가 이미 disabled 되어 있거나 모드가 permissive면 스킵해도 좋습니다. 

http://SERVER_IP/index.php로 접속했을 때, SELinux가 활성화 되어 있다면 UnexpectedValueException 에러가 나올 겁니다.
이 오류는 Laravel이 기록해야 하는 몇 개 의 디렉토리에서 SELinux가 실행하는 보안 컨텍스트와 함께 디렉토리 및 파일 소유권으로 인해 발할 수 있습니다. 다음 명령을 실행하여 이 문제를 해결하세요.

chown -R apache:root /usr/share/nginx/html/testapp/storage/*
chown -R apache:root /usr/share/nginx/html/testapp/bootstrap/cache

이렇게 하면 기본적으로 실행중인 apache 사용자인 디렉토리의 소유권이 php-fpm으로 변경됩니다.
SELinux 문제를 해결하려문 보안 컨텍스트를 httpd_sys_content_t에서 httpd_sys_rw_content_t로 업데이트해야 합니다. semange 명령어가 없다면 yum install policycoreutils-python 명령어로 다운로드 받으세요. 

semanage fcontext -a -t httpd_sys_rw_content_t '/usr/share/nginx/html/testapp/bootstrap/cache(/.*)?'
semanage fcontext -a -t httpd_sys_rw_content_t '/usr/share/nginx/html/testapp/storage(/.*)?'

변경사항을 적용하려면 다음을 실행하세요.
restorecon -Rv '/usr/share/nginx/html/testapp'

마지막으로 Laravel의 기본 화면이 나온다면 완료입니다.
