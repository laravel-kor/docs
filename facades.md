# 파사드

- [소개](#introduction)
- [설명](#explanation)
- [실제 사용법](#practical-usage)
- [파사드 생성](#creating-facades)
- [파사드 흉내(Mocking)](#mocking-facades)

<a name="introduction"></a>
## 소개

파사드는 어플리케이션의 [IoC 컨테이너](/docs/ioc)에서 사용 가능한 클래스들에게 "static" 인터페이스를 제공합니다. 라라벨은 많은 파사드를 포함하고 있으며, 아마 본인도 모른채 사용해봤을 겁니다!

가끔, 어플리케이션과 패키지에 당신만의 파사드를 만들고 싶어 할 지도 모릅니다. 그러므로 컨셉과, 개발, 이러한 클래스들의 사용법을 탐구해 봅시다.

> **노트:** 파사드를 파헤치지 전에, [IoC 컨테이너](/docs/ioc)와 매우 익숙해지길 추천합니다.

<a name="explanation"></a>
## 설명

라라벨 어플리케이션에서 파사드란, 컨테이너로부터 오브젝트에 액세스를 제공하는 클래스입니다. 이를 가능하게 만드는 장치는 `Facade` 클래스 안에 있습니다. 라라벨의 파사드와 당신이 만든 어떠한 파사드라도 기본 `Facade` 클래스를 확장합니다.

당신의 파사드 클래스는 `getFacadeAccessor` 메소드 오직 한가지만 제공하면 됩니다. 컨테이너로부터 무엇을 분석할지 정의하는게 `getFacadeAccessor` 메소드가 하는 일 입니다. `Facade` 베이스 클래스는 당신의 파사드에서 분석된 오브젝트로의 호출을 연기하기위해 `__callStatic()` 매직-메소드를 사용합니다.

<a name="practical-usage"></a>
## 실제 사용법

아래의 예제에서, 라라벨 캐시 시스템으로의 호출이 만들어졌습니다. 이 코드를 보면, `Cache` 클래스의 static 메소드 `get`이 호출됐다는 걸 짐작 할 수 있습니다.

	$value = Cache::get('key');

하지만, `Illuminate\Support\Facades\Cache` 클래스를 살펴보면, 그곳에는 static `get` 메소드가 없다는걸 보게될겁니다.:

	class Cache extends Facade {

		/**
		 * Get the registered name of the component.
		 *
		 * @return string
		 */
		protected static function getFacadeAccessor() { return 'cache'; }

	}

캐시 클래스는 `Facade` 클래스를 확장하고 `getFacadeAccessor()` 메소드를 정의합니다. 이 메소드가 하는 일은 IoC 바인딩 이름을 반환 한다는것을 기억하세요.

사용자가 `Cache` 파사드의 어떤 static 메소드를 참조 할지라도, 라라벨은 IoC 컨테이너에서 `cache` 바인딩을 분석하고 그 오브젝트에 비교하여 요청된 메소드(이경우, `get`)를 실행합니다.

그러므로, `Cache::get` 호출은 다음과 같이 바꿔쓸수 있습니다.:

	$value = $app->make('cache')->get('key');

<a name="creating-facades"></a>
## 파사드 생성

자신의 어플리케이션이나 패키지를 위한 파사드를 생성하는 일은 간단합니다. 단지 다음의 3가지가 필요합니다.:

- IoC 바인딩
- 파사드 클래스
- 파사드 별칭 구성

예제를 한번 봅시다. `PaymentGateway\Payment`라고 정의된 클래스가 있습니다.

	namespace PaymentGateway;

	class Payment {

		public function process()
		{
			//
		}

	}

IoC 컨테이너에서 이 클래스를 분석할수 있도록 해야합니다. 그러므로, 바인딩을 추가합니다.:

	App::bind('payment', function()
	{
		return new \PaymentGateway\Payment;
	});

이 바인딩을 등록할 좋은 위치는 `PaymentServiceProvider`라는 이름의 [서비스 프로바이더](/docs/ioc#service-providers)를 만들고 바인딩을 `register` 메소드에 추가하는겁니다. 그런다음 라라벨이 당신의 서비스프로바이더를 로드할수 있도록 `app/config/app.php` 설정파일에 설정합니다.

다음 자신만의 파사드 클래스를 만들 수 있습니다.:

	use Illuminate\Support\Facades\Facade;

	class Payment extends Facade {

		protected static function getFacadeAccessor() { return 'payment'; }

	}

끝으로, 원한다면, 자신의 파사드를 위한 별칭을 `app/config/app.php` 설정파일의 `aliases` 배열에 추가할 수 있습니다. 이제 `Payment` 클래스 인스턴스의 `process` 메소드를 호출 할 수 있습니다.

	Payment::process();

### 오토로드 별칭들에 대한 노트

`aliases` 배열에 있는 클래스들 중 몇몇은 [PHP가 정의되지 않은 유형암시 클래스들에 대해 오토로드를 시도하지 않으므로](https://bugs.php.net/bug.php?id=39003) 이용할 수 없습니다. 만약 `\ServiceWrapper\ApiTimeoutException`가 `ApiTimeoutException`의 별칭이라면, `\ServiceWrapper` 네임스페이스 외부에 있는 `catch(ApiTimeoutException $e)`는 예외가 발생하더라도 예외를 처리하지 않습니다. 비슷한 문제가 별칭을 사용한 클래스에 유형암시를 갖고있는 모델에서도 있습니다. 이를 해결 할 유일한 방법은 별칭을 포기하고, 각각의 클래스가 필요로하는 파일들을 파일의 상단에 `use`를 사용하여 유형 암시를 하는 방법입니다.

<a name="mocking-facades"></a>
## 파사드 흉내(Mocking)

유닛테스팅은 파사드가 작동하는 방식이 왜 중요한지 보여주는 양상입니다. 사실, 테스트성이 파사드가 존재하는 주된 이유 입니다. 매뉴얼의 [파사드 흉내(mocking)](/docs/testing#mocking-facades) 섹션에서 좀더 많은 정보를 확인 할 수 있습니다.
