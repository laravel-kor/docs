# 라라벨 캐쉬어(Cashier)

- [소개](#introduction)
- [설정](#configuration)
- [플랜에 가입](#subscribing-to-a-plan)
- [단일 청구](#single-charges)
- [신용카드 사전정보 없이 가입](#no-card-up-front)
- [구독 변경](#swapping-subscriptions)
- [구독 수량](#subscription-quantity)
- [구독 취소](#cancelling-a-subscription)
- [구독 재개](#resuming-a-subscription)
- [구독 상태 확인](#checking-subscription-status)
- [실패한 구독 처리](#handling-failed-subscription)
- [다른 스트라이프 웹후크 처리](#handling-other-stripe-webhooks)
- [청구서](#invoices)

<a name="introduction"></a>
## 소개

라라벨 캐쉬어는 풍부하고, 유연한 [스트라이프(Stripe)](https://stripe.com) 빌링 서비스의 구독 인터페이스를 제공합니다. 캐쉬어는 우리가 작성하기 두려워하는 거의 모든 빌링 구독 코드의 보일러플레이트를 다룹니다. 기본적인 구독 관리 이외에도, 캐쉬어는 쿠폰, 구독 변경, 구독 "수량", 취소 유예 기간, 또한 청구서를 PDF로 생성 할 수 있습니다.

<a name="configuration"></a>
## 설정

#### 컴포저

첫번째로, `composer.json` 파일에 캐쉬어 패키지를 추가합니다:

    "laravel/cashier": "~5.0" (스트라이프 SDK ~2.0, 그리고 스트라이프 API의 버전이 2015-02-18 또는 그 이후)
    "laravel/cashier": "~4.0" (스트라이프 API의 버전이 2015-02-18 또는 그 이후)
    "laravel/cashier": "~3.0" (스트라이프 API의 버전이 2015-02-16 또는 그 이전)

#### 서비스 프로바이더

다음으로, `app` 설정 파일에 `Laravel\Cashier\CashierServiceProvider`를 등록합니다.

#### 마이그레이션

캐쉬어를 사용하기 전에, 데이터베이스에 몇개의 컬럼들을 추가해야 합니다. 걱정마세요, `cashier:table` 아티즌 커맨드를 사용하여 필요한 컬럼들을 추가해주는 마이그레이션을 생성할 수 있습니다. 예를 들어, users 테이블에 컬럼을 추가하려면, `php artisan cashier:table users` 커맨드를 사용하면 됩니다. 마이그레이션이 생성되고 나면, 간단히 `migrate` 커맨드를 실행하세요.

#### 모델 설정

다음으로, 모델 정의 파일에 `Billable` trait와 적절한 날짜 변경자들을 추가합니다:

    use Laravel\Cashier\Billable;
    use Laravel\Cashier\Contracts\Billable as BillableContract;

    class User extends Model implements BillableContract {

        use Billable;

        protected $dates = ['trial_ends_at', 'subscription_ends_at'];

    }

#### 스트라이프 키

마지막으로, `services.php` 설정 파일에 여러분의 스트라이프 키를 저장합니다.

    'stripe' => [
        'model'  => 'User',
        'secret' => env('STRIPE_API_SECRET'),
    ],

또한 다른 방법으로는 키를 부트스트랩 파일 또는 `AppServiceProvider` 같은 서비스 프로바이더에 저장해도 됩니다.

    User::setStripeKey('stripe-key');

<a name="subscribing-to-a-plan"></a>
## 플랜에 가입

모델 인스턴스가 이미 있다면, 그 사용자를 주어진 스트라이프 플랜에 쉽게  가입 시킬 수 있습니다:

    $user = User::find(1);

    $user->subscription('monthly')->create($creditCardToken);

쿠폰을 적용하여 구독에 가입시키려면, `withCoupon` 메서드를 사용합니다:

    $user->subscription('monthly')
         ->withCoupon('code')
         ->create($creditCardToken);

`subscription` 메서드는 자동으로 스트라이프 구독을 가입하고, 데이터베이스에 스트라이프 고객 ID와 연관된 청구 정보들을 업데이트합니다. 만약 스트라이프에 시험 평가판이 구성되어 있다면, 시험 평가판 종료일 또한 사용자 레코드에 자동으로 설정됩니다.

만약 스트라이프에 시험 평가판이 구성되어 있지 **않다면**, 가입시킨 뒤에 수동으로 시험 평가판 종료일을 설정 해야합니다:

    $user->trial_ends_at = Carbon::now()->addDays(14);

    $user->save();

### 추가적인 사용자 정보를 명시

만약 추가적인 고객의 정보를 명시하고 싶다면, `create` 메서드의 두번째 인수로 전달하여 지정할 수 있습니다:

    $user->subscription('monthly')->create($creditCardToken, [
        'email' => $email, 'description' => 'Our First Customer'
    ]);

스트라이프에서 제공하는 추가적인 필드에 대해 더 알아보려면, 스트라이프의 [고객생성에 관한 문서](https://stripe.com/docs/api#create_customer)를 참고하세요.

<a name="single-charges"></a>
## 단일 청구

만약 구독한 사용자의 신용 카드에서 "일회성 청구"를 하고 싶을 때는, `charge` 메서드를 사용합니다:

    $user->charge(100);

`charge` 메서드는 청구하려는 **통화의 가장 낮은 기준**의 금액을 받습니다. 예를 들어 위의 예제는 사용자의 신용 카드에서 100 센트, 또는 1 달러를 청구합니다.

`charge` 메서드는 두번째 인수에 청구 생성에 필요한 원하는 옵션을 전달해줄수 있는 배열을 받습니다:

    $user->charge(100, [
        'source' => $token,
        'receipt_email' => $user->email,
    ]);

`charge` 메서드는 청구에 실패 할 경우 `false`를 리턴합니다. 이는 일반적으로 청구가 거절됬음을 나타냅니다:

    if ( ! $user->charge(100))
    {
        // The charge was denied...
    }

만약 청구에 성공했다면, 메서드로부터 스트라이프의 전체 응답이 리턴 됩니다.

<a name="no-card-up-front"></a>
## 신용카드 사전정보 없이 가입

만약 어플리케이션이 신용카드의 사전정보없이 무료 시험판을 제공한다면, 모델의 `cardUpFront` 속성을 `false`로 설정하세요:

    protected $cardUpFront = false;

계정을 생성할때, 모델에 시험판 종료일을 설정하는 것을 잊지마세요:

    $user->trial_ends_at = Carbon::now()->addDays(14);

    $user->save();

<a name="swapping-subscriptions"></a>
## 구독 변경

사용자를 새로운 구독으로 변경하려면, `swap` 메서드를 사용합니다.

    $user->subscription('premium')->swap();

만약 사용자가 시험판을 구독하고 있을 경우, 시험판은 정상적으로 유지됩니다. 또한, 구독의 "수량"이 존재 할 경우, 수량 또한 유지됩니다.

<a name="subscription-quantity"></a>
## 구독 수량

때때로 구독은 "수량"에 영향을 받습니다. 예를 들어, 어플리케이션이 한 계정의 사용자마다 한달에 $10를 청구 한다고 하면, `increment`와 `decrement` 메서드를 사용하여 쉽게 구독 수량을 추가시키거나 뺄 수 있습니다.:

    $user = User::find(1);

    $user->subscription()->increment();

    // 현재 구독 수량에 5개 추가
    $user->subscription()->increment(5);

    $user->subscription->decrement();

    // 현재 구독 수량에 5개 감소
    $user->subscription()->decrement(5);

<a name="cancelling-a-subscription"></a>
## 구독 취소

구독을 취소하는 것은 굉장히 쉽습니다:

    $user->subscription()->cancel();

구독이 취소되면, 캐쉬어는 자동으로 데이터베이스의 `subscription_ends_at` 컬럼을 설정합니다. 이 컬럼은 `subscribed` 메서드가 언제 `false`를 반환하기 시작하는지를 숙지하는 것에 사용됩니다. 예를 들어, 고객이 3월 1일에 구독을 취소하여도, 구독은 3월 5일까지 취소될 예정이 아니며, `subscribed` 메서드는 3월 5일까지 `true`를 반환합니다.

<a name="resuming-a-subscription"></a>
## 구독 재개

만약 사용자가 취소한 구독을 재개하려면, `resume` 메서드를 사용합니다:

    $user->subscription('monthly')->resume($creditCardToken);

만약 사용자가 구독을 취소하고, 구독이 완전히 끝나기 전에 다시 재개한다고 해도 비용이 바로 청구 되지 않습니다. 이때 사용자의 구독은 단순히 다시 활성화 되며, 기존의 청구 주기에 따라 청구 됩니다.

<a name="checking-subscription-status"></a>
## 구독 상태 확인

사용자가 현재 구독 중인지를 확인 하려면, `subscribed` 메서드를 사용합니다:

    if ($user->subscribed())
    {
        //
    }

`subscribed` 메서드는 [라우트 미들웨어](/docs/5.0/middleware)에 사용될수 있는 매우 좋은 방법중 하나입니다:

    public function handle($request, Closure $next)
    {
        if ($request->user() && ! $request->user()->subscribed())
        {
            return redirect('billing');
        }

        return $next($request);
    }

또한, `onTrial` 메서드를 사용하여 사용자가 여전히 시험판 기간(만약 해당된다면)인지를  확인 할 수 있습니다:

    if ($user->onTrial())
    {
        //
    }

사용자가 이전에 한번 구독을 했었지만, 취소 했는지 확인 하려면 `cancelled` 메서드를 사용합니다:

    if ($user->cancelled())
    {
        //
    }

또한 구독을 취소한 사용자가 구독 기간이 완전히 취소 될때까지의 "유예 기간" 상태인지 확인 할 수 있습니다. 예를 들어, 사용자가 3월 10일에 종료 예정인 구독을 3월 5일에 취소 하였다면, 해당 유저는 3월 10일까지 "유예 기간" 상태가 됩니다. 해당 기간 동안 `subscribed` 메서드는 `true`를 반환 하는것에 주의하세요.

    if ($user->onGracePeriod())
    {
        //
    }

`everSubscribed` 메서드는 사용자가 어플리케이션에서 지금껏 단 한번도 구독을 한적이 없는지 확인합니다:

    if ($user->everSubscribed())
    {
        //
    }

`onPlan` 메서드는 사용자가 플랜 ID를 기반으로 주어진 플랜을 구독 중인지 확인합니다:

    if ($user->onPlan('monthly'))
    {
        //
    }

<a name="handling-failed-subscription"></a>
## 실패한 구독 처리

만약 사용자의 신용카드가 만료 된다면? 걱정마세요 - 캐쉬어는 사용자의 구독을 쉽게 취소할 수 있는 웹후크 컨트롤러를 포함하고 있습니다. 라우트를 웹후크 컨트롤러로 가르키기만 하세요:

    Route::post('stripe/webhook', 'Laravel\Cashier\WebhookController@handleWebhook');

끝입니다! 실패한 결제는 캡쳐되고 컨트롤러에 의해 처리됩니다. 컨트롤러는 스트라이프가 구독이 실패되었다고 결정했을 때(보통 3번 결제에 실패 할 경우) 사용자의 구독을 취소시킵니다. 이 예제에 있는 `stripe/webhook` URI는 예제일 뿐입니다. 스트라이프 설정에서 해당 URI를 직접 설정해야 합니다.

<a name="handling-other-stripe-webhooks"></a>
## 다른 종류의 스트라이프 웹후크 처리

만약 처리하고 싶은 또 다른 스트라이프 웹후크 이벤트가 있다면, 웹후크 컨트롤러를 확장하면 됩니다. 메서드명은 캐쉬어의 요구된 규칙을 따라야 하며, 특히, 메서드는 `handle`로 시작해야 하며 그 다음은 당신이 처리하려는 스트라이프의 웹후크 이름이 따라와야 합니다. 예를 들어, `invoice.payment_succeeded` 웹후크를 처리하고 싶다면, 컨트롤러에 `handleInvoicePaymentSucceeded` 메서드를 추가해야 합니다.

    class WebhookController extends Laravel\Cashier\WebhookController {

        public function handleInvoicePaymentSucceeded($payload)
        {
            // Handle The Event
        }

    }

> **주의:** 웹후크 컨트롤러는 데이터베이스의 구독 정보를 업데이트 할 뿐만 아니라 스트라이프 API를 통해 구독 역시 취소시킵니다.

<a name="invoices"></a>
## 청구서

`invoices` 메서드를 사용하여 손쉽게 사용자의 청구서를 배열 형태로 조회 할 수 있습니다:

    $invoices = $user->invoices();

사용자의 청구서를 나열할 때, 청구서 정보에 연관된 헬퍼 메서드를 사용하여 표시 할 수도 있습니다:

    {{ $invoice->id }}

    {{ $invoice->dateString() }}

    {{ $invoice->dollars() }}

`downloadInvoice` 메서드를 사용하여 청구서를 PDF로 생성하고 다운로드 할 수 있습니다. 네, 정말 이렇게 쉽습니다:

    return $user->downloadInvoice($invoice->id, [
        'vendor'  => 'Your Company',
        'product' => 'Your Product',
    ]);
