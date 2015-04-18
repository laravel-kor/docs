# 오류 & 로깅

- [설정](#configuration)
- [오류 처리](#handling-errors)
- [HTTP 예외](#http-exceptions)
- [로깅](#logging)

<a name="configuration"></a>
## 설정

어플리케이션의 로깅 기능은 `Illuminate\Foundation\Bootstrap\ConfigureLogging` 부트스트래퍼 클래스에 설정되어 있습니다. 이 클래스는 여러분의 `config/app.php` 구성 파일의 `log` 구성 옵션을 사용합니다.

기본적으로, 로거는 일별 로그파일을 사용하도록 설정되어 있습니다; 하지만, 이 동작을 필요에 따라 변경 할 수 있습니다. 라라벨은 유명한 [Monolog](https://github.com/Seldaek/monolog) 로깅 라이브러리를 사용하므로, 여러분은 Monolog가 제공하는 다양한 핸들러를 활용 할 수 있습니다.

예를 들어, 일별 로그 파일대신 하나의 로그 파일을 사용하려면, `config/app.php` 설정 파일에서 다음을 변경 할 수 있습니다:

    'log' => 'single'

기본적으로, 라라벨은 `single`, `daily`, `syslog` 그리고 `errorlog` 로깅 모드를 제공합니다. 하지만, `ConfigureLogging` 부트스트래퍼 클래스를 재정의하여 로깅을 사용자 정의 할 수 있습니다.

### 오류 정보

여러분의 어플리케이션이 브라우저를 통해 나타내는 오류의 결과는 `config/app.php` 설정 파일의 `app.debug` 설정 옵션을 통해 제어됩니다. 기본으로, 이 설정 옵션은 `.env` 파일에 있는 `APP_DEBUG` 환경 변수를 준수하도록 되어있습니다.

로컬 개발에서는, `APP_DEBUG` 환경 변수를 `true`로 설정해야 합니다. **여러분의 프로덕션 환경에서는, 이 값이 항상 `false`여야 합니다.**

<a name="handling-errors"></a>
## 오류 처리

모든 예외는 `App\Exceptions\Handler` 클래스에서 처리 됩니다. 이 클래스는 다음의 두가지 메서드를 포함하고 있습니다: `report`, `render`.

`report` 메서드는 예외를 로그하거나 [BugSnag](https://bugsnag.com)같은 외부 서비스로 전송하는데 사용됩니다. 기본적으로, `report` 메소드는 간단히 예외가 로그되는 부모 클래스의 기본 구현 클래스로 예외를 전달합니다. 하지만, 여러분은 원하는대로 예외를 로그 할 수 있습니다. 만약 다른 형식의 예외를 다른 방법으로 보고해야 한다면, PHP의 `instanceof` 비교 연산자를 사용 할 수 있습니다:

    /**
     * 예외를 보고하거나 로그합니다
     *
     * 이곳은 예외를 Sentry, Bugsnag 등으로 전송하는 좋은 위치입니다
     *
     * @param  \Exception  $e
     * @return void
     */
    public function report(Exception $e)
    {
        if ($e instanceof CustomException)
        {
            //
        }

        return parent::report($e);
    }

`render` 메서드는 예외를 브라우저로 되돌려보내는 HTTP 응답으로 변환하는 책임을 지고 있습니다. 기본적으로, 예외는 여러분을 위해 응답을 생성해주는 기본 클래스로 전달 됩니다. 하지만, 여러분은 예외 형식을 확인 하거나 여러분만의 응답을 반환 할 수도 있습니다.

예외 처리의 `dontReport` 속성은 로그되지 말아야하는 예외 형식 배열을 포함합니다.  기본적으로, 404 오류의 예외 결과는 로그 파일에 작성되지 않습니다. 필요에 따라 다른 예외 형식들을 이 배열에 추가하세요.

<a name="http-exceptions"></a>
## HTTP 예외

몇몇의 예외들은 서버에서 HTTP 오류를 표시합니다. 예를 들어, "page not found" (404) 오류, "unauthorized error" (401) 오류 또는 개발자가 생성한 500 오류가 될 수 있습니다. 이러한 응답을 반환하려면, 다음을 사용하세요:

    abort(404);

옵션으로, 응답문을 제공 할 수도 있습니다:

    abort(403, 'Unauthorized action.');

이 메서드는 요청의 라이프사이클동안 언제든지 사용 될 수 있습니다.

### 사용자 정의 404 오류 페이지

모든 404 오류에 사용자 정의 뷰를 반환 하려면, `resources/views/errors/404.blade.php` 파일을 생성하세요. 이 뷰는 어플리케이션에 의해 생성되는 모든 404 오류들에게 제공 됩니다.

<a name="logging"></a>
## 로깅

라라벨 로깅 기능은 강력한 [Monolog](http://github.com/seldaek/monolog) 라이브러리 위에 간단한 레이어를 제공합니다. 기본적으로, 라라벨은 `storage/logs` 디렉토리에 저장되는 일별 로그파일을 생성하도록 설정되어 있습니다. 아래처럼 로그 정보를 작성 할 수 있습니다:

    Log::info('도움이 되는 정보입니다.');

    Log::warning('무언가가 잘못될 수 있습니다.');

    Log::error('무언가가 정말 잘못 되었습니다.');

로거는 [RFC 5424](http://tools.ietf.org/html/rfc5424)에 정의되어있는 다음의 7 가지 로깅 레벨을 제공합니다: **debug**, **info**, **notice**, **warning**, **error**, **critical**, 그리고 **alert**.

아무 문맥 데이터 배열 또한 로그 메서드에 전달 될 수 있습니다:

    Log::info('로그 메세지', ['context' => '유용한 다른 정보']);

Monolog는 로깅에 사용할 수 있는 다양한 추가 핸들러를 갖고 있습니다. 필요에 따라, 라라벨에 사용되는 Monolog 인스턴스를 액세스 할 수도 있습니다:

    $monolog = Log::getMonolog();

여러분은 또한 로그에 전달되는 모든 메세지를 캐치하는 이벤트를 등록 할 수도 있습니다:

#### 로그 이벤트 리스너 등록

    Log::listen(function($level, $message, $context)
    {
        //
    });
