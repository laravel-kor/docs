# 헬퍼 함수

- [배열](#arrays)
- [경로](#paths)
- [라우팅](#routing)
- [문자열](#strings)
- [URL들](#urls)
- [기타](#miscellaneous)

<a name="arrays"></a>
## 배열

### array_add

`array_add` 함수는 배열에 주어진 키가 이미 존재하지 않을 경우 주어진 키 /값 쌍을 추가 합니다.

    $array = ['foo' => 'bar'];

    $array = array_add($array, 'key', 'value');

### array_divide

`array_divide` 함수는 두개의 배열을 반한합니다, 하나는 오리지널 배열의 키를 포함하고있고, 다른 하나는 값을 포함하고 있습니다.

    $array = ['foo' => 'bar'];

    list($keys, $values) = array_divide($array);

### array_dot

`array_dot` 함수는 다차원 배열을 깊이를 표현하는 "점" 표기법을 사용하는 단일 레벨 배열로 평면화합니다.

    $array = ['foo' => ['bar' => 'baz']];

    $array = array_dot($array);

    // ['foo.bar' => 'baz'];

### array_except

`array_except` 메서드는 주어진 키 / 값 쌍을 배열로부터 제거합니다.

    $array = array_except($array, ['keys', 'to', 'remove']);

### array_fetch

`array_fetch` 메서드는 선택된 중첩 요소를 포함하는 평면화된 배열을 반환합니다.

    $array = [
        ['developer' => ['name' => 'Taylor']],
        ['developer' => ['name' => 'Dayle']]
    ];

    $array = array_fetch($array, 'developer.name');

    // ['Taylor', 'Dayle'];

### array_first

`array_first` 메서드는 주어진 진실 테스트를 통과하는 배열의 첫번째 요소를 반환합니다.

    $array = [100, 200, 300];

    $value = array_first($array, function($key, $value)
    {
        return $value >= 150;
    });

디폴트 값이 새번째 매개 변수로 전달 될 수도 있습니다:

    $value = array_first($array, $callback, $default);

### array_last

`array_last` 메서드는 주어진 진실 테스트르르 통과하는 배열의 마지막 요소를 반환합니다.

    $array = [350, 400, 500, 300, 200, 100];

    $value = array_last($array, function($key, $value)
    {
        return $value > 350;
    });

    // 500

디폴트 값이 새번째 매개 변수로 전달 될 수도 있습니다:

    $value = array_last($array, $callback, $default);

### array_flatten

`array_flatten` 메서드는 다차원 배열을 단일 레벨로 평면화 합니다.

    $array = ['name' => 'Joe', 'languages' => ['PHP', 'Ruby']];

    $array = array_flatten($array);

    // ['Joe', 'PHP', 'Ruby'];

### array_forget

`array_forget` 메서드는 "점" 표기법을 사용하여 깊게 중첩된 배열로부터 주어진 키 / 값 쌍을 제거합니다.

    $array = ['names' => ['joe' => ['programmer']]];

    array_forget($array, 'names.joe');

### array_get

`array_get` 메서드는 "점" 표기법을 사용하여 깊게 중첩된 배열로부터 주어진 값을 조회 합니다.

    $array = ['names' => ['joe' => ['programmer']]];

    $value = array_get($array, 'names.joe');

    $value = array_get($array, 'names.john', 'default');

> **주의:** 배열 대신 객체에서 `array_get`같은 기능을 사용하려면 `object_get`을 사용하세요.

### array_only

`array_only` 메서드는 배열로부터 오직 지정된 키 / 값 쌍을 반환합니다.

    $array = ['name' => 'Joe', 'age' => 27, 'votes' => 1];

    $array = array_only($array, ['name', 'votes']);

### array_pluck

`array_pluck` 메서드는 배열로부터 주어진 키 / 값 쌍의 목록을 뽑아냅니다.

    $array = [['name' => 'Taylor'], ['name' => 'Dayle']];

    $array = array_pluck($array, 'name');

    // ['Taylor', 'Dayle'];

### array_pull

`array_pull` 메서드는 배열로부터 주어진 키 / 값 쌍을 반환하고, 배열에서 삭제합니다.

    $array = ['name' => 'Taylor', 'age' => 27];

    $name = array_pull($array, 'name');

### array_set

`array_set` 메서드는 "dot" 표기법을 사용하여 깊게 중첩된 배열에서 값을 설정합니다.

    $array = ['names' => ['programmer' => 'Joe']];

    array_set($array, 'names.editor', 'Taylor');

### array_sort

`array_sort` 메서드는 배열을 주어진 클로저의 결과를 기준으로 정렬합니다.

    $array = [
        ['name' => 'Jill'],
        ['name' => 'Barry']
    ];

    $array = array_values(array_sort($array, function($value)
    {
        return $value['name'];
    }));

### array_where

주어진 클로저를 사용하여 배열을 필터링합니다.

    $array = [100, '200', 300, '400', 500];

    $array = array_where($array, function($key, $value)
    {
        return is_string($value);
    });

    // Array ( [1] => 200 [3] => 400 )

### head

배열의 첫번째 요소를 반환합니다.

    $first = head($this->returnsArray('foo'));

### last

배열의 마지막 요소를 반환합니다. 메서드 체이닝을 사용할 때 유용합니다.

    $last = last($this->returnsArray('foo'));

<a name="paths"></a>
## 경로

### app_path

`app` 디렉토리에 대한 전체 경로를 조회합니다.

    $path = app_path();

### base_path

어플리케이션이 설치된 루트에 대한 전체 경로를 조회합니다.

### config_path

`config` 디렉토리에 대한 전체 경로를 조회합니다.

### public_path

`public` 디렉토리에 대한 전체 경로를 조회합니다.

### storage_path

`storage` 디렉토리에 대한 전체 경로를 조회합니다.

<a name="routing"></a>
## 라우팅

### get

라우터에 새로운 GET 라우트를 등록합니다.

    get('/', function() { return 'Hello World'; });

### post

라우터에 새로운 POST 라우트를 등록합니다.

    post('foo/bar', 'FooController@action');

### put

라우터에 새로운 PUT 라우트를 등록합니다.

    put('foo/bar', 'FooController@action');

### patch

라우터에 새로운 PATCH 라우트를 등록합니다.

    patch('foo/bar', 'FooController@action');

### delete

라우터에 새로운 DELETE 라우트를 등록합니다.

    delete('foo/bar', 'FooController@action');

### resource

라우터에 새로운 RESTful 라우트를 등록합니다.

    resource('foo', 'FooController');

<a name="strings"></a>
## 문자열

### camel_case

주어진 문자열을 `camelCase`로 변환합니다.

    $camel = camel_case('foo_bar');

    // fooBar

### class_basename

주어진 클래스에서 네임스페이스명을 제외한 클래스명을 조회합니다.

    $class = class_basename('Foo\Bar\Baz');

    // Baz

### e

주어진 문자열에 UTF-8 지원과 함께 `htmlentities`를 실행합니다.

    $entities = e('<html>foo</html>');

### ends_with

주어진 문자열이 특정 문자열로 끝나는지 확인합니다.

    $value = ends_with('This is my name', 'name');

### snake_case

주어진 문자열을 `snake_case`로 변환합니다.

    $snake = snake_case('fooBar');

    // foo_bar

### str_limit

문자열의 길이를 제한합니다.

    str_limit($value, $limit = 100, $end = '...')

예제:

    $value = str_limit('The PHP framework for web artisans.', 7);

    // The PHP...

### starts_with

주어진 문자열이 특정 문자열로 시작하는지 확인합니다.

    $value = starts_with('This is my name', 'This');

### str_contains

주어진 문자열이 특정 문자열을 포함하는지 확인합니다.

    $value = str_contains('This is my name', 'my');

### str_finish

문자열에 주어진 문자열의 단일 인스턴스를 추가합니다. 다른 인스턴스들은 제거합니다.

    $string = str_finish('this/string', '/');

    // this/string/

### str_is

주어진 문자열이 주어진 패턴과 일치하는지 확인합니다. 별표는 와일드카드를 표현하는데 사용됩니다.

    $value = str_is('foo*', 'foobar');

### str_plural

문자열을 복수 형태로 변환합니다. (영어만 가능)

    $plural = str_plural('car');

### str_random

주어진 길이의 랜덤 문자열을 생성합니다.

    $string = str_random(40);

### str_singular

문자열을 단수 형태로 변환합니다. (영어만 가능)

    $singular = str_singular('cars');

### str_slug

주어진 문자열로부터 URL에 친숙한 "slug"를 생성합니다.

    str_slug($title, $separator);

예제:

    $title = str_slug("Laravel 5 Framework", "-");

    // laravel-5-framework

### studly_case

주어진 문자열을 `StudlyCase`로 변환합니다.

    $value = studly_case('foo_bar');

    // FooBar

### trans

주어진 언어 라인으로 번역합니다. `Lang::get`의 별칭입니다.

    $value = trans('validation.required'):

### trans_choice

어형 변화와 함께 주어진 라인으로 번역합니다.`Lang::choice`의 별칭입니다.

    $value = trans_choice('foo.bar', $count);

<a name="urls"></a>
## URL들

### action

주어진 컨트롤러 액션에 대한 URL을 생성합니다

    $url = action('HomeController@getIndex', $params);

### route

주어진 이름이 명시된 라우트에 대한 URL을 생성합니다.

    $url = route('routeName', $params);

### asset

자산에 대한 URL을 생성합니다.

    $url = asset('img/photo.jpg');

### secure_asset

자산에 대한 URL을 HTTPS를 사용하여 생성합니다.

    echo secure_asset('foo/bar.zip', $title, $attributes = []);

### secure_url

주어진 경로에 대한 전체 URL을 HTTPS를 사용하여 생성합니다.

    echo secure_url('foo/bar', $parameters = []);

### url

주어진 경로에 대한 전체 URL을 생성합니다.

    echo url('foo/bar', $parameters = [], $secure = null);

<a name="miscellaneous"></a>
## 기타

### csrf_token

현재의 CSRF 토큰 값을 조회합니다.

    $token = csrf_token();

### dd

주어진 변수를 Dump하고 스크립트의 실행을 종료합니다.

    dd($value);

### elixir

버전이 명시된 Elixir 파일의 경로를 조회합니다.

    elixir($file);

### env

환경 변수의 값을 조회하거나, 디폴트 값을 반환합니다.

    env('APP_ENV', 'production')

### event

이벤트를 실행합니다.

    event('my.event');

### value

만약 주어진 값이 `Closure`라면, `Closure`에 의해 반환된 값을 반환합니다. 그렇지 않을 경우 값을 반환합니다

    $value = value(function() { return 'bar'; });

### view

주어진 뷰 경로의 뷰 인스턴스를 반환합니다.

    return view('auth.login');

### with

주어진 객체를 반환합니다.

    $value = with(new Foo)->doWork();
