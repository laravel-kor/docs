# 큐

- [설정](#configuration)
- [기본적인 사용법](#basic-usage)
- [큐 클로저](#queueing-closures)
- [큐 리스너 실행](#running-the-queue-listener)
- [푸쉬 큐](#push-queues)

<a name="configuration"></a>
## 설정

Laravel 큐 컴포넌트는 서로 다른 큐 서비스의 다양성을 하나의 통합된 API로 제공합니다. 큐는 이메일을 보내는 일과 같이 시간이 많이 걸리는 작업의 처리를 연기하여 어플리케이션의 요청 속도를 크게 향상 시킬 수 있습니다.

큐 설정 파일은 `app/config/queue.php`에 있습니다. 이 파일에서는 프레임워크에 기본적으로 포함되어있는 [Beanstalkd](http://kr.github.com/beanstalkd), [IronMQ](http://iron.io), [Amazon SQS](http://aws.amazon.com/sqs), 그리고 synchronous (로컬사용을 위한) 드라이버들의 연결 설정을 찾을 수 있습니다.

위에 나열된 큐 드라이버들을 사용하기 위해서는 다음의 패키지들이 필요합니다.:

- Beanstalkd: `pda/pheanstalk`
- Amazon SQS: `aws/aws-sdk-php`
- IronMQ: `iron-io/iron_mq`

<a name="basic-usage"></a>
## 기본적인 사용법

`Queue::push` 메소드를 사용하여 큐에 새로운 작업을 추가합니다.:

**큐에 작업 추가**

    Queue::push('SendEmail', array('message' => $message));

`push` 메소드에 전달되는 첫번째 인수는 작업을 실행하는 클래스의 이름입니다. 두번째 인수는 핸들러에 전달될 데이터 배열입니다. 작업 핸들러는 아래와 같이 정의 되어야 합니다.:

**작업 핸들러 정의**

	class SendEmail {

		public function fire($job, $data)
		{
			//
		}

	}

꼭 필요한 메소드는 `Job` 인스턴스와 큐에 추가된 `data` 배열을 전달 받는 `fire` 메소드 입니다.

`fire`가 아닌 다른 메소드를 사용하려면, 작업을 푸쉬 할때 다른 메소드를 명시 해주면 됩니다.:

**사용자 정의 핸들러 메소드 명시**

	Queue::push('SendEmail@send', array('message' => $message));

작업이 끝난 후, 그 작업은 반드시 큐에서 제거되어야 합니다. `Job` 인스턴스의 `delete` 메소드를 사용하여 작업을 제거할 수 있습니다.:

**일이 끝난 작업 제거**

	public function fire($job, $data)
	{
		// Process the job...

		$job->delete();
	}

만약 작업을 다시 큐에 넣어야 한다면, `release` 메소드를 통해 그렇게 할 수 있습니다.:

**큐에 작업을 다시 넣음**

	public function fire($job, $data)
	{
		// Process the job...

		$job->release();
	}

작업을 다시 넣기전에 대기시간(초)을 지정할 수 있습니다.

	$job->release(5);

작업을 진행하는 동안 예외가 발생하면, 그 작업은 자동으로 다시 큐에 넣어집니다. `attempts` 메소드를 사용하여 그 작업이 몇번째 실행을 시도하고 있는지 확인할 수 있습니다.

**몇번째 시도인지 확인**

	if ($job->attempts() > 3)
	{
		//
	}

또한 작업 ID를 액세스 할수도 있습니다.:

**작업 ID 액세스**

	$job->getJobId();

<a name="queueing-closures"></a>
## 큐 클로저

큐에 클로저를 푸쉬 할 수도 있습니ㅏ. 이 방법은 큐가 필요한 테스크를 빠르고 간단하게 푸쉬하는데 매우 편리 합니다.:

**큐에 클로저 푸쉬**

	Queue::push(function($job) use ($id)
	{
		Account::delete($id);

		$job->delete();
	});

> **노트:** 클로저를 큐에 푸쉬할 때, `__DIR__` 와 `__FILE__` 상수가 사용되어선 안됩니다.

Iron.io [푸쉬 큐](#push-queues)를 사용한다면, 추가적인 큐 클로저 예방이 필요합니다. 큐 메세지를 받는 엔드포인트에서 요청이 실제로 Iron.io로부터 오는것인지 토큰을 확인해야 합니다. 예를 들어, 푸쉬 큐의 엔드포인트는 다음과 같아야 합니다: `https://yourapp.com/queue/receive?token=SecretToken`. 다음, 큐 요청을 마셜링 하기전에 먼저 시크릿 토큰의 값부터 확인해야합니다.

<a name="running-the-queue-listener"></a>
## 큐 리스너 실행

Laravel은 새로운 작업이 큐에 추가되는 순간 작동하게 하는 Artisan 태스크를 포함하고 있습니다. `queue:listen` 커맨드를 사용하여 이 태스크를 실행할 수 있습니다.:

**큐 리스너 시작**

	php artisan queue:listen

리스너가 어떤 큐 커넥션을 사용할지 지정할 수 있습니다.:

	php artisan queue:listen connection

한번 작업이 시작되고 나면, 수동으로 정지시킬 때까지 리스너는 계속 실행 됩니다. 반드시 [Supervisor](http://supervisord.org/)같은 프로세스 모니터를 사용하여 큐 리스너가 실행을 멈추지 않도록 합니다.

또한, 각각의 작업을 실행하는데 허용되는 시간(초)을 설정 할 수도 있습니다.:

**작업의 타임아웃 설정 **

	php artisan queue:listen --timeout=60

`queue:work` 메소드를 사용하여 큐의 가장 첫번째 작업만 실행할 수도 있습니다:

**큐의 첫번째 작업 실행**

	php artisan queue:work

<a name="push-queues"></a>
## 푸쉬 큐

푸쉬 큐는 데몬이나 백그라운드 리스너 실행 없이 막강한 Laravel 4 큐 기능들을 사용할 수 있도록 해줍니다. 현재, 푸쉬 큐는 [Iron.io](http://iron.io) 드라이버에서만 지원됩니다. 시작하기전, Iron.io 계정을 생성하고, `app/config/queue.php` 설정 파일에 Iron 인증 정보를 추가합니다.

다음, `queue:subscribe` 아티즌 커맨드를 사용하여 푸쉬된 큐 작업을 받을 URL 엔드포인트를 등록합니다.:

**푸쉬 큐 구독자 등록**

	php artisan queue:subscribe queue_name http://foo.com/queue/receive

이제, Iron 대쉬보드에 로그인 하면 구독자 URL과 새로운 푸쉬 큐를 볼 수 있을 겁니다. 주어진 큐에 원하는 만큼의 URL을 구독할 수 있습니다. 다음, `queue/receive` 엔드포인트 라우트를 생성하고, `Queue::marchal` 메소드를 응답으로 반환하면 됩니다.

	Route::post('queue/receive', function()
	{
		return Queue::marshal();
	});

`marshal` 메소드가 정확한 작업 핸들러 클래스를 알아서 실행 해줍니다. 푸쉬 큐의 작업을 실행하려면, 기존 큐와 같은 `Queue::push` 메소드를 사용하면 됩니다.