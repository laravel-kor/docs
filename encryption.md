# 암호화

- [소개](#introduction)
- [기본 사용법](#basic-usage)

<a name="introduction"></a>
## 소개

라라벨은 Mcrypt PHP 확장을 통해 강력한 AES 암호화를 제공합니다

<a name="basic-usage"></a>
## 기본 사용법

#### 값을 암호화

    $encrypted = Crypt::encrypt('secret');

> **주의:** `config/app.php` 파일의 `key` 옵션에 16, 24, 또는 32 자릿수의 랜덤 문자열을 설정해야야 합니다. 그렇지 않을 경우 암호화된 값이 안전하지 않습니다.

#### 값을 복호화

    $decrypted = Crypt::decrypt($encryptedValue);

#### 암호(cipher) 및 모드 설정

또한 암호기가 사용하는 암호와 모드를 설정 할 수 있습니다:

    Crypt::setMode('ctr');

    Crypt::setCipher($cipher);
