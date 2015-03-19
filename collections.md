# 컬렉션

- [소개](#introduction)
- [기본 사용법](#basic-usage)

<a name="introduction"></a>
## 소개

`Illuminate\Support\Collection` 클래스는 데이터 배열 데이터를 유연하고, 편리하게 사용 할 수 있도록 해줍니다. 예를 들면, 아래의 코드를 확인하세요. `collect` 헬퍼를 사용하여 배열로부터 새로운 컬렉션 인스턴스를 생성합니다:

    $collection = collect(['taylor', 'abigail', null])->map(function($name)
    {
        return strtoupper($name);
    })
    ->reject(function($name)
    {
        return empty($name);
    });


위에서 볼 수 있듯이, `Collection` 클래스는 유연한 맵핑과 배열의 감소를 수행하기 위해 메소드를 체이닝 할 수 있도록 합니다. 기본적으로, 모든 `Collection` 메서드는 완전히 새로운 `Collection` 인스턴스를 반환합니다. 더 많은것을 알아보려면 계속 읽어보세요!

<a name="basic-usage"></a>
## 기본 사용법

#### 컬렉션 생성

위에서 언급한것처럼, `collect` 헬퍼는 주어진 배열의 새로운 `Illuminate\Support\Collection` 인스턴스를 리턴합니다. 또한, `Collection` 클래스의 `make` 메서드를 사용 할 수 도 있습니다:

    $collection = collect([1, 2, 3]);

    $collection = Collection::make([1, 2, 3]);

물론 [엘로퀀트](/docs/5.0/eloquent) 객체의 컬렉션은 항상 `Collection` 인스턴스로 반환됩니다. 그러나, 여러분의 어플리케이션의 어디에서든지 `Collection` 클래스를 자유롭게 사용해도 좋습니다.

#### 컬렉션 탐색

컬렉션에서 사용가능한 모든 메서드(엄청 많은)를 나열하는 대신에, [클래스 API 문서](http://laravel.com/api/master/Illuminate/Support/Collection.html)를 확인하세요!
