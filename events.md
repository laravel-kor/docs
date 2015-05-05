# 이벤트

- [기본 사용법](#basic-usage)
- [대기중인 이벤트 핸들러](#queued-event-handlers)
- [이벤트 구독자](#event-subscribers)

<a name="basic-usage"></a>
## 기본 사용법

라라벨 이벤트 기능은 어플리케이션에서 이벤트들을 구독하고 수신할수 있도록 간단한 관찰자 구현 클래스를 제공합니다. 이벤트는 보통 `app/Events` 디렉토리에 있으며, 해당 핸들러는 `app/Handlers/Events`에 있습니다.

아티즌 CLI 도구를 사용하여 새로운 이벤트 클래스를 생성 할 수 있습니다:

    php artisan make:event PodcastWasPurchased

#### 이벤트 구독

라라벨 어플리케이션에 포함되어있는 `EventServiceProvider`는 모든 이벤트 핸들러를 등록하기에 알맞은 장소를 제공합니다. `listen` 속성은 모든 이벤트(키)와 그것들의 핸들러(값)를 배열로 포함합니다. 물론, 여러분의 어플리케이션이 요구하는만큼 많은 이벤트들을 이 배열에 추가 할 수 있습니다. 예를 들어, 우리의 `PodcastWasPurchased` 이벤트를 추가해봅시다:

    /**
     * 어플리케이션의 이벤트 핸들러 맵핑
     *
     * @var array
     */
    protected $listen = [
        'App\Events\PodcastWasPurchased' => [
            'App\Handlers\Events\EmailPurchaseConfirmation',
        ],
    ];

이벤트에 대한 핸들러를 생성하려면, `handler:event` 아티즌 CLI 명령을 사용하세요:

    php artisan handler:event EmailPurchaseConfirmation --event=PodcastWasPurchased

물론, 핸들러 또는 이벤트가 필요할때마다 수동으로 `make:event`와 `handler:event` 명령을 실행하는 것은 성가신 일입니다. 대신, `EventServiceProvider`에 핸들러와 이벤트를 추가하고 `event:generate` 명령을 실행하세요. 이 명령은 여러분의 `EventServiceProvider`에 나열되어있는 모든 이벤트 또는 핸들러를 생성해줍니다:

    php artisan event:generate

#### 이벤트 발생 시키기

이제 우리는 `Event` 파사드를 사용하여 이벤트를 발생 시킬 준비가 되었습니다:

    $response = Event::fire(new PodcastWasPurchased($podcast));

`fire` 메서드는 여러분의 어플리케이션이 다음으로 무엇을 할지 제어할때 사용 할 수 있는 응답의 배열을 반환합니다.

여러분은 또한 `event` 헬퍼를 사용하여 이벤트를 발생 시킬 수도 있습니다:

    event(new PodcastWasPurchased($podcast));

#### 클로저 리스너

여러분은 또한 분리된 핸들러 클래스를 만들지 않고서도 이벤트들을 수신 할 수 있습니다. 예를 들어, `EventServiceProvider`의 `boot` 메서드에서, 다음처럼 할 수 있습니다:

    Event::listen('App\Events\PodcastWasPurchased', function($event)
    {
        // 이벤트 처리...
    });

#### 이벤트 전달 중지

때떄로, 하나의 이벤트에서 다른 이벤트들로의 전달을 중지해야 할 때도 있습니다. 여러분의 핸들러에서 `false`를 반환하여 중지 할 수 있습니다:

    Event::listen('App\Events\PodcastWasPurchased', function($event)
    {
        // 이벤트 처리...

        return false;
    });

<a name="queued-event-handlers"></a>
## 대기중인 이벤트 핸들러

이벤트 핸들러를 [대기(queue)](/docs/{{version}}/queues) 시켜야 하나요? 이보다 더 쉬울수 없습니다. 핸들러를 생성할때, 간단히 `--queued` 플래그를 사용하세요:

    php artisan handler:event SendPurchaseConfirmation --event=PodcastWasPurchased --queued

이는 `Illuminate\Contracts\Queue\ShouldBeQueued` 인터페이스를 구현하는 핸들러 클래스를 생성합니다. 이게 다 입니다! 이제 이벤트에 대해 이 핸들러가 호출될 때, 이벤트 디스패처에 의해 자동으로 대기 됩니다.

만약 대기 중인 핸들러가 실행됐을 때 예외가 발생하지 않았다면, 해당 대기 작업은 프로세스 종료 후 자동으로 제거 됩니다. 만약 대기중인 작업의 `delete`와 `release` 메서드를 수동으로 액세스해야 한다면, 그럴 수 있습니다. 대기 핸들러에 기본으로 포함되어 있는 `Illuminate\Queue\InteractsWithQueue` 트레이트가 여러분이 이 메서드들을 엑세스 할 수 있도록 해줍니다:

    public function handle(PodcastWasPurchased $event)
    {
        if (true)
        {
            $this->release(30);
        }
    }

만약 이미 존재하는 핸들러를 대기 핸들러로 변환하고 싶다면, 간단하게 해당 클래스에 `ShouldBeQueued` 인터페이스를 수동으로 추가하세요.

<a name="event-subscribers"></a>
## 이벤트 구독자

#### 이벤트 구독저 정의

이벤트 구독자는 해당 이벤트 클래스 안에서 여러개의 이벤트들을 구독하는 클래스입니다. 구독자는 이벤트 디스패처를 전달하는 `subscribe` 메서드를 정의해야 합니다:

    class UserEventHandler {

        /**
         * 사용자 로그인 이벤트 처리
         */
        public function onUserLogin($event)
        {
            //
        }

        /**
         * 사용자 로그아웃 이벤트 처리
         */
        public function onUserLogout($event)
        {
            //
        }

        /**
         * 구독자의 리스너들을 등록
         *
         * @param  Illuminate\Events\Dispatcher  $events
         * @return array
         */
        public function subscribe($events)
        {
            $events->listen('App\Events\UserLoggedIn', 'UserEventHandler@onUserLogin');

            $events->listen('App\Events\UserLoggedOut', 'UserEventHandler@onUserLogout');
        }

    }

#### 이벤트 구독자 등록

구독자가 정의되고 나면, `Event` 클래스에 등록 될 수 있습니다.

    $subscriber = new UserEventHandler;

    Event::subscribe($subscriber);

여러분은 또한 구독을 해결(resolve)하기위해 [서비스 컨테이너](/docs/{{version}}/container)를 사용 할 수도 있습니다. 이렇게 하려면, 간단하게 `subscribe` 메서드에 구독자 글래스 명을 전달하세요:

    Event::subscribe('UserEventHandler');

