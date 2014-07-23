# 엘로퀀트 ORM

- [소개](#introduction)
- [기본적인 사용법](#basic-usage)
- [대량 할당(Mass Assignment)](#mass-assignment)
- [삽입(Insert), 수정(Update), 삭제(Delete)](#insert-update-delete)
- [소프트 삭제](#soft-deleting)
- [타임스탬프(Timestamps)](#timestamps)
- [쿼리 스코프](#query-scopes)
- [관계성(Relationships)](#relationships)
- [Querying Relations](#querying-relations)
- [Eager 로딩](#eager-loading)
- [관계된 모델 삽입](#inserting-related-models)
- [부모 타임스탬프 터치](#touching-parent-timestamps)
- [피벗 테이블과 작업](#working-with-pivot-tables)
- [컬렉션](#collections)
- [접근자 & 변경자](#accessors-and-mutators)
- [날짜 변경자](#date-mutators)
- [모델 이벤트](#model-events)
- [모델 옵저버](#model-observers)
- [배열 / JSON 으로 변환](#converting-to-arrays-or-json)

<a name="introduction"></a>
## 소개

Laravel에 포함된 엘로퀀트 ORM은 데이터베이스 작업을 위한 아름답고 간단한 액티브레코드 구현을 제공합니다. 각 테이터베이스 테이블은 테이블과 상호작용 하는데 사용되는 해당 "모델"을 가지고 있습니다.

시작하기 전에, `app/config/database.php`에서 데이터베이스 커넥션을 구성했는지 확인하십시오.

<a name="basic-usage"></a>
## 기본적인 사용법

시작하려면, 일단 엘로퀀트 모델을 생성합니다. 모델은 보통 `app/models` 디렉토리에 있지만, `composer.json` 파일에 따라 오토로드 될 수 있는 어떠한 곳에 놓아도 상관없습니다.

**엘로퀀트 모델 정의**

    class User extends Eloquent {}

`User` 엘로퀀트 모델에 어떤 테이블을 사용할지 알려주지 않았다는 것에 주목하십시오. 다른 테이블명이 명백하게 지정되지 않는한 클래스의 소문자, 복수형 이름이 테이블명으로 쓰이게 됩니다. 그러므로 이 경우, 엘로퀀트는 `User` 모델이 `users` 테이블에 레코드를 저장하도록 취하고 있습니다. 모델에서 `table` 속성을 사용하여 사용자 정의 테이블을 지정할 수 있습니다.:

	class User extends Eloquent {

		protected $table = 'my_users';

	}

> **노트:** 엘로퀀트는 또한 각각의 테이블이 `id`란 이름의 기본키 컬럼을 가지고 있다고 추정합니다. `primaryKey` 속성을 사용하여 이 규칙을 치환 할 수 있습니다. 마찬가지로, `connection` 속성을 사용하여 모델을 활용 할 때 사용할 데이터베이스 접속 명을 치환 할 수 있습니다.

이렇게 모델이 정의 되고 나면, 테이블에 레코드를 조회하거나 생성할 준비가 끝났습니다. 기본적으로 테이블에 `updated_at`와 `created_at` 컬럼을 추가해야 합니다. 만약 이 컬럼들이 자동으로 관리되지 않도록 하려면, 모델에 `$timestamp` 속성을 `false`로 설정하면 됩니다.

**모델의 모든 레코드 조회**

	$users = User::all();

**기본키를 통해 단일 레코드 조회**

	$user = User::find(1);

	var_dump($user->name);

> **노트:** 엘로퀀트 모델 역시 [쿼리 빌더](/docs/queries)에서 이용 가능한 모든 메소드를 사용할 수 있습니다.

**기본키를 통해 단일 모델을 조회 하거나 예외 발생**

때때로, 모델을 찾을 수 없을 경우 `App::error` 핸들러를 사용하여 예외를 포착하고 404 페이지를 표시 할 수 있도록 에외를 발생 시키길 원할 수도 있습니다.

	$model = User::findOrFail(1);

	$model = User::where('votes', '>', 100)->firstOrFail();

에러 핸들러를 등록하려면, `ModelNotFoundException`을 주시하면 됩니다.

	use Illuminate\Database\Eloquent\ModelNotFoundException;

	App::error(function(ModelNotFoundException $e)
	{
		return Response::make('Not Found', 404);
	});

**엘로퀀트 모델을 사용하여 질의**

	$users = User::where('votes', '>', 100)->take(10)->get();

	foreach ($users as $user)
	{
		var_dump($user->name);
	}

쿼리 빌더의 집계 함수 또한 사용 가능 합니다.

**엘로퀀트 집계**

	$count = User::where('votes', '>', 100)->count();

유연한 인터페이스를 통해 원하는 쿼리를 생성할수 없을 경우, 'whereRaw' 메소드를 사용하면 됩니다.:

	$users = User::whereRaw('age > ? and votes = 100', array(25))->get();

**쿼리 커넥션 지정**

엘로퀀트 쿼리를 실행 할때, 어떤 데이터베이스 커넥션이 사용될지 지정 할 수도 있습니다. 간단히 `on` 메소드를 사용하면 됩니다.:

	$user = User::on('connection-name')->find(1);

<a name="mass-assignment"></a>
## 대량 할당(Mass Assignment)

새로운 모델을 생성할때, 모델의 생성자에 속성배열을 전달합니다. 그러면 이 속성은 대량 할당을 통해 모델에 할당됩니다. 이 방법은 편리하지만, 무턱대고 사용자 입력을 모델로 전달할 경우, **심각한** 보안 문제가 될 수 있습니다. 만약 사용자 입력이 모델로 무턱대고 전달 된다면, 사용자는 모델의 **어떠한** 속성이나 **모든** 속성을 수정 할 수 있습니다. 이러한 이유로, 모든 엘로쿼트 모델은 기본적으로 대량 할당으로부터 보호되어 있습니다.

시작하려면, 모델에 `fillable` 또는 `guarded` 속성을 설정합니다.

`fillable` 속성(property)은 어떤 속성(attributes)들이 대량 할당을 통해 할당 가능한지 명시합니다. 이 속성(property)은 클래스나 인스턴스 레벨에서 설정할 수 있습니다.

**모델에 할당 가능한 속성 정의**

	class User extends Eloquent {

		protected $fillable = array('first_name', 'last_name', 'email');

	}

이 예제를 보면, 나열된 3개의 속성만 대량 할당을 통해 할당 될 수 있습니다. 

`fillabel`의 반대는 `guarded` 입니다. "화이트리스트" 대신 "블랙리스트"처럼 사용됩니다.:

**모델에 할당 불가능한 속성 정의**

	class User extends Eloquent {

		protected $guarded = array('id', 'password');

	}

위 예제에서, `id`와 `password` 속성은 대량 할당을 통해 할당 될 수 **없습니다**. 나머지 다른 모든 속성은 대량 할당 될수 있습니다. 또한 아래의 메소드를 사용하여 **모든** 속성을 대량 할당으로 부터 막을 수 있습니다.:

** 모든 속성을 대량 할당으로부터 차단**

	protected $guarded = array('*');

<a name="insert-update-delete"></a>
## 삽입(Insert), 수정(Update), 삭제(Delete)

모델에서 새로운 레코드를 데이터베이스에 생성하려면, 간단히 새로운 모델 인스턴스를 만들고 `save` 메소드를 호출하면 됩니다.

**새로운 모델 저장**

	$user = new User;

	$user->name = 'John';

	$user->save();

> **노트:** 일반적으로, 당신의 엘로쿼트 모델은 자동 증가 키를 갖고 있을 겁니다. 그러나, 자신만의 키를 지정하길 원한다면, 모델의 `incrementing` 속성을 `false`로 설정합니다.

또한 새로운 모델을 저장할때 `create` 메소드를 사용하여 한줄에서 저장할 수도 있습니다. 메소드에서는 삽입된 모델의 인스턴스가 반환됩니다. 그러나, 이 메소드를 사용하기 전에, 모든 엘로퀀트트 모델이 대량 할당으로 부터 보호되어 있으므로, 모델에 `fillable` 또는 `guarded` 속성을 지정해야 합니다. 

**모델에 할당 불가능한 속성 정의**

	class User extends Eloquent {

		protected $guarded = array('id', 'account_id');

	}

**모델의 create 메소드 사용**

	$user = User::create(array('name' => 'John'));

모델을 수정하려면, 모델을 조회하고 속성을 변경한 다음, `save` 메소드를 사용하면 됩니다.:

**조회된 모델 수정**

	$user = User::find(1);

	$user->email = 'john@foo.com';

	$user->save();

때때로 하나의 모델 뿐만 아니라, 해당 모델에 관계된 모든 모델까지 저장되길 원할 수도 있습니다. 이럴땐, `push` 메소드를 사용합니다.:

**모델과 관계된 모델까지 저장**

	$user->push();

또한 쿼리처럼 다수 모델의 수정을 실행할 수도 있습니다.:

	$affectedRows = User::where('votes', '>', 100)->update(array('status' => 2));

모델을 삭제하려면, 인스턴스에서 간단하게 `delete` 메소드를 호출하면 됩니다.:

**기존의 모델 삭제**

	$user = User::find(1);

	$user->delete();

**키를 통한 기존의 모델 삭제**

	User::destroy(1);

	User::destroy(array(1, 2, 3));

	User::destroy(1, 2, 3);

물론, 다수의 모델에 삭제 쿼리를 실행할 수도 있습니다.:

	$affectedRows = User::where('votes', '>', 100)->delete();

간단하게 모델의 타임스탬프만 수정하려면, `touch` 메소드를 사용하면 됩니다.:

**모델의 타임스탬프만 수정**

	$user->touch();

<a name="soft-deleting"></a>
## 소프트 삭제

모델을 소프트 삭제하면, 데이터베이스에서 실제로 삭제 되진 않습니다. 대신, 레코드에 `deleted_at` 타임스탬프가 설정됩니다. 모델에 소프트 삭제를 활성시키려면, 모델의 `softDelete` 속성을 지정하면 됩니다.:

	class User extends Eloquent {

		protected $softDelete = true;

	}

마이그레이션에서 `softDeletes` 메소드를 사용하여 테이블에 `deleted_at` 컬럼을 추가 할 수 있습니다.

	$table->softDeletes();

이제, 모델의 `delete` 메소드가 호출되면, `deleted_at` 컬럼이 현재의 타임스탬프로 업데이트 됩니다. 소프트 삭제를 사용하는 모델을 질의 할 경우, "삭제된" 모델은 질의 결과에 포함되지 않습니다. `withTrashed` 메소드를 사용하여 소프트 삭제된 모델들을 결과에 나타나게 할 수 있습니다.:

**소프트 삭제된 모델들을 결과에 포함**

	$users = User::withTrashed()->where('account_id', 1)->get();

**오직** 소프트 삭제된 모델들만 결과로 받으려면 `onlyTrashed` 메소드를 사용합니다.:

	$users = User::onlyTrashed()->where('account_id', 1)->get();

소프트 삭제된 모델을 활성화 상태로 복구 하려면, `restore` 메소드를 사용합니다.:

	$user->restore();

질의에 `restore` 메소드를 사용 할 수도 있습니다.:

	User::withTrashed()->where('account_id', 1)->restore();

관계성에서도 `restore` 메소드를 사용 할 수 있습니다.:

	$user->posts()->restore();

만약 모델이 진짜로 삭제되길 원한다면, `forceDelete` 메소드를 사용하여 데이터베이스에서 삭제 할 수 있습니다.:

	$user->forceDelete();

`forceDelete` 메소드는 관계성에서도 작동합니다.:

	$user->posts()->forceDelete();

주어진 모델 인스턴스가 소프트 삭제가 되었는지 확인 하려면, `trashed` 메소드를 사용합니다.:

	if ($user->trashed())
	{
		//
	}

<a name="timestamps"></a>
## 타임스탬프

기본적으로 엘로퀀트는 데이터베이스 테이블의 `created_at`과 `updated_at`컬럼을 자동으로 관리 합니다. 간단히 테이블에 이러한 `timestamp` 컬럼을 추가하면, 나머지는 엘로퀀트가 알아서 처리합니다. 만약 엘로퀀트가 이러한 컬럼을 관리하지 않도록 하려면, 모델에 다음의 속성을 추가합니다.:

**자동 타임스탬프를 사용하지 않도록 설정**

	class User extends Eloquent {

		protected $table = 'users';

		public $timestamps = false;

	}

타임스탬프의 형식을 지정하길 원한다면, 모델의 `getDateFormat` 메소드를 치환하면 됩니다.:

**사용자 정의 타임스탬프 형식 제공**

	class User extends Eloquent {

		protected function getDateFormat()
		{
			return 'U';
		}

	}

<a name="query-scopes"></a>
## 쿼리 스코프

스코프는 모델에서 쿼리 로직을 쉽게 재사용 할수 있도록 해줍니다. 스코프를 정의 하려면, 모델의 메소드명 앞에 `scope`를 붙이면 됩니다.:

**쿼리 스코프 정의**

	class User extends Eloquent {

		public function scopePopular($query)
		{
			return $query->where('votes', '>', 100);
		}

	}

**쿼리 스코프 활용**

	$users = User::popular()->orderBy('created_at')->get();

**동적 스코프**

때때로 매개 변수를 받는 스코프를 정의 해야 할때가 있습니다. 그렇게 하려면 간단히 스코프 메소드에 매개 변수를 추가 하면 됩니다.:

	class User extends Eloquent {

		public function scopeOfType($query, $type)
		{
			return $query->whereType($type);
		}

	}

다음, 스코프 호출에 매개 변수를 전달 합니다.:

	$users = User::ofType('member')->get();

<a name="relationships"></a>
## 관계성

몰론, 당신의 데이터베이스 테이블은 아마도 서로 연관되어 있을 겁니다. 예를 들어, 하나의 블로그 포스트가 많은 코멘트를 갖고 있거나, 하나의 주문이 주문을 한 사용자와 연관되어 있을 수 있습니다. 엘로퀀트는 이러한 관계성을 쉽게 관리하고 작동할수 있게 해줍니다. Laravel은 4가지 종류의 관계성을 지원합니다.:

- [One To One](#one-to-one)
- [One To Many](#one-to-many)
- [Many To Many](#many-to-many)
- [다형성 관계](#polymorphic-relations)

<a name="one-to-one"></a>
### One To One

one-to-one 관계는 아주 기본적인 관계 입니다. 예를 들면, `User` 모델은 하나의 `Phone` 모델을 갖고 있습니다. 엘로퀀트에서 이러한 관계를 정의 할수 있습니다.:

**One To One 관계 정의**

	class User extends Eloquent {

		public function phone()
		{
			return $this->hasOne('Phone');
		}

	}

`hasOne`에 전달되는 첫번째 인수는 관계된 모델명 입니다. 관계가 정의되고 나면, 엘로퀀트의 [동적 속성](#dynamic-properties)을 사용하여 관계된 모델을 조회할 수 있습니다.:

	$phone = User::find(1)->phone;

이 구문에 수행된 SQL은 다음과 같습니다.:

	select * from users where id = 1

	select * from phones where user_id = 1

엘로퀀트는 관계된 모델명을 기반으로 외래키를 추정 한다는 것에 주목하십시오. 이 경우, `Phone` 모델은 `user_id`를 외래키로 사용하도록 추정합니다. 이러한 규칙을 치환하려면, `hasOne` 메소드의 두번째 인수에 사용자 정의 키를 전달합니다.:

	return $this->hasOne('Phone', 'custom_key');

`Phone` 모델의 역관계를 정의하려면 `belongTo` 메소드를 사용합니다.:

**역관계 정의**

	class Phone extends Eloquent {

		public function user()
		{
			return $this->belongsTo('User');
		}

	}

위의 예제에서, 엘로퀀트는 `phone` 테이블에서 `user_id` 컬럼을 검색 합니다. 만약 다른 이름의 외래키 컬럼을 정의하고 싶다면, `belongsTo`의 2번째 인수에 전달 하면 됩니다.:

	class Phone extends Eloquent {

		public function user()
		{
			return $this->belongsTo('User', 'custom_key');
		}

	}

<a name="one-to-many"></a>
### One To Many

one-to-many 관계의 예를 보면, "많은" 코멘트를 가진 "하나"의 블로그 포스가 될 수 있습니다. 다음과 같이 이 관계를 맺을 수 있습니다.:

	class Post extends Eloquent {

		public function comments()
		{
			return $this->hasMany('Comment');
		}

	}

이제 [동적 속성](#dynamic-properties)을 사용하여 포스트의 코멘트를 액세스 할수 있습니다.:

	$comments = Post::find(1)->comments;

더 많은 제약 조건을 추가하여 코멘트를 조회하려면, `comments` 메소드를 사용 한 다음, 조건을 계속 체이닝 할 수 있습니다.:

	$comments = Post::find(1)->comments()->where('title', '=', 'foo')->first();

다시 말하지만, `hasMany` 메소드의 두번째 인수를 전달함으로서 규칙된 외래키를 변경할 수 있습니다.:

	return $this->hasMany('Comment', 'custom_key');

`Comment` 모델의 역관계를 정의하려면, `belongsTo` 메소드를 사용합니다.:

**역관계 정의**

	class Comment extends Eloquent {

		public function post()
		{
			return $this->belongsTo('Post');
		}

	}

<a name="many-to-many"></a>
### Many To Many

Many-to-many 관계는 좀 더 복잡한 관계 타입입니다. 사용자(users)는 다수의 역할(roles)을 갖고 있고 역할(roles) 또한 많은 사용자(users)에 의해 공유되는 것을 이 관계의 예제로 들 수 있습니다. 예를 들면, 다수의 사용자(users)는 "Admin" 역할(roles)을 갖고 있습니다. 이 관계에는 `users`, `roles`, `role_user` 3개의 데이터베이스 테이블이 필요합니다. `role_user` 테이블은 관련 모델 이름의 알파벳 순서에서 비롯되며, `user_id`와 `role_id` 컬럼을 포함하고 있어야 합니다.

`belongsToMany` 메소드를 사용하여 many-to-many 관계를 정의할 수 있습니다.:

	class User extends Eloquent {

		public function roles()
		{
			return $this->belongsToMany('Role');
		}

	}

이제, `User` 모델을 통해 역할(roles)을 조회할 수 있습니다.:

	$roles = User::find(1)->roles;

피벗테이블에 통상적이지 않은 테이블명을 사용하려면, `belongsToMany` 메소드의 두번째 인수에 테이블명을 전달하면 됩니다.:

	return $this->belongsToMany('Role', 'user_roles');

또한 통상적인 관련된 키를 재정의 할 수도 있습니다.:

	return $this->belongsToMany('Role', 'user_roles', 'user_id', 'foo_id');

물론, `Role` 모델에도 역관계를 정의 할 수 있습니다.:

	class Role extends Eloquent {

		public function users()
		{
			return $this->belongsToMany('User');
		}

	}

<a name="polymorphic-relations"></a>
### 다형성 관계

다형성 관계는 한개의 모델이 단일 결합을 통해 한개 이상의 다른 모델에 속할수 있도록 해줍니다. staff 모델 또는 order 모델 두곳에 속해있는 photo 모델을 예로 들 수 있습니다. 이러한 관계를 아래와 같이 구현합니다.:

	class Photo extends Eloquent {

		public function imageable()
		{
			return $this->morphTo();
		}

	}

	class Staff extends Eloquent {

		public function photos()
		{
			return $this->morphMany('Photo', 'imageable');
		}

	}

	class Order extends Eloquent {

		public function photos()
		{
			return $this->morphMany('Photo', 'imageable');
		}

	}

이제 staff, order 두곳에서 photo를 조회 할 수 있습니다.:

**다형성 관계 조회**

	$staff = Staff::find(1);

	foreach ($staff->photos as $photo)
	{
		//
	}

하지만, "다형성"의 진짜 마법은 `Photo` 모델에서 staff나 order를 액세스 할때 나타납니다.:

**다형성 관계의 소유자 조회**

	$photo = Photo::find(1);

	$imageable = $photo->imageable;

`Photo` 모델의 `imageable`은 어떤 모델이 그 photo를 소유했는지에 따라 `Staff` 또는 `Order` 인스턴스를 반환합니다.

어떻게 작동하는지 이해를 돕기 위해 다형성 관계의 데이터베이스 구조를 살펴보겠습니다.:

**다형성 관계 테이블 구조**

	staff
		id - integer
		name - string

	orders
		id - integer
		price - integer

	photos
		id - integer
		path - string
		imageable_id - integer
		imageable_type - string

여기에서 중요한 필드는 `photo` 테이블의 `imageable_id`와 `imageable_type` 입니다. 이 예제에서 ID는 소유어 있는 staff 또는 order의 ID 값을 포함하며, type은 소유 모델의 클래스명을 포함하고 있습니다. 이 부분이 `imageable` 관계를 액세스 했을 경우, ORM이 어떤 타입의 소유 모델을 반환하는지 결정할 수 있도록 해줍니다.

<a name="querying-relations"></a>
## 관계 질의

모델에서 레코드를 액세스 할 때, 관계된 레코드의 존재를 기반으로 결과를 제한 하고 싶을 때가 있습니다. 최소한 한개의 코멘트를 가진 모든 블로그 포스트를 액세스 할 경우를 예로 들수 있습니다. 이럴 경우 `has` 메소드를 사용하여 할 수 있습니다.:

**선택할 때 관계 확인**

	$posts = Post::has('comments')->get();

또한, 연산자와 카운트를 명시 할 수도 있습니다.:

	$posts = Post::has('comments', '>=', 3)->get();

<a name="dynamic-properties"></a>
### 동적 속성

엘로퀀트는 동적 속성을 통해 해당 관계를 액세스 할 수 있게 해줍니다. 엘로퀀트는 자동으로 관계성을 로드해주며 `get` (one-to-many 관계성) 또는 `first` (one-to-one 관계성) 중 무엇을 호출해야 할지 알고 있을 만큼 똑똑합니다. 그런 다음, 관계된 모델과 같은 이름인 동적 속성을 통해 액세스 할 수 있습니다. 예를 들어 다음의 `$phone` 모델은:

	class Phone extends Eloquent {

		public function user()
		{
			return $this->belongsTo('User');
		}

	}

	$phone = Phone::find(1);
	
user의 email을 아래처럼 출력하는 것 대신:

	echo $phone->user()->first()->email;

아래와 같이 간단하게 줄일수 있습니다.:

	echo $phone->user->email;

<a name="eager-loading"></a>
## Eager 로딩

Eager 로딩은 N + 1 쿼리 문제를 완화하기 위해 존재합니다. `Author` 모델에 관계된 `Book` 모델을 예로 들면 다음과 같습니다.:

	class Book extends Eloquent {

		public function author()
		{
			return $this->belongsTo('Author');
		}

	}

이제, 다음의 코드를 생각해 보십시오.:

	foreach (Book::all() as $book)
	{
		echo $book->author->name;
	}

이 반복문은 한개의 쿼리를 실행하여 테이블의 모든 book을 조회하고 각 책의 author을 조회하기 위해 또 다른 쿼리를 실행합니다. 그러므로 25개의 book이 있다면 이 반복문은 26개의 쿼리를 실행할 겁니다.

고맙게도, eager 로딩을 사용하여 쿼리 갯수를 대폭 줄일 수 있습니다. `with` 메소드를 통해 관계된 것들을 eager 로딩 할 수 있습니다.:

	foreach (Book::with('author')->get() as $book)
	{
		echo $book->author->name;
	}

위 반복문에서는 단 2개의 쿼리만 실행 됩니다.:

	select * from books

	select * from authors where id in (1, 2, 3, 4, 5, ...)

eager 로딩의 현명한 사용은 어플리케이션의 퍼포먼스를 대폭 향상 시킬 수 있습니다.

또한 다수의 관계를 한번에 eager 로딩 할 수 있습니다.:

	$books = Book::with('author', 'publisher')->get();

중첩된 관계도 eager 로딩 할 수 있습니다.:

	$books = Book::with('author.contacts')->get();

위의 예제에서 `author` 관계가 eager 로드 되며 author의 `contacts` 관계 또한 로드 됩니다.

### Eager 로딩 제약

때때로 조건을 지정하여 eager 로딩을 할 수도 있습니다. 다음의 예제를 보십시오.:

	$users = User::with(array('posts' => function($query)
	{
		$query->where('title', 'like', '%first%');
	}))->get();

이 예제는 제목에 "first" 단어를 포함하고 있는 user의 post만 eager 로딩 합니다.

### Lazy Eager 로딩

이미 존재 하는 모델 컬렉션에서 관련된 모델을 eager 로딩하는것 또한 가능 합니다. 캐시와 함께 조합하거나 관계된 모델을 로드 할지 안할지 동적으로 결정하는데 유용합니다.

	$books = Book::all();

	$books->load('author', 'publisher');

<a name="inserting-related-models"></a>
## 관계된 모델 삽입

종종 새로운 관계된 모델을 삽입해야 할 때가 있습니다. 포스트에 새로운 코멘트를 남기는 경우를 예로 들 수 있습니다. 코멘트 모델의 `post_id`에 외래 키를 직접 입력 하는 것 대신, 코멘트의 부모 `Post` 모델에 직접 새로운 코멘트를 삽입할 수 있습니다.:

**관계된 모델 부여**

	$comment = new Comment(array('message' => 'A new comment.'));

	$post = Post::find(1);

	$comment = $post->comments()->save($comment);

이 예제에서 삽입된 코멘트 `post_id` 필드는 자동으로 설정 됩니다.

### 연관 모델 (Belongs To)

`belongsTo` 관계를 업데이트 할때, `associate` 메소드를 사용할 수 있습니다. 이 메소드는 자식 모델의 외래키를 설정해줍니다.:

	$account = Account::find(10);

	$user->account()->associate($account);

	$user->save();

### 관계된 모델 삽입 (Many To Many)

many-to-many 관계에서도 관계된 모델을 삽입 할 수 있습니다. `User`과 `Role` 모델을 계속 예제로 보겠습니다. `attach` 메소드를 사용하여 쉽게 새로운 role을 user에게 부여 할 수 있습니다.:

**Many To Many 모델 부여**

	$user = User::find(1);

	$user->roles()->attach(1);

또한, 피벗 테이블에 저장될 속성을 배열로 전달 할 수도 있습니다.:

	$user->roles()->attach(1, array('expires' => $expires));

물론, `attach`의 반대는 `detach` 입니다.:

	$user->roles()->detach(1);

`sync` 메소드를 사용하여 관계된 모델을 부여 할 수도 있습니다. `sync` 메소드는 피벗 테이블 삽입 할 ID의 배열을 받습니다. 이 오퍼레이션이 완료되면 배열에 있는 ID만 해당 모델의 피벗 테이블에 놓여집니다.:

**Sync를 사용하여 Many To Many 모델 부여**

	$user->roles()->sync(array(1, 2, 3));

또한 주어진 ID들의 다른 피벗 테이블 값들을 설정 할 수 있습니다.

**Sync할때 피벗 데이터 추가**

	$user->roles()->sync(array(1 => array('expires' => true)));

새로운 관계된 모델을 생성하고 단일 커맨드를 통해 부여하려면 `save` 메소드를 사용하면 됩니다.:

	$role = new Role(array('name' => 'Editor'));

	User::find(1)->roles()->save($role);

이 경우, 새로운 `Role` 모델이 저장되고 user 모델에 부여 됩니다. 물론 배열 속성을 전달하여 피벗 테이블에 저장 할 수도 있습니다.:

	User::find(1)->roles()->save($role, array('expires' => $expires));

<a name="touching-parent-timestamps"></a>
## 부모 타임스탬프 터치

`Post`에 속해 있는 `Comment` 같이, 모델이 다른 모델에 '속해 있을(belongsTo)' 경우, 종종 자식의 모델이 업데이트 됐을 때 부모의 타임스탬프를 업데이트 하는 것이 도움이 될때가 있습니다. 예를 들어 `Comment` 모델이 업데이트 됐을 경우, 부모인 `Post`의 `updated_at` 타임스탬프를 자동으로 터치하길 원할 수도 있습니다. 엘로퀀트는 이것을 쉽게 해줍니다. 그냥 자식 모델에 관계된 모델 명을 `touches` 속성에 추가해 주면 됩니다.:

	class Comment extends Eloquent {

		protected $touches = array('post');

		public function post()
		{
			return $this->belongsTo('Post');
		}

	}

이제, `Comment`를 업데이트 할때마다, 부모 `Post`는 업데이트 된 `updated_at` 컬럼을 가지게 됩니다.:

	$comment = Comment::find(1);

	$comment->text = 'Edit to this comment!';

	$comment->save();

<a name="working-with-pivot-tables"></a>
## 피벗 테이블과 함께 작업

이미 배운바와 같이 many-to-many 관계는 중간 테이블을 필요로 합니다. 엘로퀀트는 이 테이블과 상호작용 하는 매우 도움이 되는 방법들을 제공합니다. 예를 들어 `User` 객체가 해당 객체에 관계된 많은 `Role` 객체를 갖고있다고 합시다. 관계를 액세스 한다음 모델에서 `pivot` 테이블을 액세스 할수 있습니다.

	$user = User::find(1);

	foreach ($user->roles as $role)
	{
		echo $role->pivot->created_at;
	}

조회된 각각의 `Role` 모델은 자동으로 `pivot` 속성이 부여되어 있습니다. 이 속성은 중간 테이블 모델을 나타내며 다른 엘로퀀트 모델처럼 사용될 수 있습니다.

기본적으로, 외래키만 `pivot` 객체에 나타납니다. 만약 피벗 테이블이 다른 속성들을 포함하고 있다면 관계를 정의할때 반드시 그 속성들을 명시해야 합니다.:

	return $this->belongsToMany('Role')->withPivot('foo', 'bar');

이제 `foo`와 `bar` 속성이 `Role` 모델의 `pivot` 객체에서 액세스 가능합니다.

피벗 테이블의 `created_at`과 `updated_at` 타임스탬프가 자동으로 관리되길 원한다면, 관계성 정의에 `withTimestamps` 메소드를 사용하십시오.:

	return $this->belongsToMany('Role')->withTimestamps();

모델의 피벗 테이블에 있는 모든 레코드를 삭제하려면 `detach` 메소드를 사용하십시오.:

**피벗 테이블의 레코드 삭제**

	User::find(1)->roles()->detach();

이 작업은 `roles` 테이블의 레코드를 삭제하는게 아니라, 오직 피벗 테이블의 레코드만 삭제한다는것을 명심하십시오.

<a name="collections"></a>
## 컬렉션

엘로퀀트나, `get` 메소드나 `relationship`을 통해 반환되는 모든 다수의 결과 세트는 컬렉션 오브젝트를 반환합니다. 이 객체는 PHP의 `IteratorAggregate` 인터페이스를 구현하므로 배열처럼 반복될 수 있습니다. 또한, 이 객체는 결과 세트와 작업할 수 있는 도움이 되는 다양한 메소드를 갖고있습니다.

예를 들어, `containts` 메소드를 사용하여 결과 세트가 주어진 기본키를 포함하고 있는지 알아낼 수 있습니다.:

**컬렉션이 기본키를 포함하고 있는지 확인**

	$roles = User::find(1)->roles;

	if ($roles->contains(2))
	{
		//
	}

컬렉션은 또한 배열이나 JSON으로 변환될수 있습니다.:

	$roles = User::find(1)->roles->toArray();

	$roles = User::find(1)->roles->toJson();

만약 컬렉션이 문자열로 묘사될 경우, JSON을 반환합니다

	$roles = (string) User::find(1)->roles;

엘로퀀트 컬렉션은 또한, 반복문이나 필터링을 통해 아이템이 포함되어 있는지 확인하는 메소드를 포함하고 있습니다.:

**컬레션 반복 & 필터링**

	$roles = $user->roles->each(function($role)
	{

	});

	$roles = $user->roles->filter(function($role)
	{

	});

**컬렉션에서 객체에 콜백 적용**

	$roles = User::find(1)->roles;
	
	$roles->each(function($role)
	{
		//	
	});

**값으로 컬렉션을 정렬**

	$roles = $roles->sortBy(function($role)
	{
		return $role->created_at;
	});
	
때때로, 사용자 추가 메소드와 함께 사용자 정의 컬렉션 객체를 반환할 수도 있습니다. 엘로퀀트 모델의 `newCollection` 메소드를 치환하여 새로운 컬렉션을 반환합니다.:

**사용자 정의 컬렉션 타입 반환**

	class User extends Eloquent {

		public function newCollection(array $models = array())
		{
			return new CustomCollection($models);
		}

	}

<a name="accessors-and-mutators"></a>
## 접근자 & 변경자

엘로퀀트는 모델 속성을 저장하거나 불러올때 모델 속성을 변형할수 있도록 해주는 편리한 방법을 제공합니다. 간단히 모델에 `getFooAttribute` 메소드를 정의하여 접근자를 선언할 수 있습니다. 데이터베이스의 컬럼이 스네이크 케이스 일지라도 접근자 메소드는 캐멀케이스를 형태를 따라야한다는 것을 명심하십시오.:

**접근자 정의**

	class User extends Eloquent {

		public function getFirstNameAttribute($value)
		{
			return ucfirst($value);
		}

	}

위의 예제에서 `first_name` 컬럼은 접근자를 갖고 있습니다. 이 속성의 값이 접근자로 전달 된다는 것에 주목하십시오.

변경자 역시 비슷한 방법으로 선언됩니다.

**변경자 정의**

	class User extends Eloquent {

		public function setFirstNameAttribute($value)
		{
			$this->attributes['first_name'] = strtolower($value);
		}

	}

<a name="date-mutators"></a>
## 날짜 변경자

기본적으로, 엘로퀀트는 `created_at`, `updated_at`, `deleted_at` 컬럼을 각종 도움이 되는 메소드를 제공하고, 네이티브 PHP의 `DateTime` 클래스를 확장한 [Carbon](https://github.com/briannesbitt/Carbon)의 인스턴스로 변환합니다.

모델의 `getDates` 메소드를 오버라이딩하여, 어떤 필드들이 자동으로 변경될지, 또한 지금의 변경자를 사용하지 않을 지까지 마음대로 정할 수 있습니다.:

	public function getDates()
	{
		return array('created_at');
	}

컬럼이 날짜(date)로 간주 될때는, 그 값을 UNIX 타임스탬프나, 날짜(date) 문자열 (`Y-m-d`), 날짜-시간(date-time) 문자열, 또는 `DateTime` / `Carbon` 인스턴스 중 하나로 설정 해야 합니다.

날짜 변경자를 완전히 사용하지 않으려면, 간단하게 `getDates` 메소드에서 빈 배열을 반환하면 됩니다.:

	public function getDates()
	{
		return array();
	}

<a name="model-events"></a>
## 모델 이벤트

엘로퀀트 모델은 다음의 메소드들을 사용하여 라이프사이클의 다양한 포인트에서 이벤트를 연결할 수 있도록 해줍니다.: `creating`, `created`, `updating`, `updated`, `saving`, `saved`, `deleting`, `deleted`.

새로운 아이템이 처음으로 저장될 때마다, `creating`과 `created` 이벤트가 발생 합니다. 만약 새로운 아이템이 아닐경우 `save` 메소드가 호출 되었다면, `updating` / `updated` 이벤트가 발생 합니다. 또 두 경우 모두, `saving` / `saved` 이벤트가 발생 합니다

만약 `creating`, `updating`, `saving` 또는 `deleteing` 이벤트로부터 `false`가 반환된다면 그 액션은 취소됩니다.:

**이벤트를 통한 저장 오퍼레이션 중지**

	User::creating(function($user)
	{
		if ( ! $user->isValid()) return false;
	});

엘로퀀트는 또한 당신의 이벤트 바인딩을 동록할수 있는 편리한 위치인 static `boot` 메소드를 포함하고 있습니다.

**모델 부트 메소드 설정**

	class User extends Eloquent {

		public static function boot()
		{
			parent::boot();

			// Setup event bindings...
		}

	}

<a name="model-observers"></a>
## 모델 옵저버

모델 옵저버를 등록하여, 모델 이벤트 처리를 강화 할 수 있습니다. 옵저버 클래스에는 모델 이벤트에 해당하는 다양한 메소드들이 포함될 수 있습니다. 예를 들면, `creating`, `updating`, `saving` 메소드들과 기타 모델 이벤트명이 옵저버가 될 수 있습니다.

그 예로, 모델 옵저버는 다음과 같이 생길 수 있습니다.:

	class UserObserver {

		public function saving($model)
		{
			//
		}

		public function saved($model)
		{
			//
		}

	}

`observe` 메소드를 사용하여, 옵저버 인스턴스를 등록 할 수 있습니다.:

	User::observe(new UserObserver);

<a name="converting-to-arrays-or-json"></a>
## 배열 / JSON 으로 변환

JSON API를 구축할때, 종종 모델과 관계들을 배열이나 JSON으로 변환해야 합니다. 그래서 엘로퀀트는 변환해주는 메소드를 포함하고 있습니다.`toArray` 메소드를 사용하여 모델과 로드된 관계를 배열로 변환할 수 있습니다.:

**모델을 배열로 변환**

	$user = User::with('roles')->first();

	return $user->toArray();

모델의 전체 컬렉션 또한 배열로 변환될 수 있습니다.:

	return User::all()->toArray();

모델을 JSON으로 변환하려면, `toJson` 메소드를 사용합니다.:

**모델을 JSON으로 변환**

	return User::find(1)->toJson();

모델이나 컬렉션이 문자열으로 묘사된다면 JSON으로 변환됩니다. 이 뜻은, 어플리케이션의 라우트에서 엘로퀀트 객체를 바로 반환 할 수 있습니다!

**라우트에서 모델 반환**

	Route::get('users', function()
	{
		return User::all();
	});

가끔 비밀번호 같이 모델에 포함된 속성들을 제한해야 할때가 있습니다. 이럴땐, 모델에 `hidden` 속성을 추가하면 됩니다.:

**배열이나 JSON 변환으로부터 속성을 숨김**

	class User extends Eloquent {

		protected $hidden = array('password');

	}

반대로, `visible` 속성을 사용하여 화이트리스트를 정의 할 수 있습니다.:

	protected $visible = array('first_name', 'last_name');

<a name="array-appends"></a>
가끔 데이터베이스에 해당되지 않는 배열 속성이 필요할 때도 있습니다. 그러려면, 그 값을 위한 접근자를 지정하면 됩니다.:

	public function getIsAdminAttribute()
	{
		return $this->attributes['admin'] == 'yes';
	}

접근자를 만들고 나면, 모델의 `appends` 속성에 값을 추가 하면 됩니다.:

	protected $appends = array('is_admin');

그 속성이 `appends` 리스트에 추가되고 나면, 모델의 배열과 JSON 형식 모두에 포함될겁니다.