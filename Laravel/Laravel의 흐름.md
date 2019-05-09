HTTP 라우팅
-> HTTP 컨트롤러

# HTTP 라우팅
## 기본적인 라우팅
App\Providers\RouteServiceProvider 클래스에 의해 로드되는 app/Http/routes.php
###기본 형식 
Route::method호출방식('경로', function() {return 값});
###다수의 HTTP 메소드에 대응하는 라우트 등록 Route 파사드의 match 메소드 사용
Route::match(['get', 'post'], '경로', function() {return 값});
###전체 메소드에 대응하기
Route::any('경로', function() {return 값});
###라우트에 대응하는 URL 생성하기 
$url=url('foo');

## 라우트 파라미터
### 필수 파라미터
Route::get('user/{id}', function($id){ return 값});
* 라우트 파라미터는 중괄호로 쌓여 있어야 함.
## 선택적 파라미터

# HTTP 미들웨어 
HTTP 미들웨어는 애플리케이션으로 들어온 HTTP 요청을 간편하게 필터링 할 수 있는 방법을 제공함.
라라벨 프레임워크에는 보수(maintenance), 인증(authentication), CSRF 보안 등을 위한 미들웨어들이 포함되어 있음. 이러한 미들웨어들은 모두 app/Http/Middleware 디렉토리 안에 위치하고 있습니다.

## 미들웨어 정의하기
php artisan make:middleware OldMiddleware
-> app/Http/Middleware 디렉토리에 들어감.

## Before / After 미들웨어 
### Before
<?php

namespace App\Http\Middleware;

use Closure;

class BeforeMiddleware
{
    public function handle($request, Closure $next)
    {
        // Perform action

        return $next($request);
    }
}

### After
<?php

namespace App\Http\Middleware;

use Closure;

class AfterMiddleware
{
    public function handle($request, Closure $next)
    {
        $response = $next($request);

        // Perform action

        return $response;
    }
}

## 미들웨어 등록하기
### 글로벌-전역 미들웨어 
모든 HTTP 요청에 대하여 미들웨어가 작동되어야 한다면, app/Http/Kernel.php 클래스의 $middleeware 프로퍼티에 미들웨어를 등록하면됨.

### 라우트에 미들웨어 지정하기
1. app/Http/Kernel.php 파일에 미들웨어의 키를 지정한다. 
protected $routeMiddleware = [
    'auth' => \App\Http\Middleware\Authenticate::class,
    'auth.basic' => \Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
    'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,
];
2. 라우트의 옵션 배열에서 middleware 키 지정
Route::get('admin/profile', ['middleware' => 'auth', function () {
    //
}]);

# HTTP 컨트롤러
애플리케이션의 요청에 대한 모든 처리 로직을 routes.php 파일 하나에 정의하는 것보다 별도의 컨트롤러 클래스를 통해 구성할 수 있음.
컨트롤러는 일반적으로 app/Http/Controllers 디렉토리에 저장한다.

## 기본적인 컨트롤러 
모든 컨트롤러는 base 컨트롤러를 상속받아야 함.
<?php  
namespace App\Http\Controllers;  

use App\User;  
use App\Http\Controllers\Controller;  

class UserController extends Controller  
{  
    /**  
     * Show the profile for the given user.  
     *  
     * @param  int  $id  
     * @return Response  
     */  
    public function showProfile($id)  
    {  
        return view('user.profile', ['user' => User::findOrFail($id)]);  
    }  
}  
다음과 같이 라우트를 지정할 수 있다.  
Route::get('user/{id}', 'UserController@showProfile');

## 컨트롤러 미들웨어 
특정 컨트롤러에서 미들웨어를 지정하고 싶으면 컨트롤러의 생성자에서 middleware 메소드를 사용하면 된다.
public function __construct()
    {
        $this->middleware('auth');

        $this->middleware('log', ['only' => ['fooAction', 'barAction']]);

        $this->middleware('subscribed', ['except' => ['fooAction', 'barAction']]);
    }
    
## RESTful 리소스 컨트롤러 
php artisan make:controller PhotoController'
Route::resource('photo', 'PhotoController');를 등록하면 됨.

## 묵시적 컨트롤러


public/index.php 파일 시작점. index.php 파일은 컴포저가 생성한 오토로더 정의를 로딩함. 
그리고 bootstrap/app.php 스크립트에서 라라벨애플리케이션의 인스턴스를 가져옴.
라라벨 자신의 첫 번째 동작은 서비스 컨테이너 인스턴스를 생성함.

routes/web.php에서 url 규칙에 맞게   
