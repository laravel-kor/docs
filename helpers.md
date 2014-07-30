# 헬퍼 함수

* [배열(Arrays)](#arrays)
* [경로(Paths)](#paths)
* [문자열(Strings)](#strings)
* [URLs](#urls)
* [그 외(Miscellaneous)](#miscellaneous)

<a name="arrays"></a>
## 배열(array)

### array_add
`array_add` 함수는 중복되는 키가 없다면 주어진 키/값 쌍을 배열에 추가한다.

```php
$array = array('foo' => 'bar');

$array = array_add($array, 'key', 'value');
```

### array_divide
`array_divide` 함수는 두 배열을 반환한다, 하나는 키를, 다른 하나는 값을 반환하는데 둘 다 원본 배열에 있는 값이다.

```php
$array = array('foo' => 'bar');

list($keys, $values) = array_divide($array);
```
### array_dot
`array_dot` 함수는 다차원 배열을 1차원화 시키는데 "점" 을 사용해서 깊이를 표한한다.

```php
$array = array('foo' => array('bar' => 'baz'));

$array = array_dot($array);

// array('foo.bar' => 'baz');
```
### array_except
The `array_except` 메서드는 주어진 키/값 쌍을 배열에서 제거한다.

```php
$array = array_except($array, array('keys', 'to', 'remove'));
```
### array_fetch
The `array_fetch` 메서드는 선택된 nested 요소를 포함하는 일차원 배열을 반환한다.
> 여러 배열에서 주어진 키 값에 맞는 값만 합쳐서 배열로 반환한다.

```
$array = array(array('name' => 'Taylor'), array('name' => 'Dayle'));

var_dump(array_fetch($array, 'name'));

// array('Taylor', 'Dayle');
```

### array_first
The `array_first` 메서드는 주어진 참 테스트를 통과한 배열의 첫 요소를 반환한다.

```php
$array = array(100, 200, 300);

$value = array_first($array, function($key, $value)
{
    return $value >= 150;
});
```

기본값을 세 번째 매개변수를 통해 설정될 수 있다:

```
$value = array_first($array, $callback, $default);
```

### array_flatten
The `array_flatten` 메서드는 다차원 배열을 일차원 배열로 반환한다.
> 여러 배열이 제일 말단에 있는 값만을 끌어모아 새 배열을 만든다.

```
$array = array('name' => 'Joe', 'languages' => array('PHP', 'Ruby'));

$array = array_flatten($array);

// array('Joe', 'PHP', 'Ruby');
```

### array_forget
The `array_forget` 메서드는 "점" 기호를 사용해 주어진 키/값 쌍을 깊이 nested 된 배열에서  제거한다.

```php
$array = array('names' => array('joe' => array('programmer')));

$array = array_forget($array, 'names.joe');
```
### array_get
The `array_get` 메서드는 "점"을 이용해서 깊이 nested 된 배열로부터 주어진 값을 가져온다.

```php
$array = array('names' => array('joe' => array('programmer')));

$value = array_get($array, 'names.joe');
```
### array_only
The `array_only` 메서드는 배열로부터 오직 지정된 키/ 값만을 반환한다.

```php
$array = array('name' => 'Joe', 'age' => 27, 'votes' => 1);

$array = array_only($array, array('name', 'votes'));
```
### array_pluck
The `array_pluck` 메서드는 배열에서 주어진 키/값 쌍의 목록을 당겨(pluck)온다.

```php
$array = array(array('name' => 'Taylor'), array('name' => 'Dayle'));

$array = array_pluck($array, 'name');

// array('Taylor', 'Dayle');
```

### array_pull
The `array_pull` 메서드는 주어진 키/값 쌍을 배열로부터 반환하고, 또한 그것들 제거한다.
>그냥 제거하는게 아니라 제거한 값을 반환까지 해주는 메서드.

```
$array = array('name' => 'Taylor', 'age' => 27);

$name = array_pull($array, 'name');
```
### array_set
The `array_set` 메서드는 "점" 을 사용해서 깊이 nested 된 배열 안에 값을 지정한다.

```php
$array = array('names' => array('programmer' => 'Joe'));

array_set($array, 'names.editor', 'Taylor');
```
### array_sort
The `array_sort` 메서드는 주어진 클로우저에 결과로 배열을 정렬한다.

```php
$array = array(
    array('name' => 'Jill'),
    array('name' => 'Barry'),
);

$array = array_values(array_sort($array, function($value)
{
    return $value['name'];
}));
```

다음 함수는 php5의 함수다.
>[array_values](http://php.shukuma.net/php/function.array-values.html) 주어진 배열에서 키를 모두 숫자로 만들어 숫자-값 형태로 배열을 반환
>[array_key](http://php.shukuma.net/php/function.array-keys.html) 주어진 배열에서 최상단의 키만 숫자-키 형태로 배열을 반환. 혹은 주어진 조건에 일치하는 키가 있는 배열변호를  숫자-배열번호 형태로 반환.

### head
배열의 첫 번째 요소를 반환한다. PHP 5.3.x 에서 메소드 채이닝 하는데 유용하다.

```php
$first = head($this->returnsArray('foo'));
```
### last
배열의 마지막 요소를 반환한다. 마찬가지로 메소드 체이닝에 유용하다.

```php
$last = last($this->returnsArray('foo'));
```

<a name="paths"></a>
## 경로

### app_path
`application` 디렉토리 까지 전체 주소 경로를 가져온다.

### base_path
어플리케이션 설치 루트 디렉토리 까지 전체 주소 경로를 가져온다.

### public_path
`public` 디렉토리까지 전체 주소 경로를 가져온다.

### storage_path
`application/storage` 디렉토리 까지 전체 주소 경로를 가져온다.

<a name="strings"></a>
## 문자열

### camel_case
주어진 문자열을 `camelCase` 화 해준다.
>카멜케이스는 단어 사이의 `_` 표시를 없애고 대신 단어 앞을 대문자화 시키는 것.

```
$camel = camel_case('foo_bar');

// fooBar 이 것 처럼 뒤에 foo 와 bar 사이를 없애고 bar 를 Bar 로 바꾸었다.
```

### class_basename
네임 스페이스 이름 없이 주어진 클래스의 클래스 이름을 가져온다.

```php
$class = class_basename('Foo\Bar\Baz');

// Baz
```
### e
UTF-8 지원 상태로 주어진 문자열을 `htmlentites` 요소로 바꿉니다.

```
$entities = e('<html>foo</html>');
```
아래는 php의 htmlentities 함수 예제입니다.

><?php
$str = "A 'quote' is <b>bold</b>";

>출력: A 'quote' is &lt;b&gt;bold&lt;/b&gt;
echo htmlentities($str);

>출력: A &#039;quote&#039; is &lt;b&gt;bold&lt;/b&gt;
echo htmlentities($str, ENT_QUOTES);
?>
출처 [클릭](http://php.shukuma.com/php/function.htmlentities.html)

### ends_with
Determine if the given haystack ends with a given needle.
주어진 건초더미가 주어진 바늘로 끝나는지 판단한다.

```
$value = ends_with('This is my name', 'name');
```
### snake_case
주어진 문자열을 `snake_case` 화 시킨다..
>camel case 화 된 단어 사이에 `_` 표시를 넣고 뒷 단어 첫글자를 소문자화한다.

```
$snake = snake_case('fooBar');
// foo_bar
```

### starts_with
주어진 건초더미가 주어진 바늘로 시작하는지 판단한다.

```
$value = starts_with('This is my name', 'This');
```
### str_contains
주어진 건초더미가 주어진 바늘을 포함하는지 판단한다.

```
$value = str_contains('This is my name', 'my');
```
### str_finish
건초더미 끝에 주어진 바늘을 붙인다. Remove any extra instances.

```
$string = str_finish('this/string', '/');
// this/string/
```

### str_is
주어진 문자열이 주어진 패턴을 따르는지 판단한다. Asterisks may be used to indicate wildcards.

```
$value = str_is('foo*', 'foobar');
```
### str_plural
문자열을 복수형으로 바꾼다. (영어만 가능하다).

```
$plural = str_plural('car');
```
### str_random
주어진 길이의 랜덤한 문자열을 생성한다.

```
$string = str_random(40);
```
### str_singular
주어진 문자열을 단수로 바꾼다. (영어만 가능하다).

```
$singular = str_singular('cars');
```
### studly_case
주어진 문자열을 `StudlyCase`화 시킨다..
> 카멜케이스는 맨 앞 글자는 그대로 두었지만 studlycase 는 맨 앞 글자로 대문자화 시킨다.

```
$value = studly_case('foo_bar');

// FooBar
```

### trans
주어진 언어 라인으로 번역한다. Alias of `Lang::get`.

```
$value = trans('validation.required'):
```
### trans_choice
Tranlate a given language line with inflection. Alias of `Lang::choice`.

```
$value = trans_choice('foo.bar', $count);
```
<a name="urls"></a>
## URLs

### action
주어진 컨트롤러 액션으로 향하는 URL을 생성한다.

```
$url = action('HomeController@getIndex', $params);
```
### asset
asset으로 향하는 URL을 생성한다.

```
$url = asset('img/photo.jpg');
```
### link_to
주어진 URL로 향하는 HTML 링크를 생성한다.

```
echo link_to('foo/bar', $title, $attributes = array(), $secure = null);
```
### link_to_asset
주어진 asset으로 향하는 HTML 링크를 생성한다.

```
echo link_to_asset('foo/bar.zip', $title, $attributes = array(), $secure = null);
```
### link_to_route
주어진 라우트로 향하는 HTML 링크를 생성한다.

```
echo link_to_route('route.name', $title, $parameters = array(), $attributes = array());
```
### link_to_action
주어진 컨트롤러 액션으로 향하는 HTML 링크를 생성한다.

```
echo link_to_action('HomeController@getIndex', $title, $parameters = array(), $attributes = array());
```
### secure_asset
주어진 asset으로 향하며 HTTPS를 사용하도록 하는 HTML 링크를 만든다.

```
echo secure_asset('foo/bar.zip', $title, $attributes = array());
```
### secure_url
HTTPS 주어진 경로로 향하며 HTTPS를 사용하는 전체 경로 URL을 생성한다.

```
echo secure_url('foo/bar', $parameters = array());
```
### url
주어진 경로로 향하는 전체 경로 URL을 생성한다.

```
echo url('foo/bar', $parameters = array(), $secure = null);
```
## 그 외

<a name="miscellaneous"></a>
### csrf_token
현재 CSRF 토큰 값을 가져온다.

```
$token = csrf_token();
```
### dd
주어진 변수를 Dump 하고 스크립트의 실행을 종료시킨다.

```
dd($value);
```
### value
만약 주어진 값이 클로우저면, 클로우저가 반환한 값을 반환한다. 그게 아니면, 값을 반환한다.

```
$value = value(function() { return 'bar'; });
```

### with
주어진 오브젝트를 반환한다. PHP 5.3.x 에서 메서드 체이닝 생성자에 유용하다.
Return the given object. Useful for method chaining constructors in PHP 5.3.x.

```
$value = with(new Foo)->doWork();
```
