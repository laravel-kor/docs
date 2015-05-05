# 파일시스템 / 클라우드 스토리지

- [소개](#introduction)
- [구성](#configuration)
- [기본 사용법](#basic-usage)
- [사용자 정의 파일시스템](#custom-filesystems)

<a name="introduction"></a>
## 소개

라라벨은 굉장히 훌륭한 파일시스템 추상화를 제공합니다. Frank de Jonge의 [Flysystem](https://github.com/thephpleague/flysystem) PHP 패키지에게 감사합니다. 라라벨 플라이시스템 통합은 로컬 파일시스템, 아마존 S3, 그리고 Rackspace 클라우드 스토리지와의 작업을 위한 드라이버의 사용을 간단하게 제공합니다. 더 나아가, 각각의 시스템에 대한 API가 동일하므로 이러한 스토리지 옵션 사이의 전환은 놀라울 정도로 간단합니다!

<a name="configuration"></a>
## 구성

파일시스템 구성 파일은 `config/filesystems.php`에 있습니다. 이 파일에서 여러분의 모든 "디스크"를 구성 할 수 있습니다. 각각의 디스크는 개별적인 스토리지 드라이버와 스토리지 로케이션을 나타냅니다. 각각의 지원하는 드라이버의 구성 예제가 구성 파일에 포함되어 있습니다. 그러므로, 간단하게 여러분의 스토리지 환경 설정 및 자격 증명을 반영하여 구성을 설정하세요!

S3 또는 Rackspace 드라이버를 사용하기 전에, 여러분은 컴포저를 통해 전용 드라이버를 설치해야 합니다:

- Amazon S3: `league/flysystem-aws-s3-v2 ~1.0`
- Rackspace: `league/flysystem-rackspace ~1.0`

물론, 원하는 만큼의 디스크를 구성 할 수 있고, 같은 드라이버를 사용 하는 여러개의 디스크를 사용 할 수도 있습니다.

`local` 드라이버를 사용할 때, 구성 파일에 정의되는 모든 파일 기능들은 `root` 디렉토리 기준이라는 점에 유의하세요. 기본적으로, 이 값은 `storage/app` 디렉토리에 정의되어 있습니다. 따라서, 다음의 메서드는 `storage/app/file.txt`에 파일을 저장합니다:

    Storage::disk('local')->put('file.txt', 'Contents');

<a name="basic-usage"></a>
## 기본 사용법

`Storage` 파사드가 여러분의 구성된 디스크와 상호 작용 하는데 사용될 수 있습니다. 또는, 라라벨 [서비스 컨테이너](/docs/{{version}}/container)를 통해 해결되는 아무런 클래스에 `Illuminate\Contracts\Filesystem\Factory` contract를 타입-힌트할 수도 있습니다.

#### 특정 디스크를 조회

    $disk = Storage::disk('s3');

    $disk = Storage::disk('local');

#### 파일이 존재하는지 확인

    $exists = Storage::disk('s3')->exists('file.jpg');

#### 기본 디스크에서 메서드를 호출

    if (Storage::exists('file.jpg'))
    {
        //
    }

#### 파일의 컨텐츠를 조회

    $contents = Storage::get('file.jpg');

#### 파일의 컨텐츠 작성

    Storage::put('file.jpg', $contents);

#### 파일 컨텐츠 앞 부분에 컨텐츠 추가

    Storage::prepend('file.log', 'Prepended Text');

#### 파일 컨텐츠 끝 부분에 컨텐츠 작성

    Storage::append('file.log', 'Appended Text');

#### 파일 삭제

    Storage::delete('file.jpg');

    Storage::delete(['file1.jpg', 'file2.jpg']);

#### 파일을 새로운 위치로 복사

    Storage::copy('old/file1.jpg', 'new/file1.jpg');

#### 파일을 새로운 위치로 이동

    Storage::move('old/file1.jpg', 'new/file1.jpg');

#### 파일 크기 조회

    $size = Storage::size('file1.jpg');

#### 마지막 수정 시간 조회 (UNIX)

    $time = Storage::lastModified('file1.jpg');

#### 디렉토리 안의 모든 파일 조회

    $files = Storage::files($directory);

    // 재귀...
    $files = Storage::allFiles($directory);

#### 디렉토리 안의 모든 디렉토리 조회

    $directories = Storage::directories($directory);

    // 재귀...
    $directories = Storage::allDirectories($directory);

#### 디렉토리 생성

    Storage::makeDirectory($directory);

#### 디렉토리 삭제

    Storage::deleteDirectory($directory);

<a name="custom-filesystems"></a>
## 사용자 정의 파일시스템

라라벨의 플라이시스템 통합은 여러 "드라이버들"을 위한 드라이버를 제공합니다. 하지만, 플라이시스템은 이들에 제한되어있지 않고 많은 다른 스토리지 시스템들을 위한 어댑터들을 갖고 있습니다. 여러분은 라라벨에서 추가적인 어댑터들을 사용하길 원한다면 사용자 정의 드라이버를 생성할 수 있습니다. 걱정마세요, 그렇게 어렵지 않습니다!

사용자 정의 파일시스템을 설정하려면, 여러분은 `DropboxFilesystemServiceProvider`와 같은 서비스 공급자를 생성해야 합니다. 공급자의 `boot` 메서드에서, `Illuminate\Contracts\Filesystem\Factory` contract를 주입할 수 있으며 주입된 인스턴스의 `extend` 메서드를 호출 할 수 있습니다. 다른 방법으로, `Disk` 파사드의 `extend` 메서드를 사용 할 수도 있습니다.

`extend` 메서드의 첫번째 인수는 드라이버의 이름이며 두번째는 `$app`과 `$config` 변수들을 받는 클로저입니다. 확인자 클로저는 `League\Flysystem\Filesystem` 인스턴스를 반환해야만 합니다.

> **주의:** $config 변수는 `config/filesystems.php`에 지정된 디스크의 값을 이미 포함하고 있습니다.

#### Dropbox 예제

    <?php namespace App\Providers;

    use Storage;
    use League\Flysystem\Filesystem;
    use Dropbox\Client as DropboxClient;
    use League\Flysystem\Dropbox\DropboxAdapter;
    use Illuminate\Support\ServiceProvider;

    class DropboxFilesystemServiceProvider extends ServiceProvider {

        public function boot()
        {
            Storage::extend('dropbox', function($app, $config)
            {
                $client = new DropboxClient($config['accessToken'], $config['clientIdentifier']);

                return new Filesystem(new DropboxAdapter($client));
            });
        }

        public function register()
        {
            //
        }

    }
