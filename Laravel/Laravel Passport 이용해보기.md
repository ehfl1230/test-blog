# Laravel Passport?

# Laravel Passport 설치해보기
1. Oauth 서버 프로젝트를 새롭게 생성한다.
laravel new passport-server

2. 프로젝트 폴더 아래에 .env 파일을 오픈하여 DB 설정값을 변경해준다.
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=DB명
DB_USERNAME=username
DB_PASSWORD=password

3. auth scaffold 생성해준다.
php artisan make:auth

4. 컴포저를 이용하여 passport 패키지를 설치해준다.
composer require laravel/passport
