# 라라벨 Homestead

- [소개](#introduction)
- [포함된 소프트웨어](#included-software)
- [설치 & 설정](#installation-and-setup)
- [매일 사용하는 법](#daily-usage)
- [포트](#ports)
- [Blackfire 프로파일러](#blackfire-profiler)

<a name="introduction"></a>
## 소개

라라벨은 여러분의 로컬 개발 환경을 포함하여 PHP 개발 환경 전체가 즐거울 수 있도록 노력합니다. [Vagrant](http://vagrantup.com)는 가상 컴퓨터를 프로비전하고 관리할 수 있는 간단하고, 멋진 방법을 제공합니다.

라라벨 Homestead는 여러분이 PHP, HHVM, 웹서버, 그리고 다른 서버 소프트웨어들을 여러분의 로컬 컴퓨터에 설치하도록하는 요구없이 훌륭한 개발 환경을 제공하는 공식적인, 프리-패키지된 Vagrant "box" 입니다. 더이상 여러분의 운영 시스템이 난장판이 되는것을 걱정할 필요가 없습니다! Vagrant 박스들은 완전하게 쓰고 버릴 수 있습니다. 만약 무언가가 잘못되었다면, 여러분은 해당 박스를 제거하고 몇분 안에 재생성 할 수 있습니다!

Homestead는 Windows, Mac, Linux 시스템에서 실행되고, Nginx 웹 서버, PHP 5.6, MySQL, Postgres, Redis, Memcached, 그리고 놀라운 라라벨 어플리케이션을 개발할 때 필요한 모든것들을 포함하고 있습니다.

> **주의:** 만약 여러분이 윈도우를 사용한다면, hardware virtualization (VT-x)을 활성화 해야합니다. 버토오 BIOS를 통해 활성화 시킬 수 있습니다.

Homestead는 현재 Vagrant 1.7를 사용하여 만들어졌고 테스트 되었습니다.

<a name="included-software"></a>
## 포함된 소프트웨어

- Ubuntu 14.04
- PHP 5.6
- HHVM
- Nginx
- MySQL
- Postgres
- Node (With Bower, Grunt, and Gulp)
- Redis
- Memcached
- Beanstalkd
- [Laravel Envoy](/docs/{{version}}/envoy)
- [Blackfire 프로파일러](#blackfire-profiler)

<a name="installation-and-setup"></a>
## 설치 & 설정

### VirtualBox / VMware & Vagrant 설치

Homestead 환경을 시작하기 전에, [VirtualBox](https://www.virtualbox.org/wiki/Downloads)와 [Vagrant](http://www.vagrantup.com/downloads.html)를 설치해야 합니다. 이 소프트웨어 패키지 모두 유명한 운영 시스템에서 쉽게 사용할 수 있는 비쥬얼 인스톨러를 제공합니다.

#### VMware

VirtualBox와 더불어, Homestead는 VMware 또한 지원합니다. VMware 공급자를 사용하려면, VMware Fusion / Desktop과 [VMware Vagrant 플러그인](http://www.vagrantup.com/vmware)을 구매해야 합니다. VMware는 기본적으로 더 빠른 공유 폴더 성능을 제공합니다.

### Vagrant 박스 추가

VirtualBox / VMware와 Vagrant가 설치되고 나면, 터미널에서 다음의 커맨드를 사용하여 여러분의 Vagrant 설치에 `laravel/homestead` 박스를 추가해야 합니다. 여러분의 인터넷 속도에 따라 박스를 다운로드 하는데 몇분이 걸릴수 도 있습니다:

    vagrant box add laravel/homestead

만약 이 커맨드가 실패힌다면, 여러분은 아마 전체 URL을 요구하는 오래된 버전의 Vagrant를 갖고 있을 수도 있습니다:

    vagrant box add laravel/homestead https://atlas.hashicorp.com/laravel/boxes/homestead

### Homestead 설치

여러분은 간단하게 리포지토리를 복제하여 Homestead를 설치 할 수 있습니다. Homestead 박스가 여러분의 모든 라라벨 (그리고 PHP) 프로젝트들의 호스트로 제공할 "home" 디렉토리 안의 `Homestead` 폴더로 리포지토리를 복제하는 것을 고려해보세요:

    git clone https://github.com/laravel/homestead.git Homestead

Homestead 리포지토리를 복제하고 나면, `Homestead.yaml` 구성 파일을 생성하기 위해 Homestead 디렉토리에서 `bash init.sh` 커맨드를 실행하세요:

    bash init.sh

`Homestead.yaml` 파일은 여러분의 `~/.homestead` 디렉토리에 위치하게 됩니다.

### 공급자 구성

`Homestead.yaml` 파일의 `provider` 키는 `virtualbox` 또는 `vmware_fusion` 중 어떤 Vagrant 공급자가 사용되어야 하는지를 나타냅니다. 여러분이 선호하는 공급자를 설정하세요.

    provider: virtualbox

### SSH 키 설정

다음으로, `Homestead.yaml` 파일을 수정해야합니다. 이 파일에서, 여러분의 공개 SSH 키 경로와, 메인 컴퓨터와 Homestead 가상 컴퓨터 사이에 공유되길 원하는 폴더들을 구성 할 수 있습니다.

SSH 키가 없나요?? Mac과 Linux에서는, 다음의 커맨드를 사용하여 SSH 키 쌍을 생성할 수 있습니다:

    ssh-keygen -t rsa -C "you@homestead"

Windows에서는, [Git](http://git-scm.com/)을 설치하고 Git에 포함된 `Git Bash` 쉘을 사용하여 위의 커맨드를 사용 할 수 있습니다. 다른 대안으로, [PuTTY](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html)와 [PuTTYgen](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html)을 사용할 수도 있습니다.

SSH 키를 생성하고나면, `Homestead.yaml` 파일의 `authorize` 속성에 해당 키의 경로를 명시하세요.

### 공유 폴더 구성

`Homestead.yaml` 파일에 있는 `folders` 속성은 Homestead 환경과 공유할 모든 폴더들의 목록을 나타냅니다. 이 폴더들에 있는 파일들이 변함에 따라, 여러분의 로컬 컴퓨터와 Homestead 환경 사이의 동기화를 유지합니다. 여러분은 필요한만큼 공유 폴더들을 구성 할 수 있습니다!

[NFS](http://docs.vagrantup.com/v2/synced-folders/nfs.html)를 활성화 시키려면, 여러분의 동기화된 폴더에 간단한 플래그를 추가하세요:

    folders:
        - map: ~/Code
          to: /home/vagrant/Code
          type: "nfs"

### Nginx 사이트 구성

Nginx와 친숙하지 않은가요? 걱정마세요. `sites` 속성은 "도메인"을 Homestead 환경의 폴더로 쉽게 맵핑 해줍니다. 샘플 사이트 구성이 `Homestead.yaml` 파일에 포함되어 있습니다. 또, 여러분은 사이트를 필요한만큼 Homestead 환경에 추가할 수 있습니다. Homestead는 여러분이 개발하는 모든 라라벨 프로젝트를 편리하고, 가상화된 환경을 제공할 수 있도록 해줍니다!

여러분은 `hhvm` 옵션을 `true`로 설정하여 [HHVM](http://hhvm.com)을 사용하여 Homestead 사이트를 만들 수 있습니다:

    sites:
        - map: homestead.app
          to: /home/vagrant/Code/Laravel/public
          hhvm: true

각각의 사이트는 8000 포트를 통해 HTTP를 그리고 44300 포트를 통해 HTTPS로 액세스 될 수 있습니다.

### Bash 별칭

Homestead 박스에 Bash 별칭을 추가하려면, 간단히 `~/.homestead` 디렉토리의 루트에 `aliases` 파일을 추가하세요.

### Vagrant 박스 시작

`Homestead.yaml` 파일을 여러분의 입맛에 맞게 수정하고 나면, Homestead 디렉토리에서 `vagrant up` 커맨드를 실행하세요.

Vagrant는 가상 컴퓨터를 부팅하고, 공유 폴더들과 Nginx 사이트들을 자동으로 구성합니다! 컴퓨터를 제거하려면, `vagrant destroy --force` 커맨드를 사용합니다.

여러분 컴퓨터의 `hosts` 파일에 Nginx 사이트 "도메인"을 추가하는것을 잊지마세요! `hosts` 파일이 로컬 도메인에 대한 요청을 Homestead 환경으로 리디렉트 시킵니다. Mac과 Linux에서, 이 파일은 `/etc/hosts`에 위치해 있습니다. Windows에서, 이 파일은 `C:\Windows\System32\drivers\etc\hosts`에 있습니다. 이 파일에 추가해야하는 라인들은 다음과 같이 생겼습니다:

    192.168.10.10  homestead.app

나열된 IP 주소가 여러분의 `Homestead.yaml` 파일에 설정된 것과 같은지 확인하세요. 여러분의 `hosts` 파일에 도메인을 추가하고 나면, 여러분의 웹 브라우저를 통해 사이트를 액세스할 수 있습니다!

    http://homestead.app

여러분의 데이터베이스에 접속하는 방법을 배우려면, 계속 읽어보세요!

<a name="daily-usage"></a>
## 매일 사용하는 법

### SSH를 통한 접속

여러분은 아마도 SSH를 통하여 Homestead 컴퓨터에 자주 접속해야 할 수도 있으므로, 여러분의 호스트 컴퓨터에서 빠르게 SSH를 통해 Homestead 박스에 접속하는 "별칭"을 생성하는것을 고려해보세요:

    alias vm="ssh vagrant@127.0.0.1 -p 2222"

이 별칭을 생성하고 나면, 여러분은 여러분의 시스템 아무곳에서 간단하게 "vm" 커맨드를 사용하여 Homestead 컴퓨터로 SSH를 통하여 접속할 수 있습니다.

다른 방법으로, 여러분의 Homestead 디렉토리에서 `vagrant ssh` 커맨드를 사용할 수도 있습니다.

### 데이터베이스에 접속

`homestead` 데이터베이스는 기본적으로 MySQL과 Postgres 둘 다 구성되어 있습니다. 좀 더 편리함을 위해, 라라벨의 `local` 데이터베이스 구성이 기본적으로 이 데이터베이스를 사용하도록 설정되어 있습니다.

메인 컴퓨터에서 Navicat 또는 Sequel Pro를 통하여 MySQL이나 Postgres 데이터베이스를 접속하려면, `127.0.0.1`과 33060 (MySQL) 또는 54320 (Postgres) 포트로 접속해야 합니다. 두 데이터베이스의 사용자명과 비밀번호는 `homestead` / `secret` 입니다.

> **주의:** 여러분은 오직 메인 컴퓨터에서 가상 컴퓨터의 데이터베이스를 접속할 때에만 이런 비-표준 포트들을 사용해야합니다. 라라벨은 가상 컴퓨터 _안에서_ 실행되므로 라라벨 데이터베이스 구성 파일에서는 기본 3306과 5432 포트들을 사용해야 합니다.

### 사이트 추가

여러분의 Homestead 환경이 프로비전되고 실행된 뒤, 여러분은 추가적인 Nginx 사이트들을 추가해야 할 수도 있습니다. 여러분은 원하는 만큼의 라라벨 설치를 하나의 Homestead 환경에 실행 할 수 있습니다. 추가하는 방법에는 두가지가 있습니다: 첫번째로, 간단하게 `Homestead.yaml` 파일에 사이트들을 추가하고, Homestead 디렉토리에서 `vagrant provision` 커맨드를 실행할 수 있습니다.

> **주의:** 이 프로세스는 파괴적입니다. `provision` 커맨드를 실행할 때, 이미 존재하는 여러분의 데이터베이스는 파괴되고 재생성됩니다.

다른 대안으로, Homestead 환경에서 사용가능한 `serve` 스크립트를 사용할 수 있습니다. `serve` 스크립트를 사용하려면, Homestead 환경으로 SSH를 통해 접속하여 다음의 커맨드를 실행합니다:

    serve domain.app /home/vagrant/Code/path/to/public/directory 80

> **주의:** `serve` 커맨드를 실행한 다음, 메인 컴퓨터의 `hosts` 파일에 새로운 사이트를 추가하는것을 잊지마세요!

<a name="ports"></a>
## 포트

다음의 포트들은 여러분의 Homestead 환경으로 포워드 됩니다:

- **SSH:** 2222 &rarr; Forwards To 22
- **HTTP:** 8000 &rarr; Forwards To 80
- **HTTPS:** 44300 &rarr; Forwards To 443
- **MySQL:** 33060 &rarr; Forwards To 3306
- **Postgres:** 54320 &rarr; Forwards To 5432

### 포트 추가

원한다면, Vagrant 박스에 프로토콜과 함께 추가적인 포트들을 명시하여 포워드 할 수도 있습니다:

    ports:
        - send: 93000
          to: 9300
        - send: 7777
          to: 777
          protocol: udp

<a name="blackfire-profiler"></a>
## Blackfire 프로파일러

SensioLabs의 [Blackfire 프로파일러](https://blackfire.io)는 여러분의 코드 실행에 대한 RAM, CPU 시간, 디스크 I/O같은 데이터를 수집합니다. Homestead 이 프로파일러를 여러분만의 어플리케이션에서 용이하게 사용할 수 있도록 해줍니다.

적합한 패키지들은 모두 이미 여러분의 Homestead 박스에 설치되어 있으며, `Homestead.yaml` 파일에 간단히 Blackfire **Server** ID와 토큰을 설정하기만하면 됩니다:

    blackfire:
        - id: your-server-id
          token: your-server-token
          client-id: your-client-id
          client-token: your-client-token

Blackfire 자격증명을 구성하고 나면, 여러분의 Homestead 디렉토리에서 `vagrant provision` 커맨드를 사용하여 박스를 다시 프로비전 하세요. 물론, 여러분의 웹 브라우저에 Blackfire 공유 확장을 어떻게 설치하는지 배우려면 [Blackfire 문서](https://blackfire.io/getting-started)를 리뷰하세요.
