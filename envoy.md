# 엔보이(envoy) 작업 실행기

- [소개](#introduction)
- [설치](#envoy-installation)
- [작업 실행](#envoy-running-tasks)
- [다중 서버](#envoy-multiple-servers)
- [병렬 실행](#envoy-parallel-execution)
- [작업 매크로](#envoy-task-macros)
- [알림](#envoy-notifications)
- [엔보이 업데이트](#envoy-updating-envoy)

<a name="introduction"></a>
## 소개

[라라벨 엔보이](https://github.com/laravel/envoy)는 여러분의 원격 서버에서 실행되는 일적인 작업을 정의해주는 깨끗하고, 최소한의 구문을 제공합니다. 블레이드 스타일의 구문을 사용하여, 여러분은 배포, 아티즌 커맨드 등을 위한 작업을 쉽게 설정 할 수 있습니다.

> **주의:** 엔보이는 PHP 5.4 이상의 버전을 요구하며, Mac과 Linux 운영체제에서만 실행됩니다.

<a name="envoy-installation"></a>
## 설치

첫번째로, 컴포저 `global` 커맨드를 사용하여 엔보이를 설치합니다:

    composer global require "laravel/envoy=~1.0"

터미널에서 `envoy` 커맨드를 실행 했을때 `envoy`가 실행 가능하도록 여러분의 PATH에 `~/.composer/vendor/bin` 경로가 추가되어있는지 확인하세요.

다음으로, 여러분의 프로젝트 루트에 `Envoy.blade.php` 파일을 생성합니다. 여기 여러분이 시작할 수 있도록 예제가 있습니다:

    @servers(['web' => '192.168.1.1'])

    @task('foo', ['on' => 'web'])
        ls -la
    @endtask

보이는 바와 같이, `@servers` 배열이 파일의 가장 윗부분에 정의되어 있습니다. 여러분은 작업 선언의 `on` 옵션에서 이 서버들을 참조 할 수 있습니다. `@task` 선언에서는 작업이 실행 됐을때 여러분의 서버에서 실행되어야 할 배쉬 코드를 배치해야 합니다.

`init` 커맨드는 스텁 엔보이 파일을 쉽게 생성하는데 사용 할 수 있습니다:

    envoy init user@192.168.1.1

<a name="envoy-running-tasks"></a>
## 작업 실행

작업을 실행하려면, 엔보이의 `run` 커맨드를 사용합니다:

    envoy run foo

필요하다면, 커맨드 라인 스위치를 사용하여 엔보이에 변수를 전달 할 수도 있습니다:

    envoy run deploy --branch=master

여러분이 사용하던 블레이드 구문을 통해 해당 옵션을 사용 할 수 있습니다:

    @servers(['web' => '192.168.1.1'])

    @task('deploy', ['on' => 'web'])
        cd site
        git pull origin {{ $branch }}
        php artisan migrate
    @endtask

#### 부트스트랩

```@setup``` 지시문을 사용하여 엔보이 파일 안에서 변수들을 선언하고 일반적인 PHP 작업을 할 수 있습니다:

    @setup
        $now = new DateTime();

        $environment = isset($env) ? $env : "testing";
    @endsetup

또한 ```@include```를 사용하여 PHP 파일들도 포함 시킬 수 있습니다:

    @include('vendor/autoload.php');

#### 실행하기 전에 확인하기

만약 주어진 작업을 실행하기 전에 확인 안내를 받으려면, `confirm` 지시문을 사용합니다:

    @task('deploy', ['on' => 'web', 'confirm' => true])
        cd site
        git pull origin {{ $branch }}
        php artisan migrate
    @endtask

<a name="envoy-multiple-servers"></a>
## 다중 서버

여러분은 쉽게 여러 서버에서 작업을 실행 할 수 있습니다. 간단히 작업 선언에 해당 서버들을 나열 하세요:

    @servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

    @task('deploy', ['on' => ['web-1', 'web-2']])
        cd site
        git pull origin {{ $branch }}
        php artisan migrate
    @endtask

기본적으로, 작업은 각각의 서버에 직렬로 실행됩니다. 이는 다음 서버에서 실행되기전에 첫번째 서버에서 실행이 완료됨을 의미합니다.

<a name="envoy-parallel-execution"></a>
## 병렬 실행

만약 여러개의 서버에서 병렬로 작업을 실행하고 싶다면, 작업 선언에 `parallel` 옵션을 추가하세요:

    @servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

    @task('deploy', ['on' => ['web-1', 'web-2'], 'parallel' => true])
        cd site
        git pull origin {{ $branch }}
        php artisan migrate
    @endtask

<a name="envoy-task-macros"></a>
## 작업 매크로

매크로는 하나의 커맨드를 사용하여 일련의 작업을 순서대로 실행하도록 정의하게 해줍니다. 예를 들면:

    @servers(['web' => '192.168.1.1'])

    @macro('deploy')
        foo
        bar
    @endmacro

    @task('foo')
        echo "HELLO"
    @endtask

    @task('bar')
        echo "WORLD"
    @endtask

`deploy` 매크로는 이제 하나의 간단한 커맨드를 통해 실행 될 수 있습니다:

    envoy run deploy

<a name="envoy-notifications"></a>
<a name="envoy-hipchat-notifications"></a>
## 알림

#### HipChat

작업을 실행한 뒤, `@hipchat` 지시문을 사용하여 여러분 팀의 HipChat 방에 알림을 보낼 수 있습니다.:

    @servers(['web' => '192.168.1.1'])

    @task('foo', ['on' => 'web'])
        ls -la
    @endtask

    @after
        @hipchat('token', 'room', 'Envoy')
    @endafter

또한 사용자 정의 메세지를 지어 할 수도 있습니다. ```@setup``` 또는 ```@include```를 사용하여 포함되어 선언된 어떤 변수라도 메세지를 작성하는데 사용 될 수 있습니다:

    @after
        @hipchat('token', 'room', 'Envoy', "$task ran on [$environment]")
    @endafter

이는 여러분의 팀이 서버에서 실행되는 작업의 알림을 계속 받을 수 있는 놀랍도록 간단한 방법입니다.

#### Slack

다음의 구문은 [Slack](https://slack.com)에 알림을 전송하는데 사용됩니다:

    @after
        @slack('hook', 'channel', 'message')
    @endafter

여러분은 슬랙의 웹사이트에서 `Incoming WebHooks`을 생성하여 여러분의 웹후크 URL을 조회 할 수 있습니다. `hook` 인수는 슬랙의 인커밍 웹훅에서 제공되는 전체 URL이어야 합니다. 예를 들면:

    https://hooks.slack.com/services/ZZZZZZZZZ/YYYYYYYYY/XXXXXXXXXXXXXXX

여러분은 channel 인수에 다음 중 하나를 제공 할 수 있습니다:

- 채널에 알림을 전송: `#channel`
- 사용자에게 알림을 전송: `@user`

만약 `channel` 인수가 제공되지 않는다면 기본 채널이 사용됩니다.

> 주의: 슬랙 알림은 모든 태스크가 성공적으로 완료되었을 때만 전송됩니다.

<a name="envoy-updating-envoy"></a>
## 엔보이 업데이트

엔보이를 업데이트하려면, 컴포저를 사용하세요:

    composer global update

