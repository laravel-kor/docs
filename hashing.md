# 해시화

- [소개](#introduction)
- [기본 사용법](#basic-usage)

<a name="introduction"></a>
## 소개

라라벨 `Hash` 파사드는 사용자 비밀번호를 저장하는데 안전한 Bcrypt 해싱을 제공합니다. 만약 여러분이 라라벨 어플리케이션에 포함되어있는 `AuthController` 컨트롤러를 사용하고 있다면, 사용자로부터 제공된 해시되지 않은 버전에 대하여 Bcrypt 비밀번호를 증명하는 일을 처리 해줄겁니다.

마찬가지로, 라라벨에 포함된 사용자 `Registrar` 서비스는 비밀번호를 해시하여 저장하는 적절한 bcrypt` 함수를 호출합니다.

<a name="basic-usage"></a>
## 기본 사용법

#### Bcrypt를 사용하여 비밀번호 해시화

    $password = Hash::make('secret');

또한 `bcrypt` 헬퍼 함수를 사용할 수도 있습니다:

    $password = bcrypt('secret');

#### 해시화된 비밀번호에 대하여 확인

    if (Hash::check('secret', $hashedPassword))
    {
        // The passwords match...
    }

#### 비밀번호가 다시 해시화 될 필요가 있는지 확인

    if (Hash::needsRehash($hashed))
    {
        $hashed = Hash::make('secret');
    }
