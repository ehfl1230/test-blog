https://medium.com/@jsdecena/simple-tdd-in-laravel-with-11-steps-c475f8b1b214

1. 먼저 테스트 환경을 설정해봅시다.
루트 디렉토리에서 phpunit.xml 파일을 오픈하여 다음 내용을 추가합니다.
<php></php>내에 환경변수에 추가해주면 됩니다.

```
<env name="DB_CONNECTION" value="sqlite"/>
<env name="DB_DATABASE" value=":memory:"/>
<env name="API_DEBUG" value="false"/>
<ini name="memory_limit" value="512M" />
```
우리는 테스트를 빨리 할 수 있도록 인메모리에서 테스트를 하겠습니다. sqlite와 :memory: 를 데이터베이스로 사용하겠습니다. 
우선 테스트만 할 것이므로 디버그는 false로 설정하겠습니다. 메모리 제한을 높이는 것은 추후에 하도록 합니다.

```
<?php
namespace Tests;
use Illuminate\Foundation\Testing\DatabaseMigrations;
use Illuminate\Foundation\Testing\DatabaseTransactions;
use Illuminate\Foundation\Testing\TestCase as BaseTestCase;
use Faker\Factory as Faker;
/**
 * Class TestCase
 * @package Tests
 * @runTestsInSeparateProcesses
 * @preserveGlobalState disabled
 */
abstract class TestCase extends BaseTestCase
{
    use CreatesApplication, DatabaseMigrations, DatabaseTransactions;
    protected $faker;
    /**
     * Set up the test
     */
    public function setUp()
    {
        parent::setUp();
        $this->faker = Faker::create();
    }
    /**
     * Reset the migrations
     */
    public function tearDown()
    {
        $this->artisan('migrate:reset');
        parent::tearDown();
    }
}
```
DatabaseMigrations을 추가해야 하므로 테스트를 실행할 때마다 마이그레이션 파일도 실행됩니다. 
또한 테스트 중에 응용 프로그램의 사이클을 완료하는 데 필요한 setUp() 및 tearDown() 메서드가 필요합니다.

2. 실제 테스트 작성하기 
PHPUNIT이 특정 함수가 테스트 함수라는 것을 알게 하기 위해서 /** @test */ 어노테이션을 docblock에 추가하거나 test_를 prefix로 붙여줍니다. 
```
<?php
namespace Tests\Unit;
use Tests\TestCase;
class ArticleApiUnitTest extends TestCase
{
  public function it_can_create_an_article()
  {
      $data = [
        'title' => $this->faker->sentence,
        'content' => $this->faker->paragraph
      ];
    
      $this->post(route('articles.store'), $data)
        ->assertStatus(201)
        ->assertJson($data);
  }
}
```
이 테스트에서는 article을 만드는 것을 체크하고 있습니다.
우리는 status 201과 JSON 형식으로 데이터가 리턴되는지 확인할 수 있습니다.
첫번째 테스트를 생성하고 나서 phpunit 혹은 vendor/bin/phpunit을 실행해 봅시다.
vendor/bin/phpunit: 그런 파일이나 디렉터리가 없습니다
라는 오류가 발생한다면 composer require --dev symfony/phpunit-bridge 명령어로 패키지를 설치해줍니다.

phpunit을 실행시키면 테스트는 당연히 실패할 겁니다. 
하지만 테스트를 생성한 다음에는 실패해야 한다는 TDD의 룰을 잘 지키고 있습니다.

그렇다면 왜 실패할까요? 테스트에서는 status 201이 반환될 것을 예상했는데, 404가 왔습니다. 왜일까요?
URL이 구성되지 않았기 때문입니다. [POST] /api/v1/articles 가 존재하지 않기 때문에 404오류가 발생하는 것입니다. 

3. route 파일에 url을 추가해봅시다.
/routes/api.php 파일을 찾아 URL을 생성해봅시다. api 파일이 url을 추가하면 자동으로 /api prefix가 됩니다.
```
<?php
use App\Http\Controllers\Api\ArticlesApiController;
use Illuminate\Support\Facades\Route;
Route::group(['prefix' => 'v1'], function () {
  Route::resource('articles', ArticlesApiController::class);
});
```

그리고 다음 명령어도 수행합니다.
php artisan make:controller ArticlesApiController —-resource
POST 요청은 ArticlesApiController의 store() 메소드로 연결될 겁니다.

4. Controller를 디버깅해봅시다.
<?php
namespace App\Http\Controllers\Api;
class ArticlesApiController extends Controller 
{
    public function store() {
        dd('success!');
    }
}
이제 phpunit을 다시 호출하여 결과를 살펴봅시다.
success!라는 문자열이 터미널에 찍히는 것을 볼 수 있을 겁니다!


5. input 검증하기
DB에 저장할 값을 검증는 것도 잊지마세요! 

