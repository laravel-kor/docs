# Laravel에 기여

- [소개](#introduction)
- [풀 리퀘스트](#pull-requests)
- [코딩 가이드라인](#coding-guidelines)

<a name="introduction"></a>
## 소개

Laravel은 누구나 개발과 진행에 기여할 수 있는 자유, 오픈소스 소프트웨어 입니다. Laravel 소스는 현재 프로젝트 포킹과 기여를 병합할 수 있도록 쉬운 메소드를 제공하는 [Github](http://github.com/laravel)에 호스팅되어 있습니다.

<a name="pull-requests"></a>
## 풀 리퀘스트

새로운 기능과 버그에 대한 풀 리퀘스트는 서로 다릅니다. 새로운 기능의 풀 리퀘스트를 보내기 전에는, 제목에 `[Proposal]`를 추가하여 이슈를 먼저 작성해야 합니다. 제안서는 새로운 기능뿐 아니라 구현 아이디어를 서술해야 합니다. 그러면 제안서는 검토된 후, 승인되거나 거부 됩니다. 제안서가 승인되고 나면, 새로운 기능을 구현하는 풀 리퀘스트를 생성할 수 있습니다. 이 가이드라인을 따르지 않은 풀 리퀘스트는 즉시 종료 됩니다.

버그에 대한 풀 리퀘스트는 제안서 이슈 작성 없이 보낼수 있습니다. 만약 Github에 제기된 버그의 해결 방법을 알고 있다면, 제안된 해결 방법을 자세하게 코멘트에 남겨주세요.

매뉴얼의 추가나 수정은 Github의 [documentation repository](https://github.com/laravel/docs)에서 기여할 수 있습니다.

### 기능 요청

Laravel에 추가되었으면 하는 새로운 기능의 아이디어가 있다면, 제목에 `[Request]`를 추가하여 Github에 이슈를 작성하면 됩니다. 그러면 핵심 기여자가 그 요청을 검토합니다.

<a name="coding-guidelines"></a>
## 코딩 가이드라인

Laravel은 [PSR-0](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-0.md)와 [PSR-1](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-1-basic-coding-standard.md) 코딩 표준을 따릅니다. 아래는 이러한 기준 외에도 준수해야할 다른 코딩 표준들입니다.:

- 네임스페이스 선언은 `<?php`와 같은 라인에 있어야 합니다.
- 클래스를 여는 `{`는 클래스명과 같은 라인에 있어야 합니다.
- 함수와 제어구조문을 여는 `{`는 별도의 라인에 있어야 합니다.
- 인터페이스명은 항상 `Interface` 접미사가 붙어야 합니다. (`FooInterface`)
