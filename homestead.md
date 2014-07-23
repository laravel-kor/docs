# Laravel Homestead

- [Introduction](#introduction)
- [Included Software](#included-software)
- [Installation & Setup](#installation-and-setup)
- [Daily Usage](#daily-usage)
- [Ports](#ports)

<a name="introduction"></a>
## 소개

라라벨은 로컬 개발환경을 포함한 PHP 개발환경 전체를 즐겁게 하기 위해서 노력합니다. Vagrant는 가상머신들을 관리하고 프로비져닝 (Provision) 하는 간단하고 세련된 (Elegant) 방법을 제공합니다.

라라벨 홈스테드는 당신의 로컬 머신에 PHP, 웹서버, 그리고 다른 서버 소프트웨어를 설치를 요구하지 않으면서 아주 멋진 (Wonderful) 개발 환경을 제공하는 공식 적인 Vagrant 프리패키지 박스입니다. 당신의 운영체제를 더 이상 더럽히는 걱정을 안해도 됩니다! Vagrant 박스들은 완전히 (completely) 일회용 (disposal) 입니다. 뭔가 잘못되면, 몇 분안에 제거하고 다시 박스를 추가 할 수 있습니다!

홈스테드는 Windows, Mac, 그리고 리눅스에서 작동하며 Nginx 웹서버, PHP 5.5, MySQL, Postgres, Reds, Memcached 등 멋진 라라벨 어플리케이션을 개발하기에 필요한 다른 모든 어플리케이션을 포함합니다.

홈스테드는 Vagrant 1.6에서 테스트 됬습니다.

<a name="included-software"></a>
## Included Software

- Ubuntu 14.04
- PHP 5.5
- Nginx
- MySQL
- Postgres
- Node (With Bower, Grunt, and Gulp)
- Redis
- Memcached
- Beanstalkd
- [Laravel Envoy](/docs/ssh#envoy-task-runner)
- Fabric + HipChat Extension

<a name="installation-and-setup"></a>
## Installation & Setup

### VirtualBox & Vagrant 설치하기

홈스테드 환경을 실행하기 전에 [VirtualBox](https://www.virtualbox.org/wiki/Downloads)와  [Vagrant](http://www.vagrantup.com/downloads.html) 꼭 설치해야 합니다. VirtualBox와 Vagrant는 모든 유명한 운영체제에 대해서 쉽게 사용가능한 비주얼 인스톨러를 제공합니다.



### Adding The Vagrant Box

Virtualbox와 Vagrant가 설치되면, 아래의 명령어를 실행해서 `laravel/homestead` 박스를 Vagrant에 추가합니다. 사용자의 인터넷 환경에 따라 몇 분 정도 걸립니다.

	vagrant box add laravel/homestead

### 홈스테드 저장소 (Repository) 복제하기

Vagrant에 박스를 추가하고 나면, 홈스테드 저장소를 복제하거나 다운로드 해야 합니다. 홈스테드는 당신의 모든 라라벨 프로젝트에서 사용할수 있으므로 당신의 라라벨 프로젝트들을 저장하는 디렉토리에 복제 할것을 고려하십시오.

	git clone https://github.com/laravel/homestead.git Homestead

### SSH Key 셋업

다음으로 홈스테드 저장소에 포함된 Homestead.yaml을 편집해야 합니다. 이 파일로 SSH 공개키의 Path와 당신의 메인 컴퓨터와 홈스테드 가상머신 사이에 공유된 폴더들을 지정할 수 있습니다.

공개키가 없으시나요? Mac과 Linux에서는 SSH 공개키를 다음 명령어로 만들 수 있습니다. 윈도우에서는 [Git](http://git-scm.com/) 설치하고 Git에 포함된 `Git bash shel`l로 위의 명렁어를 실행할수 있습니다. 다른 방법으로는, [PuTTY](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html) 와 [PuTTYgen](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html) 사용하셔도 됩니다.

공개키를 만들고 나면, 공개키의 path를 Homestead.yaml 파일의 authorize 속성에 명시해줍니다.

###공유폴더 설정

Homestead.yaml 파일에 있는 folders 속성은 당신의 홈스테드 환경과 공유하고 싶은 모든 폴더를 열거합니다. Folders 속성에 열거된 폴더안의 파일들이 변할때마다 해당 파일들은 당신의 로컬 머신과 홈스테드와 싱크 됩니다. 원하는 만큼의 공유폴더를 설정할 수 있습니다!

### Nginx 사이트들 설정

Nginx에 대해서 잘 모르시나요? 문제 없습니다. sites 속성은 도메인을 홈스테드 폴더에 쉽게 연결해줍니다. Homestead.yaml 파일에 샘플 사이트 설정이 포함되어 있습니다. 원하는 만큼 많은 사이트를 홈스테드에 추가할 수 있습니다. 작업하고 있는 모든 라라벨 프로젝트에 홈스테드는 사용하기 쉬운 가상환경을 제공합니다.

### Bash Aliases

홈스테드 박스에 Bash aliases를 추가하려면, 간단하게 홈스테드 루트폴더에 aliases 파일을 추가합니다.

### Vagrant 박스 실행

원하시는데로 Homestead.yaml을 변경하였으면, 터미널에서 vagrant up을 홈스테드 디렉토리에서 실행합니다. Vagrant가 가상 머신을 부트하고 공유 폴더와 Nginx 사이트들을 자동으로 설정합니다.

당신의 컴퓨터에 Nginx 사이트드들에 대한 도메인을 추가하는것을 잊지마십시오! Hosts 파일은 당신의 요청을 로컬 머신을 로컬 도메인에서 홈스테드 환경으로 돌려줍니다. 윈도우에서 호스트 파일은 C:\Windows\System32\drivers\etcs\hosts 에 위치되어 있습니다. 추가하는 줄들은 아래와 같이 보일겁니다.

`127.0.01 homestead.app`

Hosts 파일에 도메인을 추가하고 나면 포트 8000로 웹브라우저에서 접속할수 있습니다!

`http://hometead.app:8000`

데이타 베이스 연결에 대해서 배우려면 계속 읽으십시오!

<a name="daily-usage"></a>
## Daily Usage

### SSH 접속하기

홈스테드 환경에 SSH로 접속하려면 포트 Homestead.yaml에 명시한 SSH키로 포트 2222를 통해서 127.0.0.1로 접속해야 합니다. 간편하게 홈스테으 디렉토리에서 vagrant ssh를 실행하셔도 됩니다.
더 편하게 사용 하려면, 아래의 alias를 ~/.bash_aliases 또는 ~/.bash_profile에 추가해주면 도움이 됩니다.

	alias vm='ssh vagrant@127.0.0.1 -p 2222'

### 데이타베이스에 접속하기

홈스테드 데이타베이스는 기본적으로 MySQL과 Postgres가 설정되어 있습니다. 좀 더 편리함을 위해서, 라라벨의 로컬 데이타베이스는 기본 홈스테드 데이타 베이스를 사용하도록 설정되어 있습니다.

메인 머신에서 Navicat 또는 Sequel Pro로 MySQL또는 Postgres 데이타베이스에 접속하려면 127.0.01로 포트 33060 (MySQL) 또는 54320 (Postgres)로 접속합니다. 사용자 이름과 비밀번호는 MySQL과 Postgres 둘 다 homestead/secruet 입니다.

> **노트:** 33060과 54320 같은 비표준 포트들은 메인 머신에서만 사용합니다. 라라벨은 가상머신 안에서 작동하기 때문에 라라벨 데이타베이스 설정 파일에서는 기본 포트 3306과 5432를 사용합니다.

### 사이트 추가하기

홈스테드 환경이 프로비젼되고 작동하면, 추가적인 사이트들 추가를 원할수도 있습니다. 하나의 홈스테드 환경에서 원하는 만큼 라라벨 을 설치할수 있습니다. 두 가지 방법이 있습니다. 첫번째는, Homestaed.yaml 파일에 사이트들을 추가하고 vagrant provision을 실행할수 있습니다.

다른 방법 으로는, 홈스테드 환경에 포함된 serve 스크립트를 사용할수 있습니다. Serve 스크립트를 사용하기 위해서는 홈스테드 환경에 SSH로 접속하고 아래 명령어를 실행합니다.

	serve domain.app /home/vagrant/Code/path/to/public/directory

> **노트:** serve 커맨드를 실행하고 나서, 메인머신 hosts파일에 도메인을 추가하는것을 잊지마십시오!

<a name="ports"></a>
## Ports

The following ports are forwarded to your Homestead environment:

- **SSH:** 2222 -> Forwards To 22
- **HTTP:** 8000 -> Forwards To 80
- **MySQL:** 33060 -> Forwards To 3306
- **Postgres:** 54320 -> Forwards To 5432
