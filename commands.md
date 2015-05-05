# 아티즌 개발

- [소개](#introduction)
- [커맨드 작성](#building-a-command)
- [커맨드 등록](#registering-commands)

<a name="introduction"></a>
## 소개

아티즌이 제공하는 커맨드 뿐만 아니라, 어플리케이션에서 사용 할 수 있도록 사용자 정의 커맨드를 만들수도 있습니다. 여러분은 `app/Console/Commands` 디렉토리에 사용자 정의 커맨드를 저장합니다. 하지만 `composer.json` 설정에 따라 오토로드만 된다면 어느 곳에 저장 해도 상관 없습니다.

<a name="building-a-command"></a>
## 커맨드 작성

### 클래스 생성

새로운 커맨드를 만들려면, 여러분이 시작하는데 도움을 줄 수 있는 커맨드 스텁을 생성 하는 `make:console` 아티즌 커맨드를 사용할 수 있습니다:

#### 새로운 커맨드 클래스 생성

    php artisan make:console FooCommand

위의 커맨드는 `app/Console/Commands/FooCommand.php`에 클래스를 생성합니다.

커맨드를 생성할 때, `--command` 옵션은 터미널 커맨드명을 부여할때 사용됩니다:

    php artisan command:make AssignUsers --command=users:assign

### 커맨드 작성

커맨드가 생성되고 나면, `list` 화면에 여러분의 커맨드를 표시 할 때 사용되는 클래스의 `name`과 `description` 속성을 기입해야 합니다.

`fire` 메서드는 커맨드가 실행될 때 호출됩니다. 이 메서드에 모든 커맨드 로직을 배치하면 됩니다.

### 인수 & 옵션

`getArguments`와 `getOptions` 메서드는 커맨드가 받는 모든 인수와 옵션을 정의 하는 곳입니다. 두 메서드 모두 배열 옵션 목록이 묘사되어 있는 커맨드의 배열을 반환합니다.

`arguments`를 정의 할 때, 배열 정의 값은 다음을 나타냅니다:

    [$name, $mode, $description, $defaultValue]

`mode` 인수는 `InputArgument::REQUIRED`와 `InputArgument::OPTIONAL` 둘 중 어느 것이든 될 수 있습니다.:

`options`를 정의 할 때, 배열 정의 값은 다음을 나타냅니다:

    [$name, $shortcut, $mode, $description, $defaultValue]

옵션에서 `mode` 인수는 다음 중 하나가 될 수있습니다: `InputOption::VALUE_REQUIRED`, `InputOption::VALUE_OPTIONAL`, `InputOption::VALUE_IS_ARRAY`, `InputOption::VALUE_NONE`.

`VALUE_IS_ARRAY` 모드는 커맨드를 호출할 때 해당 스위치가 여러번 사용 될 수 있음을 나타냅니다:

    InputOption::VALUE_REQUIRED | InputOption::VALUE_IS_ARRAY

위의 모드는 다음의 커맨드를 허용합니다:

    php artisan foo --option=bar --option=baz

`VALUE_NONE` 옵션은 해당 옵션 자체가 간단하게 "스위치"로 사용 될 수 있음을 나타냅니다:

    php artisan foo --option

### Input 조회

커맨드가 실행되는 동안 어플리케이션이 받는 인수와 옵션의 값을 액세스 할 수 있어야 합니다. `argument`와 `option` 메서드를 사용하여 엑세스 할 수 있습니다:

#### 커맨드의 해당 인수 값 조회

    $value = $this->argument('name');

#### 모든 인수 조회

    $arguments = $this->argument();

#### 커맨드의 해당 옵션 값 조회

    $value = $this->option('name');

#### 모든 옵션 조회

    $options = $this->option();

### 출력 작성

콘솔에 출력을 보내려면 `info`, `comment`, `question`, `error` 메서드를 사용합니다. 각각의 메서드는 해당 목적에 맞는 ANSI 색상을 사용합니다.

#### 콘솔에 정보 출력

    $this->info('Display this on the screen');

#### 콘솔에 오류 메세지 출력

    $this->error('Something went wrong!');

### 질의

또한 `ask`와 `confirm` 메서드를 사용하여 사용자 프롬프트 입력을 유도 할 수 있습니다:

#### 사용자 입력 요청

    $name = $this->ask('What is your name?');

#### 비공개 사용자 입력 요청

    $password = $this->secret('What is the password?');

#### 사용자 확인 요청

    if ($this->confirm('Do you wish to continue? [yes|no]'))
    {
        //
    }

또한 `confirm` 메서드에 `true`나 `false`가 되는 기본 값을 지정 할 수도 있습니다:

    $this->confirm($question, true);

### 다른 커맨드 호출

때때로 여러분의 커맨드에서 다른 커맨드를 호출 해야 할 때도 있습니다. 그럴 땐 `call` 메서드를 사용합니다:

    $this->call('command.name', ['argument' => 'foo', '--option' => 'bar']);

<a name="registering-commands"></a>
## 커맨드 등록

#### 아티즌 커맨드 등록

커맨드가 완성되면, 커맨드를 사용 할 수 있도록 아티즌에 등록해야 합니다. 이 작업은 일반적으로 `app/Console/Kernel.php` 파일에서 수행됩니다. 이 파일에서, 여러분은 `commands` 속성에서 커맨드 목록을 찾아 볼 수 있습니다. 여러분의 커맨드를 등록하려면, 간단하게 이 목록에 추가하세요.

    protected $commands = [
        'App\Console\Commands\FooCommand'
    ];

아티즌이 부팅할 때, 이 속성에 나열된 모든 커맨드들은 [서비스 컨테이너](/docs/{{version}}/container)에 의해 해결되고 아티즌에 등록됩니다.
