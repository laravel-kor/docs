# 기여 가이드

- [버그 신고](#bug-reports)
- [코어 개발 토론](#core-development-discussion)
- [어떤 브랜치?](#which-branch)
- [보안 취약점](#security-vulnerabilities)
- [코딩 스타일](#coding-style)

<a name="bug-reports"></a>
## 버그 신고

적극적인 협력을 장려하기 위해, 라라벨은 버그 신고 뿐만 아니라, 풀 요청을 강력하게 장려하고 있습니다. "버그 신고" 또한 실패한 유닛 테스트를 포함하여 폼 요청을 통해 전송 될 수 있습니다.

그렇지만, 여러분이 버그를 신고 할때, 해당 이슈는 제목과 명확한 설명을 포함해야 합니다. 또한 가능한 많은 관련된 정보와 해당 이슈를 보여주는 샘플 코드를 포함해야 합니다. 버그 신고의 목표는 여러분과 다른 사람이 쉽게 해당 버그를 복제하고 수정 개발하도록 하는 것 입니다.

기억하세요, 버그 신고는 같은 문제를 다른 사람과 협력하여 함께 해당 문제를 해결 할 수 있도록 신고하는 것입니다. 버그를 신고하면 자동으로 활동적인 사람이나 다른 사람이 버그에 뛰어들어 수정할거라고 기대 하지 마세요. 버그 신고를 하는건 여러분과 다른 사람들이 문제를 해결하기 위한 시발점을 제공합니다.

라라벨 소스 코드는 깃헙에서 관리되며, 각각의 라라벨 프로젝트들이 저장소에 있습니다:

- [Laravel Framework](https://github.com/laravel/framework)
- [Laravel Application](https://github.com/laravel/laravel)
- [Laravel Documentation](https://github.com/laravel/docs)
- [Laravel Cashier](https://github.com/laravel/cashier)
- [Laravel Envoy](https://github.com/laravel/envoy)
- [Laravel Homestead](https://github.com/laravel/homestead)
- [Laravel Homestead Build Scripts](https://github.com/laravel/settler)
- [Laravel Website](https://github.com/laravel/laravel.com)
- [Laravel Art](https://github.com/laravel/art)

<a name="core-development-discussion"></a>
## 코어 개발 토론

버그, 새 기능 그리고 기존 기능의 이행에 관련된 토론은 `#laravel-dev` IRC 채널(Freenode)에서 열립니다. 라라벨의 개발자 Taylor Otwell은 보통 평일 오전 8시부터 오후 5시 (UTC-06:00 또는 America/Chicago)까지 채널에 있으며, 이따금 다른 시간대에도 있습니다.

`#laravel-dev` IRC 채널은 모두를 위해 열려 있습니다. 채널에 합류하는 토론 참여자나 그냥 눈팅만 하는 분들 모두 환영합니다!

<a name="which-branch"></a>
## 어떤 브랜치?

**모든** 버그 수정은 최신의 안정 브랜치에 전송되어야 합니다. 버그 수정은 곧 릴리즈 될 코드에 있는 기능을 수정하지 않는 이상 **절대** `master` 브랜치로 전송되어서는 안됩니다.

 현재의 라라벨 릴리즈와 이전 버전에서 완벽하게 호환되는 **중요하지 않은** 기능은 최신 안정 브랜치로 전송 될 수도 있습니다.

**중요한** 새로운 기능은 항상 다음 라라벨 릴리즈를 포함하고 있는 `master` 브랜치로 전송되어야 합니다.

여러분의 기능이 중요한지 중요하지 않은지 확실하지 않다면, `#laravel-dev` IRC 채널에서 (Freenode) Taylor Otwell에게 물어보세요.

<a name="security-vulnerabilities"></a>
## 보안 취약점

만약 라라벨에서 보안 취약점을 발견했다면, <a href="mailto:taylor@laravel.com">taylor@laravel.com</a>로 Taylor Otwell에게 이메일을 보내세요. 모든 보안 취약점은 신속하게 해결 될 것입니다.

<a name="coding-style"></a>
## 코딩 스타일

라라벨은 [PSR-4](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-4-autoloader.md)과 [PSR-1](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-1-basic-coding-standard.md) 코딩 표준을 따릅니다. 이러한 표준과 추가적으로 다음의 코딩 표준을 따라야 합니다.

- 클래스 네임스페이스 선언은 반드시 `<?php`와 같은 라인에 있어야 합니다.
- 클래스 열림 `{` 은 반드시 클래스 명과 같은 라인에 있어야 합니다.
- 함수와 제어 구조는 반드시 Allman 스타일 중괄호를 사용 해야 합니다.
- 들여쓰기는 탭으로, 정렬은 스페이스를 사용합니다.
