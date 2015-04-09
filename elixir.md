# 라라벨 Elixir(엘릭서)

- [소개](#introduction)
- [설치 & 설정](#installation)
- [사용법](#usage)
- [Gulp](#gulp)
- [확장](#extensions)

<a name="introduction"></a>
## 소개

라라벨 엘릭서는 여러분의 라라벨 어플리케이션에 기본적인 [Gulp](http://gulpjs.com) 태스크들을 정의 할 수 있도록 깔끔하고, 매끄러운 API를 제공합니다. 엘릭서는 여러가지의 공통 CSS, JavaScript 전처리기와 함께 테스팅 도구들도 제공합니다.

만약 Gulp와 자산(asset) 편집을 어떻게 시작해야 하는지 혼란스러웠던 적이 있다면, 여러분은 라라벨 엘릭서를 사랑하게 될겁니다!

<a name="installation"></a>
## 설치 & 설정

### Node 설치

엘릭서를 트리거하기 전에, 여러분의 컴퓨터에 Node.js가 설치 되어있어야 합니다.

    node -v

기본적으로 라라벨 홈스테드는 여러분이 필요로 하는 모든것들을 포함하고 있습니다; 하지만, Vagrant를 사용하지 않는다면, 여러분은 [다운로드 페이지](http://nodejs.org/download/)에서 쉽게 Node를 설치 할 수 있습니다. 걱정마세요, 빠르고 간단합니다!

### Gulp

다음으로, 아래와 같이 [Gulp](http://gulpjs.com)를 글로벌 NPM 패키지로 설치 합니다:

    npm install --global gulp

### 라라벨 엘릭서

이제 마지막으로 엘릭서를 설치하는 일만 남았습니다! 새로 설치된 라라벨의 루트 디렉토리에서 여러분은 `package.json` 파일을 찾을 수 있습니다. 이 파일을 PHP 대신 Node의 의존성을 정의한다는 점을 제외하고 `composer.json` 파일과 같다고 생각하세요. 아래의 커맨드를 실행하여 해당 의존성들을 설치 할 수 있습니다:

    npm install

<a name="usage"></a>
## 사용법

이제 엘릭서를 설치했습니다, 여러분은 곧 컴파일과 연결을 사용 할 수 있습니다! 프로젝트의 루트 디렉토리에 있는 `gulpfile.js` 파일은 여러분의 모든 엘릭서 태스크를 포함하고 있습니다.

#### Less 컴파일

```javascript
elixir(function(mix) {
    mix.less("app.less");
});
```

위의 예제에서, 엘릭서는 여러분의 Less 파일이 `resources/assets/less` 디렉토리에 위치해 있다고 가정합니다.

#### 여러개의 Less 파일 컴파일

```javascript
elixir(function(mix) {
    mix.less([
        'app.less',
        'something-else.less'
    ]);
});
```

#### Sass 컴파일

```javascript
elixir(function(mix) {
    mix.sass("app.sass");
});
```

이는 여러분의 Sass 파일들이 `resources/assets/sass` 디렉토리에 위치해 있다고 가정합니다.

기본적으로 엘릭서는, 컴파일 하는데 LibSass 라이브러리를 사용합니다. 더 느리고 기능이 풍부한 Ruby 버전을 사용하는것이 더 유리할수도 있습니다. Ruby와 Sass gem(`gem install sass`)이 설치되어 있다는 가정 하에, 여러분은 다음과 같이 Ruby-모드를 사용 할 수 있습니다:

```javascript
elixir(function(mix) {
    mix.rubySass("app.sass");
});
```

#### 소스맵없이 컴파일

```javascript
elixir.config.sourcemaps = false;

elixir(function(mix) {
    mix.sass("app.scss");
});
```

소스맵은 기본적으로 활성화되어 있습니다. 이를 테면, 컴파일된 각각의 파일마다 같은 디렉토리 안에 짝을 이루는 `*.css.map` 파일을 찾을 수 있습니다. 이 맵핑은 디버깅을 할때 여러분이 컴파일된 스타일시트 셀렉터들을 오리지널 Sass 또는 Less 파일로 추적 할 수 있도록 해줍니다! 이 기능을 비활성 해야한다면, 위의 샘플 코드가 그렇게 해줄겁니다.

#### CoffeeScript 컴파일

```javascript
elixir(function(mix) {
    mix.coffee();
});
```

이는 여러분의 CoffeeScript 파일들이 `resources/assets/coffee` 디렉토리에 위치해 있다고 가정합니다.

#### 모든 Less와 CoffeeScript 컴파일

```javascript
elixir(function(mix) {
    mix.less()
       .coffee();
});
```

#### PHPUnit 테스트 트리거

```javascript
elixir(function(mix) {
    mix.phpUnit();
});
```

#### PHPSpec 테스트 트리거

```javascript
elixir(function(mix) {
    mix.phpSpec();
});
```

#### 스타일시트 결합

```javascript
elixir(function(mix) {
    mix.styles([
        "normalize.css",
        "main.css"
    ]);
});
```

이 메서드에 전달되는 경로는 `resources/css` 디렉토리를 기준으로 합니다.

#### 결합한 스타일시트를 사용자 정의 디렉토리로 저장

```javascript
elixir(function(mix) {
    mix.styles([
        "normalize.css",
        "main.css"
    ], 'public/build/css/everything.css');
});
```

#### 사용자 정의 기본 디렉토리로부터 스타일시트를 결합

```javascript
elixir(function(mix) {
    mix.styles([
        "normalize.css",
        "main.css"
    ], 'public/build/css/everything.css', 'public/css');
});
```

`styles`와 `scripts` 두 메서드의 세번째 인수는 해당 메서드에 전달되는 모든 경로의 기준 디렉토리를 결정 합니다.

#### 디렉토리의 모든 스타일시트를 결합

```javascript
elixir(function(mix) {
    mix.stylesIn("public/css");
});
```

#### 스크립트 결합

```javascript
elixir(function(mix) {
    mix.scripts([
        "jquery.js",
        "app.js"
    ]);
});
```

또 한번, 이는 모든 경로가 `resources/js` 디렉토리 기준이라고 가정합니다.

#### 디렉토리의 모든 스크립트를 결합

```javascript
elixir(function(mix) {
    mix.scriptsIn("public/js/some/directory");
});
```

#### 여러 세트의 스크립트를 결합

```javascript
elixir(function(mix) {
    mix.scripts(['jquery.js', 'main.js'], 'public/js/main.js')
       .scripts(['forum.js', 'threads.js'], 'public/js/forum.js');
});
```

#### 버전 / 파일 해시

```javascript
elixir(function(mix) {
    mix.version("css/all.css");
});
```

이는 캐시-버스팅이 가능하도록 파일명에 유니크 해시를 덧붙힙니다. 예를 들어, 생성된 파일명은 다음처럼 보입니다: `all-16d570a7.css`.

뷰에서, 해시된 적합한 에셋을 로드하기 위해 여러분은 `elixir()` 함수를 사용 할 수 있습니다. 여기 예제가 있습니다:

```html
<link rel="stylesheet" href="{{ elixir("css/all.css") }}">
```

내부적으로, `elixir()` 함수는 포함시켜야 하는 해시된 파일의 이름을 결정합니다. 여러분은 이미 어깨에서 내려놓는 무게가 느껴지지 않습니까?

또한 `version` 메서드에 배열을 전달하여 여러개의 파일에 버전을 부여 할 수도 있습니다:

```javascript
elixir(function(mix) {
    mix.version(["css/all.css", "js/app.js"]);
});
```

```html
<link rel="stylesheet" href="{{ elixir("css/all.css") }}">
<script src="{{ elixir("js/app.js") }}"></script>
```

#### 새로운 위치로 파일을 복사

```javascript
elixir(function(mix) {
    mix.copy('vendor/foo/bar.css', 'public/css/bar.css');
});
```

#### 새로운 위치로 디렉토리 전체를 복사

```javascript
elixir(function(mix) {
    mix.copy('vendor/package/views', 'resources/views');
});
```

#### Browserify 트리거

```javascript
elixir(function(mix) {
    mix.browserify('index.js');
});
```

브라우저에 모듈을 포함하고 싶은가요? EcmaScript 6를 좀더 일찍 사용 하고 싶은가요? 빌트인 JSX 트랜스포머가 필요한가요? 그렇다면, [Browserify](http://browserify.org/), `browserify` 엘릭서 태스크와 함께 작업을 멋지게 처리하세요.

이 태스크는 여러분의 스크립트가 `resources/js`에 있다고 가정하지만, 기본 디렉토리를 재정의 할 수도 있습니다.

#### 메서드 체이닝

물론, 여러분은 엘릭서의 거의 모든 메서드를 체이닝 할 수 있습니다:

```javascript
elixir(function(mix) {
    mix.less("app.less")
       .coffee()
       .phpUnit()
       .version("css/bootstrap.css");
});
```

<a name="gulp"></a>
## Gulp

이제 여러분은 어떤 태스크를 실행해야하는지 들었습니다, 여러분은 이제 커맨드 라인에서 Gulp를 트리거하기만 하면 됩니다.

#### 등록된 모든 태스크를 한번 실행

    gulp

#### 변경되는 자산을 감시

    gulp watch

#### 스크립트만 컴파일

    gulp scripts

#### 스타일만 컴파일

    gulp styles

#### 테스트 감시와 PHP 클래스 변경 감시

    gulp tdd

> **주의:** 모든 태스크는 개발 환경이라고 간주하여, 축소는 제외됩니다. 프로덕션 환경에서는, `gulp --production`을 사용하세요.

<a name="extensions"></a>
## 사용자 정의 태스크와 확장

때때로, 여러분이 만든 Gulp 태스크를 엘릭서로 연결하길 원할수도 있습니다. 아마도 여러분은 엘릭서가 혼합하고 감시를 할 수 있는 스페셜 기능이 필요할지도 모릅니다. 문제 없습니다!

예를 들어, 호출될 때 문장을 프린트하는 간단한 태스크가 있다고 상상해봅시다.

```javascript
gulp.task("speak", function() {
    var message = "Tea...Earl Grey...Hot";

    gulp.src("").pipe(shell("say " + message));
});
```

충분히 쉽습니다. 커맨드 라인에서 여러분은 물론 `gulp speak`를 호출하여 해당 태스크를 트리거 할 수 있습니다. 한편, 엘릭서에 해당 태스크를 추가하려면, `mix.task()` 메서드를 사용합니다:

```javascript
elixir(function(mix) {
    mix.task('speak');
});
```

이게 전부입니다! 이제 Gulp가 실행 될 때마다, 여러분의 사용자 정의 "speak" 태스크는 여러분이 믹스한 엘릭서 태스크들과 함께 실행 됩니다. 한개 또는 그 이상의 파일들이 수정 됐을 때, 여러분의 사용자 정의 태스크가 다시 트리거 되도록 추가적인 감시자를 등록하려면, 두번째 인수에 정규표현식을 전달하면 됩니다.

```javascript
elixir(function(mix) {
    mix.task('speak', 'app/**/*.php');
});
```

두번째 인수를 추가함으로써, 우리는 "app/" 디렉토리 안에 PHP 파일이 저장될 때마다 "speak" 태스크가 트리거 되도록 엘릭서에게 지시했습니다.


좀 더 많은 유연성을 위해, 여러분은 완전한 엘릭서 확장을 만들수 있습니다. 이전의 "speak" 예제를 사용하여 여러분은 다음과 같이 확장을 작성 할 수 있습니다:

```javascript
var gulp = require("gulp");
var shell = require("gulp-shell");
var elixir = require("laravel-elixir");

elixir.extend("speak", function(message) {

    gulp.task("speak", function() {
        gulp.src("").pipe(shell("say " + message));
    });

    return this.queueTask("speak");

 });
```

Gulp 파일에서 참조할 이름을 첫번째 인수로 전달하여 엘릭서의 API를 `확장`한 것과 Gulp 태스크를 생성하는 콜백 함수를 사용한 점을 주목하세요.

이전과 같이, 여러분의 사용자 정의 태스크를 감시하려면, 감시자를 등록하세요.

```javascript
this.registerWatcher("speak", "app/**/*.php");
```

위의 라인은 `app/**/*.php` 정규표현식과 일치하는 파일이 하나라도 수정되면, `speak` 태스크를 트리거합니다.

이게 전부입니다! 여러분은 이 코드를 Gulp 파일 가장 윗부분에 위치시키거나, 사용자 정의 태스크 파일로 추출할 수도 있습니다. 만약 여러분이 후자를 선택하다면, 다음과 같이 간단하게 여러분의 Gulp 파일에 포함시키세요:

```javascript
require("./custom-tasks")
```

포함 됐습니다! 이제 여러분은 태스크를 혼합하여 사용 할 수 있습니다.

```javascript
elixir(function(mix) {
    mix.speak("Tea, Earl Grey, Hot");
});
```

위에서 추가한 부분과 함께 Gulp가 트리거 될때마다 피카르(Picard)는 차(tea)를 요청할 것 입니다.
