https://laravel.kr/docs/5.8/container 글에서 필요한 부분만 발췌함.

# Laravel?
Laravel은 php 웹 프레임워크의 하나로, MVC 패턴에 따라 웹 애플리케이션을 개발하기 위해 고안됨.


# Laravel의 장점
## Artisan  
php artisan make: 라는 명령으로 모델, 컨트롤러, 데이터베이스 마이그레이션 등 다양한 파일들을 자동으로 생성 가능.
## composer 사용 가능  
php 패키지 매니저인 composer를 지원한다. 

# Laravel 설계 컨셉
## 서비스 컨테이너  
### 바인딩
#### 기본적인 바인딩
##### 간단한 바인딩
서비스 프로바이더 안에서는 항상 $this->app 속성을 통해 컨테이너 인스턴스에 접근 가능. bind 메소드를 사용하여 클래스나 인터페이스 이름에 대한 의존성을 원하는 클래스의 인스턴스를 반환하는 Closure를 등록하여 바인딩 할 수 있다.
$this->app->bind('HelpSpot\API', function ($app) {
    return new HelpSpot\API($app->make('HttpClient'));
});
