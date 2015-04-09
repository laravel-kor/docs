# 서비스 컨테이너

- [소개](#introduction)
- [기본 사용법](#basic-usage)
- [구현 클래스에 인터페이스를 주입](#binding-interfaces-to-implementations)
- [상황에 따른 바인딩](#contextual-binding)
- [태깅](#tagging)
- [실용적인 어플리케이션](#practical-applications)
- [컨테이너 이벤트](#container-events)

<a name="introduction"></a>
## 소개

라라벨 서비스 컨테이너는 의존된 클래스를 관리하는데 쓰이는 강력한 도구입니다. 의존성 주입은 본질적으로 다음을 뜻하는 멋진 단어입니다: 클래스의 의존성들은 생성자 또는, 어떤 상황에서는, "setter" 메서드를 통해 클래스로 "주입" 된다.

간단한 예제를 보겠습니다:

    <?php namespace App\Handlers\Commands;

    use App\User;
    use App\Commands\PurchasePodcastCommand;
    use Illuminate\Contracts\Mail\Mailer;

    class PurchasePodcastHandler {

        /**
         * 메일러 구현
         */
        protected $mailer;

        /**
         * 새로운 인스턴스 생성
         *
         * @param  Mailer  $mailer
         * @return void
         */
        public function __construct(Mailer $mailer)
        {
            $this->mailer = $mailer;
        }

        /**
         * 팟캐스트 구입
         *
         * @param  PurchasePodcastCommand  $command
         * @return void
         */
        public function handle(PurchasePodcastCommand $command)
        {
            //
        }

    }

이 예제에서, `PurchasePodcast` 커맨드는 팟캐스트를 구입 했을 때 이메일 전송을 처리하는데 필요한 커맨드 입니다. 그러므로, 우리는 이메일을 전송 할 수 있도록 해주는 서비스를 **주입** 합니다. 해당 서비스는 주입이 되기 때문에, 우리는 쉽게 다른 구현 클래스로 교체 할 수 있습니다. 또한 우리의 어플리케이션을 테스트 할때 쉽게 "모형(mock)화" 하거나 더미 구현 클래스를 생성 할 수도 있습니다.

라라벨 서비스 컨테이너에 대한 깊은 이해는 강력하고, 커다란 어플리케이션을 만드는것 뿐만 아니라, 라라벨 코어 자체에 기여할 수 있는 본질적인 요소입니다.

<a name="basic-usage"></a>
## 기본 사용법

### 바인딩

대부분의 서비스 컨테이너 바인딩들은 [서비스 공급자](/docs/5.0/providers)안에 등록됩니다, 그러므로 이 모든 예제들은 해당 문맥안에 있는 컨테이너를 사용하는 것으로 표현됩니다. 그렇지만, 만약 여러분이, 팩토리 같이, 어플리케이션의 다른 곳에서 컨테이너 인스턴스를 필요로 한다면, `Illuminate\Contracts\Container\Container` 계약(contract)을 타입-힌트하여 컨테이너 인스턴스를 주입 할 수 있습니다. 또 다른 방법으로는, `App` 파사드를 사용하여 컨테이너에 엑세스 할 수도 있습니다.

#### 기본 해결자 등록

서비스 공급자 안에서, 여러분은 `$this->app` 인스턴스 변수를 통하여 항상 컨테이너에 엑세스 할 수 있습니다.

서비스 공급자가 클로저 콜백과 구현 클래스에 인터페이스를 바인딩하는 것과 함께 의존성을 등록할 수 있는 방법에는 여러가지가 있습니다. 첫번째로, 우리는 클로저 콜백을 살펴 보겠습니다. 클로저 해결자는 키 (보통은 클래스명)와 어떤 값을 반환하는 클로저와 함께 컨테이너에 등록됩니다:

    $this->app->bind('FooBar', function($app)
    {
        return new FooBar($app['SomethingElse']);
    });

#### 싱글톤 등록

가끔, 여러분은 딱 한번 해결 되어야하는 무언가를 컨테이너로 바인딩 하고, 다음에 호출할 때 같은 인스턴스가 반환되길 원할 때도 있습니다:

    $this->app->singleton('FooBar', function($app)
    {
        return new FooBar($app['SomethingElse']);
    });

#### 이미 존재하는 인스턴스를 컨테이너로 바인딩

여러분은 또한 `instance` 메서드를 사용하여 이미 존재하는 객체 인스턴스를 컨테이너로 바인딩 해야 할 때도 있습니다. 다음에 호출 할 때 주어진 인스턴스가 항상 컨테이너로 반환 됩니다:

    $fooBar = new FooBar(new SomethingElse);

    $this->app->instance('FooBar', $fooBar);

### 해결

컨테이너에서 무언가를 해결하는 방법에는 여러가지가 있습니다. 첫번째로, `make` 메서드를 사용합니다:

    $fooBar = $this->app->make('FooBar');

두번째로, 컨테이너가 PHP의 `ArrayAccess` 인터페이스를 구현하므로 "배열 조회"를 사용합니다:

    $fooBar = $this->app['FooBar'];

마지막이지만 가장 중요한 방법으로, 여러분은 컨테이너에 의해 해결되는 컨트롤러, 이벤트 리스너, 잡 등록, 필터 그리고 더 많은 의존성을 클래스의 생성자에 "타입-힌트" 할 수 있습니다. 컨테이너는 의존성들을 자동으로 주입합니다:

    <?php namespace App\Http\Controllers;

    use Illuminate\Routing\Controller;
    use App\Users\Repository as UserRepository;

    class UserController extends Controller {

        /**
         * 사용자 리포지토리 인스턴스
         */
        protected $users;

        /**
         * 새로운 컨트롤러 인스턴스를 생성
         *
         * @param  UserRepository  $users
         * @return void
         */
        public function __construct(UserRepository $users)
        {
            $this->users = $users;
        }

        /**
         * 주어진 ID의 사용자를 표시
         *
         * @param  int  $id
         * @return Response
         */
        public function show($id)
        {
            //
        }

    }

<a name="binding-interfaces-to-implementations"></a>
## 구현 클래스에 인터페이스를 주입

### 구체적인 의존성을 주입

서비스 컨테이너의 매우 강력한 기능은 주어진 구현 클래스에 인터페이스를 바인딩 할 수 있는 능력입니다. 예를 들어, 우리의 어플리케이션은 어쩌면 실시간 이벤트를 주고 받는 [Pusher](https://pusher.com) 웹서비스를 통합하고 있을 수도 있습니다. 만약 우리가 Pusher의 PHP SDK를 사요하고 있다면, 우리는 Pusher 클라이언트의 인스턴스를 클래스로 주입 할 수 있습니다:

    <?php namespace App\Handlers\Commands;

    use App\Commands\CreateOrder;
    use Pusher\Client as PusherClient;

    class CreateOrderHandler {

        /**
         * Pusher SDK 클라이언트 인스턴스
         */
        protected $pusher;

        /**
         * 새로운 주문 처리 인스턴스 생성
         *
         * @param  PusherClient  $pusher
         * @return void
         */
        public function __construct(PusherClient $pusher)
        {
            $this->pusher = $pusher;
        }

        /**
         * 주어진 커맨드를 실행
         *
         * @param  CreateOrder  $command
         * @return void
         */
        public function execute(CreateOrder $command)
        {
            //
        }

    }

이 예제에서, 우리가 클래스의 의존성을 주입하는 것은 좋습니다. 하지만, 우리는 Pusher SDK와 단단하게 연결되어 있습니다. 만약 Pusher SDK 메서드가 변경되거나 새로운 이벤트 서비스로 변경하기로 결정 했다면, `CreateOrderHandler` 코드를 변경해야합니다.

### 인터페이스에 프로그램

이벤트 푸쉬의 변경에 대해 `CreateOrderHandler`를 "분리"하려면, `EventPusher` 인터페이스와 `PusherEventPusher` 구현 클래스를 정의 하면 됩니다:

    <?php namespace App\Contracts;

    interface EventPusher {

        /**
         * 모든 클라이언트에게 새로운 이벤트를 푸쉬
         *
         * @param  string  $event
         * @param  array  $data
         * @return void
         */
        public function push($event, array $data);

    }

이 인터페이스의 `PusherEventPusher` 구현 클래스를 작성하고 나면, 아래처럼 서비스 컨테이너에 등록 할 수 있습니다:

    $this->app->bind('App\Contracts\EventPusher', 'App\Services\PusherEventPusher');

이것은 클래스가 `EventPusher` 인터페이스를 필요로 할때, `PusherEventPusher`를 주입해야한다고 컨테이너에게 알려줍니다:

        /**
         * 새로운 주문 처리 인스턴스를 생성
         *
         * @param  EventPusher  $pusher
         * @return void
         */
        public function __construct(EventPusher $pusher)
        {
            $this->pusher = $pusher;
        }

<a name="contextual-binding"></a>
## 상황에 따른 바인딩

가끔 동일한 인터페이스를 사용하는 두개의 클래스가 있을 수도 있습니다. 그렇지만 여러분은 각각의 클래스에 서로 다른 구현 클래스가 주입되길 원할 때도 있습니다. 예를 들어, 시스템이 새로운 주문을 받을때, Pusher보다 [PubNub](http://www.pubnub.com/)을 통하여 이벤트를 보내길 원할 때도 있습니다. 라라벨은 이런 기능을 정의하도록 해주는 간단하고, 유연한 인터페이스를 제공합니다

    $this->app->when('App\Handlers\Commands\CreateOrderHandler')
              ->needs('App\Contracts\EventPusher')
              ->give('App\Services\PubNubEventPusher');

<a name="tagging"></a>
## 태깅

가끔, 바인딩의 특정 "분류" 전체를 해결 해야 할 때도 있습니다. 예를 들어, 많은 `Report` 인터페이스 구현 클래스를 포함한 배열을 받는 보고서 수집기를 개발하고 있다고 해봅시다. `Report` 구현클래스를 등록한 뒤, `tag` 메서드를 사용하여 태그를 명시 할 수 있습니다:

    $this->app->bind('SpeedReport', function()
    {
        //
    });

    $this->app->bind('MemoryReport', function()
    {
        //
    });

    $this->app->tag(['SpeedReport', 'MemoryReport'], 'reports');

이 서비스들이 태그된 다음엔, `tagged` 메서드를 통해 모든 서비스를 쉽게 해결 할 수 있습니다:

    $this->app->bind('ReportAggregator', function($app)
    {
        return new ReportAggregator($app->tagged('reports'));
    });

<a name="practical-applications"></a>
## 실용적인 어플리케이션

라라벨은 어플리케이션의 유연성과 테스트 용이성을 높이기위해 서비스 컨테이너를 사용 할 수 있도록 여러가지 기회를 제공합니다. 하나의 중요한 예제는 컨트롤러 해결 입니다. 모든 클래스는 서비스 컨테이너를 통해 해결됩니다. 이는 여러분이 컨트롤러의 생성자에 의존성을 타입-힌트 할 수 있으며 해당 의존성들은 자동으로 주입됨을 뜻합니다.

    <?php namespace App\Http\Controllers;

    use Illuminate\Routing\Controller;
    use App\Repositories\OrderRepository;

    class OrdersController extends Controller {

        /**
         * 주문 리포지토리 인스턴스
         */
        protected $orders;

        /**
         * 컨트롤러 인스턴스 생성
         *
         * @param  OrderRepository  $orders
         * @return void
         */
        public function __construct(OrderRepository $orders)
        {
            $this->orders = $orders;
        }

        /**
         * 모든 주문 표시
         *
         * @return Response
         */
        public function index()
        {
            $orders = $this->orders->all();

            return view('orders', ['orders' => $orders]);
        }

    }

이 예제에서, `OrderRepository` 클래스는 컨트롤러에 자동으로 주입됩니다. 이는 [유닛 테스트](/docs/5.0/testing)를 할때, 데이터베이스 레이어 상호작용에 고통이 없는 스터빙을 하도록 해주는 "모형" `OrderRepository`가 컨테이너에 바인딩 될 수 있음을 의미합니다.

#### 컨테이너 사용의 다른 예제들

물론, 위에서 언급된 것처럼, 컨트롤러가 라라벨이 서비스 컨테이너를 통해 해결하는 클래스의 전부가 아닙니다. 여러분은 또한 라우트 클로저, 필터, 잡 등록, 이벤트 리스너 그리고 더 많은 곳에 의존성을 타입-힌트 할 수 있습니다. 이러한 곳에서 서비스 컨테이너를 사용하는 예제는, 각각의 문서를 참조하세요.

<a name="container-events"></a>
## 컨테이너 이벤트

#### 해결 리스터 등록

컨테이너는 객재를 해결 할때마다 이벤트를 실행합니다. 여러분은 `resolving` 메서드를 사용하여 이벤트를 수신 할 수 있습니다:

    $this->app->resolving(function($object, $app)
    {
        // 컨테이너가 아무 타입의 객체를 해결 할 때 호출
    });

    $this->app->resolving(function(FooBar $fooBar, $app)
    {
        // 컨테이너가 "FooBar" 타입의 객체를 해결 할 때 호출
    });

해결된 객체가 콜백으로 전달됩니다.
