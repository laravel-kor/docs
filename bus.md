# 커맨드 버스

- [소개](#introduction)
- [커맨드 생성](#creating-commands)
- [커맨드 디스패치](#dispatching-commands)
- [대기 커맨드](#queued-commands)
- [커맨드 파이프라인](#command-pipeline)

<a name="introduction"></a>
## 소개

라라벨 커맨드 버스는 여러분의 어플리케이션이 이해하기 쉽고 간편한 "커맨드"를 수행하도록 태스크를 캡슐화 시키는 편리한 방법을 제공합니다. 커맨드의 목적을 이해하기 위해, 사용자가 팟캐스트를 구입 할 수 있는 어플리케이션을 만들고있다고 가정해봅시다.

사용자가 팟캐스트를 구매할 때, 굉장히 많은 상황들 일어납니다. 예를 들어, 우리는 사용자의 신용카드를 청구해야하고, 구매 내역을 데이터베이스에 기록해야하고, 구매 확인 이메일을 보내야 합니다. 어쩌면 우리는 사용자가 팟캐스트를 구매 할 수 있는지 검증과 같은 일을 해야 할지도 모릅니다.

우리는 이 모든 로직들을 하나의 컨트롤러의 메서드 안에 넣을 수 있습니다; 하지만, 이는 몇가지의 단점이 있습니다. 첫번째 단점은, 우리의 컨트롤러는 아마 여러개의 다른 HTTP 인커밍 액션들을 처리할것이고, 복잡한 로직을 포함하고 있는 각각의 메서드는 곧 우리의 컨트롤러를 쓸데없이 확중 시킬것이며 읽기 어렵게 만들것입니다. 두번째로, 컨트롤러 컨텍스트의 외부에서 팟캐스트 구입 로직을 재사용하기가 어렵습니다. 새번째로, 스텁 HTTP 요청을 생성하고 팟캐스트 구입 로직을 테스트 하는데 필요한 모든 요청을 해야하므로 이러한 커맨드를 유닛테스트 하기가 더 어렵습니다.

이러한 로직을 컨트롤러 안에 넣는 것 대신에, 우리는 `PurchasePodcast` 커맨드 같은 "command" 객체 안에 캡슐화 시키는 방법을 선택 할 수도 있습니다.

<a name="creating-commands"></a>
## 커맨드 생성

아티즌 CLI의 `make:command` 명령을 사용하여 새로운 커맨드 클래스를 생성 할 수 있습니다:

    php artisan make:command PurchasePodcast

새롭게 생성된 클래스는 `app/Commands` 디렉토리 안에 위치해 있습니다. 기본적으로, 이 커맨드는 생성자와 `handle` 2개의 메서드를 포함하고있습니다. 물론 생성자는 여러분이 관련된 아무 객체나 전달 할 수 있도록 하고, `handle` 메서드는 커맨드를 실행합니다. 예를 들어:

    class PurchasePodcast extends Command implements SelfHandling {

        protected $user, $podcast;

        /**
         * 새로운 커맨드 인스턴스 생성
         *
         * @return void
         */
        public function __construct(User $user, Podcast $podcast)
        {
            $this->user = $user;
            $this->podcast = $podcast;
        }

        /**
         * 커맨드 실행
         *
         * @return void
         */
        public function handle()
        {
            // Handle the logic to purchase the podcast...

            event(new PodcastWasPurchased($this->user, $this->podcast));
        }

    }

`handle` 메서드는 또한 의존성들을 타입힌트 할 수 있으며, [서비스 컨테이너](/docs/5.0/container)에 의해 자동으로 주입 됩니다. 예를 들어:

        /**
         * 커맨드 실행
         *
         * @return void
         */
        public function handle(BillingGateway $billing)
        {
            // Handle the logic to purchase the podcast...
        }

<a name="dispatching-commands"></a>
## 커맨드 디스패치

자 이제, 커맨드를 생성하고 나면, 어떻게 디스패치 할것인가요? 물론, 우리는 `handle` 메서드를 직접 호출 할 수도 있습니다. 하지만, 라라벨의 "커맨드 버스"를 통해 커맨드를 디스패치 하면 나중에 이야기 할 몇몇의 장점이 있습니다.

여러분의 베이스 컨트롤러를 훑어본다면, `DispatchesCommands` 트레이트를 볼 수 있습니다. 이 트레이트는 우리가 어떤 컨트롤러에서도 `dispatch` 메서드를 호출 할 수 있도록 해줍니다. 예를 들어:

    public function purchasePodcast($podcastId)
    {
        $this->dispatch(
            new PurchasePodcast(Auth::user(), Podcast::findOrFail($podcastId))
        );
    }

이 커맨드 버스는 해당 커맨드의 실행을 맡아주고 `handle` 메서드 필요한 의존성들을 주입하는 IoC 컨테이너를 호출합니다.

`Illuminate\Foundation\Bus\DispatchesCommands` 트레이트를 여러분이 원하는 아무 클래스에 추가 할 수도 있습니다. 만약 클래스의 생성자를 통하여 커맨드 버스 인스턴스를 받길 원한다면, `Illuminate\Contracts\Bus\Dispatcher` 인터페이스를 타입-힌트 하세요. 끝으로, 빠른 디스패치 커맨드를 위해 `Bus` 파사드를 사용 할 수 도 있습니다.

        Bus::dispatch(
            new PurchasePodcast(Auth::user(), Podcast::findOrFail($podcastId))
        );

### 요청으로부터 커맨드 프로퍼티 맵핑

HTTP 요청을 변수들을 커맨드로 맵핑 하는일이 종종 있습니다. 그래서, 각각의 요청마다 여러분이 이런 일을 수동으로 직접 하는것 대신에, 라라벨은 몇가지의 헬퍼 메서드를 제공 합니다. `DispatchesCommands` 트레이트에서 사용 가능한 `dispatchFrom` 메서드를 봅시다:

    $this->dispatchFrom('Command\Class\Name', $request);

이 메서드는 커맨드 클래스에 주어진 생성자를 검사하고, 커맨드 생성자에 필요한 매개변수들을 채우기 위해 HTTP 요청으로부터 변수들(또는 다른 `ArrayAccess` 객체)을 추출합니다. 그러므로, 만약 우리의 커맨드 클래스가 생성자에 `firstName` 변수를 받는다면, 커맨드 버스는 HTTP 요청으로부터 `firstName` 매개변수를 받으려고 시도합니다.

`dispatchFrom` 메서드의 새번째 인수에 배열을 전달 할 수도 있습니다. 이 배열은 요청에서 사용할 수 없는 매개변수를 생성자의 매개변수에 채워주는데 사용됩니다:

    $this->dispatchFrom('Command\Class\Name', $request, [
        'firstName' => 'Taylor',
    ]);

<a name="queued-commands"></a>
## 대기 커맨드

커맨드 버스는 현재의 요청 주기동안 실행되는 동기 작업을 위한것 뿐만이 아니라, 라라벨에서 대기중인 작업을 구축 하는 기본 방법으로서 역할도 합니다. 그러면, 어떻게 우리는 커맨드 버스가 동기적 처리 대신 백그라운드 처리를 위한 대기 작업을 실행하도록 할까요? 그것은 간단합니다. 첫번째로, 새로운 커맨드를 생성할 때, `--queued` 플래그만 추가하면 됩니다:

    php artisan make:command PurchasePodcast --queued

여러분이 볼 수 있듯이, 이는 `Illuminate\Contracts\Queue\ShouldBeQueued` 인터페이스와 `SerializesModels` 트레이트 기능을 커맨드에 추가합니다. 이것들은 커맨드를 대기시키도록 커맨드 버스에 지시할 뿐만아니라, 우아하게 시리얼라이즈화시키고, 여러분의 커맨드가 속성으로 저장하고있는 엘로퀀트 모델들을 역시리얼라이즈화 시킵니다.

여러분이 만약 이미 존재하는 커맨드를 대기 커맨드로 변환하려 한다면, 간단하게 그 클래스에 수동으로 `Illuminate\Contracts\Queue\ShouldBeQueued` 인터페이스를 구현하세요. 이 인터페이스는 메서드를 포함하고 있지 않으며, 단지 디스패처를 위한 "마커 인터페이스"의 역할을 합니다.

그런 다음, 평소처럼 여러분의 커맨드를 작성하세요. 커맨드를 버스에 디스패치 할때, 자동으로 버스가 해당 커맨드를 백그라운드 처리로 대기 시킵니다. 이보다 더 쉬울 순 없습니다.

대기 커맨드의 상호작용에 대한 자세한 정보는 전체 [대기 문서](/docs/5.0/queues)를 참조하세요.

<a name="command-pipeline"></a>
## 커맨드 파이프라인

커맨드가 핸들러에 디스패치 되기 전에, "파이프라인"에서 커맨드를 다른 클래스로 전달 할 수도 있습니다. 커맨드 파이프는 커맨드 전용이라는 것을 빼고는 HTTP 미들웨어처럼 동작합니다! 예를 들어, 커맨드 파이프는 데이터베이스 트랜잭션 내에서 전체 커맨드 연산을 래핑 할 수도 있고, 간단하게 실행을 로그 할 수도 있습니다.

여러분의 버스에 파이프를 추가하려면, `App\Providers\BusServiceProvider::boot` 메서드에서 디스패처의 `pipeThrough` 메서드를 호출합니다:

    $dispatcher->pipeThrough(['UseDatabaseTransactions', 'LogCommand']);

커맨드 파이프는 미들웨어처럼 `handle` 메서드와 함께 정의됩니다:

    class UseDatabaseTransactions {

        public function handle($command, $next)
        {
            return DB::transaction(function() use ($command, $next)
            {
                return $next($command);
            });
        }

    }

커맨드 파이프 클래스는 [IoC 컨테이너](/docs/5.0/container)를 통해서 해결됩니다, 그러므로 생성자에서 필요한 아무 의존성이나 타입-힌트하여 사용하세요.

또한 `Closure`를 커맨드 파이프로 정의 할 수도 있습니다:

    $dispatcher->pipeThrough([function($command, $next)
    {
        return DB::transaction(function() use ($command, $next)
        {
            return $next($command);
        });
    }]);
