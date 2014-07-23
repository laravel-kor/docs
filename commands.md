# 아티즌 개발

- [소개](#introduction)
- [커맨드 작성](#building-a-command)
- [커맨드 등록](#registering-commands)
- [다른 커맨드 호출](#calling-other-commands)

<a name="introduction"></a>
## 소개

아티즌이 제공하는 커맨드 뿐만 아니라, 어플리케이션에서 사용할 수 있도록 사용자 정의 커맨드를 만들 수 도 있습니다. `app/commands` 디렉토리에 사용자 정의 커맨드를 저장합니다. 하지만 `composer.json` 설정에 따라 오토로드만 된다면 어느 곳 이든 저장 해도 상관 없습니다.

<a name="building-a-command"></a>
## 커맨드 작성

### 클래스 생성

시작 하는데 도움이 될 뼈대를 생성해주는 `command:make` 아티즌 커맨드를 사용하여 새로운 커맨드를 생성할 수 있습니다.:

**새로운 커맨드 클래스 생성**

    php artisan command:make FooCommand

기본적으로, 생성된 커맨드는 `app/commands` 디렉토리에 저장됩니다. 하지만 사용자 정의 경로나 네임스페이스를 지정 할 수도 있습니다.:

	php artisan command:make FooCommand --path=app/classes --namespace=Classes

### 커맨드 작성

커맨드가 생성되고 나면 `list` 화면에 당신의 커맨드를 표시 할 때 사용되는 클래스의 `name`과 `description` 속성을 기입해야 합니다.

`fire` 메소드는 커맨드가 실행될 때 호출됩니다. 이 메소드에 어떤 커맨드 로직이든 배치하면 됩니다.

### 인수 & 옵션

`getArguments`와 `getOptions` 메소드는 커맨드가 받는 모든 인수와 옵션을 정의 하는 곳입니다. 두 메소드 모두 배열 옵션 목록이 묘사되어 있는 커맨드의 배열을 반환합니다.

`arguments` 를 정의 할 때, 배열 정의 값은 다음을 나타냅니다.:

	array($name, $mode, $description, $defaultValue)

`mode` 인수는 `InputArgument::REQUIRED`와 `InputArgument::OPTIONAL` 어느 것이든 될 수 있습니다.:

`options` 를 정의 할때, 배열 정의 값은 다음을 나타냅니다.:

	array($name, $shortcut, $mode, $description, $defaultValue)ㄴㅇㅇ

옵션에서 `mode` 인수는 `InputOption::VALUE_REQUIRED`, `InputOption::VALUE_OPTIONAL`, `InputOption::VALUE_IS_ARRAY`, `InputOption::VALUE_NONE` 넷 중 하나가 될수 있습니다.

`VALUE_IS_ARRAY` 모드는 커맨드를 호출할 때 해당 스위치가 여러번 사용 될 수 있음을 나타냅니다.:

	php artisan foo --option=bar --option=baz

`VALUE_NONE` 옵션은 해당 옵션 자체가 간단하게 "스위치"로 사용 될 수 있음을 나타냅니다.:

	php artisan foo --option

### Input 조회

커맨드가 실행되는 동안 어플리케이션이 받는 인수와 옵션의 값을 액세스 할 수 있어야 합니다. 그렇게 하려면 `argument`와 `option` 메소드를 사용 합니다.:

**커맨드의 해당 인수 값 조회**

	$value = $this->argument('name');

**모든 인수 조회**

	$arguments = $this->argument();

**커맨드의 해당 옵션 값 조회**

	$value = $this->option('name');

**모든 옵션 조회**

	$options = $this->option();

### 출력 작성

콘솔에 출력을 보내려면 `info`, `comment`, `question`, `error` 메소드를 사용합니다. 각각의 메소드는 해당 목적에 맞는 ANSI 색상을 사용합니다.

**콘솔에 정보 출력**

	$this->info('이 문장을 스크린에 출력');

**콘솔에 오류 메세지 출력**

	$this->error('무언가 잘못 되었습니다.!');

### 질문

또한 `ask`와 `confirm` 메소드를 사용하여 사용자 프롬프트 입력을 할 수 있습니다.:

**사용자 입력 요청**

	$name = $this->ask('What is your name?');

**비공개 사용자 입력 요청**

	$password = $this->secret('What is the password?');

**사용자 확인 요청**

	if ($this->confirm('Do you wish to continue? [yes|no]'))
	{
		//
	}

또한 `confirm` 메소드에 `true`나 `false`가 되는 기본 값을 지정 할 수도 있습니다.:

	$this->confirm($question, true);

<a name="registering-commands"></a>
## 커맨드 등록

커맨드가 완성되면, 커맨드를 사용 할 수 있도록 아티즌에 등록해야 합니다. 이 작업은 일반적으로 `app/start/artisan.php` 파일에서 수행됩니다. 이 파일에서 `Artisan::add` 메소드를 사용하여 커맨드를 등록 할 수 있습니다.:

**아티즌 커맨드 등록**

	Artisan::add(new CustomCommand);

만약 커맨드가 어플리케이션의 [IoC 컨테이너](/docs/ioc)에 등록되어 있다면, `Artisan::resolve` 메소드를 사용하여 아티즌에서 사용 가능 하도록 할 수 있습니다.

**IoC 컨테이너에 있는 커맨드 등록**

	Artisan::resolve('binding.name');

<a name="calling-other-commands"></a>
## 다른 커맨드 호출

때때로 당신의 커맨드에서 다른 커맨드를 호출 해야 할 때도 있습니다. 그럴 땐 `call` 메소드를 사용합니다.:

**다른 커맨드 호출**

	$this->call('command.name', array('argument' => 'foo', '--option' => 'bar'));
