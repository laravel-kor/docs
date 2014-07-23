# 아티즌 CLI

- [소개](#introduction)
- [사용법](#usage)

<a name="introduction"></a>
## 소개

아티즌(Artisan)은 Laravel에 포함되어 있는 커맨드라인 인터페이스의 이름입니다. 아티즌은 어플리케이션을 개발 하는 동안 도움이 되는 몇가지의 커맨드를 제공합니다. 아티즌은 강력한 심포니 콘솔 컴포넌트에 의해 구동 됩니다.

<a name="usage"></a>
## 사용법

`list` 커맨드를 사용하여 이용 가능한 모든 아티즌 커맨드 리스트를 볼 수 있습니다.:

**이용 가능한 모든 커맨드 나열**

    php artisan list

모든 커맨드는 이용 가능한 인수와 옵션을 보여주고 설명해주는 "도움말" 화면을 포함하고 있습니다. 간단히 커맨드 이름 앞에 `help`를 붙여 도움말 화면을 볼 수 있습니다.:

**커맨드의 도움말 화면 보기**

	php artisan help migrate

`--env` 스위치를 사용하여 커맨드를 실행하는 동안 사용될 구성 환경을 지정 할 수 있습니다.:

**구성 환경 지정**

	php artisan migrate --env=local

또한 `--version` 옵션을 사용하여 현재 설치된 Laravel의 버전을 볼 수도 있습니다.:

**현재 Laravel의 버전 표시**

	php artisan --version
