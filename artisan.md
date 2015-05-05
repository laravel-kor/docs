# 아티즌 CLI

- [소개](#introduction)
- [사용법](#usage)
- [CLI 밖에서 커맨드 호출](#calling-commands-outside-of-cli)
- [아티즌 커맨드 스케쥴링](#scheduling-artisan-commands)

<a name="introduction"></a>
## 소개

아티즌(Artisan)은 라라벨에 포함되어 있는 커맨드라인 인터페이스 이름입니다. 아티즌은 어플리케이션을 개발 하는 동안 도움이 되는 몇가지의 커맨드를 제공합니다. 아티즌은 강력한 심포니 콘솔 컴포넌트에 의해 구동 됩니다.

<a name="usage"></a>
## 사용법

#### 사용 가능한 모든 커맨드 나열

`list` 커맨드를 사용하여 사용 가능한 모든 아티즌 커맨드 리스트를 볼 수 있습니다:

    php artisan list

#### 커맨드의 도움말 화면 보기

모든 커맨드는 사용 가능한 인수들과 옵션들을 보여주고 설명해주는 "도움말" 화면을 포함하고 있습니다. 간단히 커맨드 이름 앞에 `help`를 붙여 도움말 화면을 볼 수 있습니다:

    php artisan help migrate

#### 구성 환경 지정

`--env` 스위치를 사용하여 커맨드를 실행하는 동안 사용될 구성 환경을 지정 할 수 있습니다:

    php artisan migrate --env=local

#### 라라벨의 현재 버전 표시

또한 `--version` 옵션을 사용하여 현재 설치된 라라벨의 버전을 볼 수도 있습니다:

    php artisan --version

<a name="calling-commands-outside-of-cli"></a>
## CLI 밖에서 커맨드 호출

때때로 아티즌 커맨드를 CLI 밖에서 호출하기를 원할 경우도 있습니다. 예를 들어, HTTP 라우트에서 아티즌 커맨드를 실행하기를 원할 경우도 있을겁니다. 이럴 경우엔 `Artisan` 파사드를 사용하세요:

    Route::get('/foo', function()
    {
        $exitCode = Artisan::call('command:name', ['--option' => 'foo']);

        //
    });

또한 백그라운드에서 [큐 워커](/docs/{{version}}/queues)에 의해 실행되도록 아티즌 커맨들을 대기열에 등록 할 수도 있습니다.:

    Route::get('/foo', function()
    {
        Artisan::queue('command:name', ['--option' => 'foo']);

        //
    });

<a name="scheduling-artisan-commands"></a>
## 아티즌 커맨드 스케쥴링

과거에 개발자는 크론을 생성하여 각각의 콘솔 커맨드들을 스케쥴화 시켰습니다. 하지만, 이는 골치 아픕니다. 콘솔 스케쥴은 소스 컨트롤에 포함되어 있지도 않으며, 크론을 등록하려면 SSH를 통하여 서버에 접속해야만 합니다. 이제 우리의 삶을 편하게 만들어 봅시다. 라라벨 커맨드 스케쥴러는 우아하고 풍부하게 라라벨 자체에 커맨드 스케쥴을 정의 할수 있도록 해주며, 서버에 단 하나의 크론 엔트리만을 필요로 합니다.

커맨드 스케쥴은 `app/Console/Kernel.php` 파일에 위치해 있습니다. 이 클래스에서 `schedule` 메서드를 볼수 있습니다. 당신이 쉽게 시작할수 있도록 도와주는 간단한 예제가 포함되어 있습니다. `Schedule` 객체에 원하는 만큼의 스케쥴 작업을 추가할 수 있습니다. 서버에 등록하는 유일한 크론 엔트리는 아래와 같습니다:

    * * * * * php /path/to/artisan schedule:run 1>> /dev/null 2>&1

이 크로는 라라벨 커맨드 스케쥴러를 매분마다 호출합니다. 그런 다음, 라라벨이 각각의 스케쥴 작업들을 평가하여 기한된 작업들을 실행합니다. 이것보다 더 쉬울순 없습니다!

### 더 많은 스케쥴링 예제들

몇개의 스케쥴링 예제를 더 보도록 하겠습니다:

#### 클로저를 스케쥴화

    $schedule->call(function()
    {
        // Do some task...

    })->hourly();

#### 터미널 커맨드를 스케쥴화

    $schedule->exec('composer self-update')->daily();

#### 수동 크론 식

    $schedule->command('foo')->cron('* * * * *');

#### 반복적인 작업

    $schedule->command('foo')->everyFiveMinutes();

    $schedule->command('foo')->everyTenMinutes();

    $schedule->command('foo')->everyThirtyMinutes();

#### 매일 실행되는 작업

    $schedule->command('foo')->daily();

#### 매일 특정 시간에 실행되는 작업 (24 시간)

    $schedule->command('foo')->dailyAt('15:00');

#### 매일 하루에 두번 실행되는 작업

    $schedule->command('foo')->twiceDaily();

#### 모든 평일에 실행되는 작업

    $schedule->command('foo')->weekdays();

#### 주간마다 실행되는 작업

    $schedule->command('foo')->weekly();

    // 특정 요일과 시간마다 실행되는 주간 작업 스케쥴화
    $schedule->command('foo')->weeklyOn(1, '8:00');

#### 매월 실해되는 작업

    $schedule->command('foo')->monthly();

#### 특정 요일마다 실행되는 작업

    $schedule->command('foo')->mondays();
    $schedule->command('foo')->tuesdays();
    $schedule->command('foo')->wednesdays();
    $schedule->command('foo')->thursdays();
    $schedule->command('foo')->fridays();
    $schedule->command('foo')->saturdays();
    $schedule->command('foo')->sundays();

#### 작업이 겹치는 것을 방지

기본적으로, 스케쥴된 작업들은 이전의 작업 인스턴스가 계속 실행중인 상태라도 실행됩니다. 이를 방지하려면, `withoutOverlapping` 메서드를 사용합니다:

    $schedule->command('foo')->withoutOverlapping();

이 예제에서, `foo` 커맨드는 이미 실행되어 있지 않을 경우에만 매분마다 실행됩니다.

#### 작업이 실행되는 환경을 제한

    $schedule->command('foo')->monthly()->environments('production');

#### 어플리케이션이 유지관리 모드일때도 실행되도록 선언

    $schedule->command('foo')->monthly()->evenInMaintenanceMode();

#### 콜백이 `true` 일때만 작업 실행되도록 허용

    $schedule->command('foo')->monthly()->when(function()
    {
        return true;
    });

#### 스케쥴된 작업의 아웃풋을 이메일로 발송

    $schedule->command('foo')->sendOutputTo($filePath)->emailOutputTo('foo@example.com');

> **주의:** 이메일로 보내기 전에 먼저 파일로 저장해야합니다.

#### 주어진 위치의 파일로 스케쥴된 작업의 아웃풋을 송출

    $schedule->command('foo')->sendOutputTo($filePath);

#### 작업이 실행된 후, 주어진 URL로 핑

    $schedule->command('foo')->thenPing($url);

`thenPing($url)` 기능을 사용하려면 Guzzle HTTP 라이브러리가 필요합니다. 다음의 라인을 `composer.json` 파일에 추가하여 Guzzle 5를 여러분의 프로젝트에 추가 할 수 있습니다:

    "guzzlehttp/guzzle": "~5.0"
