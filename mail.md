# 메일

- [설정](#configuration)
- [기본적인 사용법](#basic-usage)
- [인라인 첨부 사용](#embedding-inline-attachments)
- [메일 대기(큐)](#queueing-mail))
- [메일 & 로컬 개발](#mail-and-local-development)

<a name="configuration"></a>
## 설정

Laravel은 유명한 [SwiftMailer](http://swiftmailer.org) 라이브러리를 바탕으로 깨끗하고 간단한 API를 제공 합니다. 메일 설정 파일은 `app/config/mail.php` 이며, SMTP 호스트, 포트, 자격증명 뿐만 아니라 라이브러리에 의해 전송되는 모든 메시지에 사용되는 글로벌 `from` 주소를 변경할 수 있도록 해주는 옵션들을 포함하고 있습니다. 당신이 원하는 어떤 SMTP 서버를 사용해도 좋습니다. PHP `mail` 함수를 사용하여 메일을 보내려면, 설정 파일의 `driver`를 `mail`로 변경하시기 바랍니다. `sendmail` 드라이버 또한 사용 가능합니다.

<a name="basic-usage"></a>
## 기본적인 사용법

`Mail::send` 메소드를 사용하여 이메일을 메시지를 보낼수 있습니다.:

    Mail::send('emails.welcome', $data, function($message)
  	{
  		$message->to('foo@example.com', 'John Smith')->subject('Welcome!');
  	});

`send` 메소드에 전달되는 첫번째 인수는 이메일의 바디로 사용될 뷰 파일의 이름입니다. 두번째 인수는 뷰에 전달 될 `$data` 이며, 세번째는 이메일 메시지에 다양한 옵션을 지정하게 해주는 클로저입니다.

> **노트:** `$message` 변수는 항상 이메일 뷰에 전달되며, 인라인 첨부를 사용할 수 있게 해줍니다. 그러므로 뷰 페이로드에 `message` 이름의 변수 전달은 피하는게 좋습니다.

또한 HTML 뷰를 추가로 사용할 수 있는 플레인 텍스트 뷰를 지정할 수도 있습니다.:

	Mail::send(array('html.view', 'text.view'), $data, $callback);

또는 `html` 나 `text` 키를 사용하여 한개의 뷰 타입만 지정할 수도 있습니다:

	Mail::send(array('text' => 'view'), $data, $callback);

참조나 첨부파일 같은 다른 옵션 또한 이메일 메시지에 지정할 수 있습니다.:

	Mail::send('emails.welcome', $data, function($message)
	{
		$message->from('us@example.com', 'Laravel');

		$message->to('foo@example.com')->cc('bar@example.com');

		$message->attach($pathToFile);
	});

메시지에 파일을 첨부할 때는 파일의 MIME 타입이나 표시명을 지정할 수 있습니다.:

	$message->attach($pathToFile, array('as' => $display, 'mime' => $mime));

> **노트:** `Mail::send` 클로저에 전달되는 메시지 인스턴스는 SwiftMailer의 메시지 클래스를 확장하므로 클래스의 모든 메소드를 호출하여 메시지를 작성할 수 있도록 해줍니다.

<a name="embedding-inline-attachments"></a>
## 인라인 첨부 사용

이메일에 이미지를 포함하는 것은 성가신 일입니다. 하지만 Laravel은 이메일에 이미지를 첨부하고 적절한 CID를 조회할 수 있는 편리한 방법을 제공합니다.

**이메일 뷰에 이미지 첨부**

	<body>
		Here is an image:

		<img src="<?php echo $message->embed($pathToFile); ?>">
	</body>

**이메일 뷰에 미가공 데이터 첨부**

	<body>
		Here is an image from raw data:

		<img src="<?php echo $message->embedData($data, $name); ?>">
	</body>

`$message` 변수는  `Mail` 클래스에 의해 항상 이메일 뷰에 전달 된다는 것을 주의 하십시오.

<a name="queueing-mail"></a>
## 메일 대기(큐)

e-mail 메시지를 보내는 일은 어플리케이션의 응답 속도를 대폭 길게 만드므로, 많은 개발자들이 e-mail을 백그라운드에서 보낼 수 있도록 대기 시키는 방법을 선택합니다. Laravel은 내장된 [통합 큐 API](/docs/queues)를 사용해 이를 쉽게 할 수 있도록 해줍니다. 간단하게 `Mail` 클래스의 `queue` 메소드를 사용하여 메일을 대기 시킬 수 있습니다.:

**메일 대기시키기**

	Mail::queue('emails.welcome', $data, function($message)
	{
		$message->to('foo@example.com', 'John Smith')->subject('Welcome!');
	});

또한 `later` 메소드를 사용하여 메일을 몇 초 뒤에 보낼 지 명시 할수도 있습니다.:

	Mail::later(5, 'emails.welcome', $data, function($message)
	{
		$message->to('foo@example.com', 'John Smith')->subject('Welcome!');
	});

어떤 큐나, "튜브"에 메일 메시지를 푸쉬 할 것인지 지정하려면, `queueOn`과 `laterOn` 메소드를 사용하여 그렇게 할 수 있습니다.:

	Mail::queueOn('queue-name', 'emails.welcome', $data, function($message)
	{
		$message->to('foo@example.com', 'John Smith')->subject('Welcome!');
	});

<a name="mail-and-local-development"></a>
## 메일 & 로컬 개발

이메일을 보내는 어플리케이션을 개발할 경우, 로컬이나 개발 환경에서 메시지 보내는 기능을 비활성화 시키는 것이 때로는 바람직 할 때가 있습니다. 그렇게 하려면, `Mail::pretend` 메소드를 호출하거나, `app/config/mail.php` 설정 파일의 `pretend` 옵션을 `true`로 설정 하면 됩니다. 메일러가 `pretend` 모드일 경우, 메시지를 수신자에게 보내는 것 대신, 어플리케이션의 로그 파일에 기록합니다.

**Pretend 메일 모드 활성화**

	Mail::pretend();