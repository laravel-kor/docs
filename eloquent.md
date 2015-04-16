# 엘로퀀트 ORM

- [소개](#introduction)
- [기본 사용법](#basic-usage)
- [대량 할당](#mass-assignment)
- [삽입, 업데이트, 삭제](#insert-update-delete)
- [일시 삭제](#soft-deleting)
- [타임스탬프](#timestamps)
- [쿼리 스코프](#query-scopes)
- [전역 스코프](#global-scopes)
- [관계](#relationships)
- [관계 쿼리](#querying-relations)
- [즉시 로드](#eager-loading)
- [관련 모델 삽입](#inserting-related-models)
- [부모 타임스탬프 터치](#touching-parent-timestamps)
- [피벗 테이블과 작업](#working-with-pivot-tables)
- [컬렉션](#collections)
- [접근자 & 설정자](#accessors-and-mutators)
- [날짜 설정자](#date-mutators)
- [속성 캐스팅](#attribute-casting)
- [모델 이벤트](#model-events)
- [모델 관찰자](#model-observers)
- [모델 URL 생성](#model-url-generation)
- [배열 / JSON으로 변환](#converting-to-arrays-or-json)

<a name="introduction"></a>
## 소개

라라벨에 포함되어있는 엘로퀀트 ORM은 여러분의 데이터베이스와 작동하는 아름답고 간단한 액티브레코드 구현 클래스를 제공합니다. 각각의 데이터베이스 테이블은 서로 상호 작용하는데 사용되는 "Model"을 갖고 있습니다.

시작하기전에 `config/database.php`에 데이터베이스 연결정보가 구성되어 있는지 확인하세요.

<a name="basic-usage"></a>
## 기본 사용법

시작하려면, 엘로퀀트 모델을 생성합니다. 모델은 일반적으로 `app` 디렉토리에 존재합니다, 하지만 `composer.json` 파일에 따르는 오토-로드가 되는 곳 어디든지 위치해도 좋습니다. 모든 엘로퀀트 모델은 `Illuminate\Database\Eloquent\Model`을 확장합니다.

#### 엘로퀀트 모델 정의

    class User extends Model {}

여러분은 또한 `make:model` 커맨드를 사용하여 엘로퀀트 모델을 생성 할 수도 있습니다:

    php artisan make:model User

`User` 모델이 어떤 테이블을 사용해야 하는지 엘로퀀트에게 말해주지 않았다는 점을 명심하세요. 다른 이름이 명시되지 않는 이상, 클래스의 복수 형태의 스네이크 표기법이 테이블명으로 사용됩니다. 그러므로, 이 경우, 엘로퀀트는 `User` 모델이 `users` 테이블에 레코드를 저장한다고 가정합니다. 여러분은 모델에 `table` 속성을 지정하여 사용자 정의 테이블을 명시 할 수도 있습니다:

    class User extends Model {

        protected $table = 'my_users';

    }

> **주의:** 엘로퀀트는 또한 각각의 테이블이 `id`라는 기본 키를 가지고 있다고 가정합니다. `primaryKey` 속성을 정의하여 이 규칙을 재정의 할 수 있습니다.  property to override this convention. 마찬가지로 `connection` 속성을 정의하여 모델이 사용할 데이터베이스 연결명을 재정의할 수도 있습니다.

모델이 정의되고 나면, 여러분은 테이블에서 레코드를 조회하고 생성할 준비가 되었습니다. 테이블에 `updated_at`와 `created_at` 컬럼을 기본적으로 포함하고 있어야 한다는 것을 명심하세요. 만약 이러한 컬럼들이 자동으로 관리되길 바라지 않는다면, 모델의 `$timestamps` 속성을 `false`로 설정하세요.

#### 모든 레코드 조회

    $users = User::all();

#### 기본 키를 바탕으로 레코드 조회

    $user = User::find(1);

    var_dump($user->name);

> **주의:** [쿼리 빌더](/docs/queries)에서 사용 가능한 모든 메서드는 엘로퀀트 모델에서도 역시 사용 가능합니다.

#### 기본 키를 바탕으로 레코드를 조회하거나 예외 발생

때로는 모델이 발견되지 않을 때 예외를 발생 시키고 싶을 때도 있습니다. 이렇게 하려면, `firstOrFail` 메서드를 사용합니다:

    $model = User::findOrFail(1);

    $model = User::where('votes', '>', 100)->firstOrFail();

이는 여러분이 예외를 받을수 있도록 해주므로 필요에 따라 로그를 남기거나 오류 페이지를 표시 할 수도 있습니다. `ModelNotFoundException` 예외를 받으려면 여러분의 `app/Exceptions/Handler.php` 파일에 관련 로직을 추가하세요.

    use Illuminate\Database\Eloquent\ModelNotFoundException;

    class Handler extends ExceptionHandler {

        public function render($request, Exception $e)
        {
            if ($e instanceof ModelNotFoundException)
            {
                // Custom logic for model not found...
            }

            return parent::render($request, $e);
        }

    }

#### 엘로퀀트 모델을 사용하여 쿼리

    $users = User::where('votes', '>', 100)->take(10)->get();

    foreach ($users as $user)
    {
        var_dump($user->name);
    }

#### 엘로퀀트 집계

물론, 여러분은 쿼리 빌더의 집계 함수 또한 사용 할 수 있습니다.

    $count = User::where('votes', '>', 100)->count();

만약 우아한 인터페이스를 사용하여 여러분이 필요한 쿼리를 생성할 수 없다면, `whereRaw`를 사용하세요:

    $users = User::whereRaw('age > ? and votes = 100', [25])->get();

#### 결과 청크

만약 많은 수(몇천개)의 엘로퀀트 레코드를 처리해야 한다면, `chunk` 커맨드를 사용하여 램 사용량을 줄일 수 있습니다:

    User::chunk(200, function($users)
    {
        foreach ($users as $user)
        {
            //
        }
    });

메서드에 전달한 첫번째 인수는 한 "청크"마다 받을 레코드의 갯수입니다. 두번째로 전달된 클로저는 각 청크마다 데이터베이스로부터 받은 뒤 호출됩니다.

#### 쿼리 연결 명시

또한 엘로퀀트 쿼리를 실행할때 어떤 데이터베이스 연결을 사용할지 명시 할 수도 있습니다. 간단히 `on` 메서드를 사용하세요:

    $user = User::on('connection-name')->find(1);

만약 [읽기 / 쓰기 연결](/docs/5.0/database#read-write-connections)을 사용하고 있다면, 다음의 메서드와 함께 쿼리가 강제로 "쓰기" 연결을 사용하도록 할 수도 있습니다:

    $user = User::onWriteConnection()->find(1);

<a name="mass-assignment"></a>
## 대량 할당

새로운 모델을 생성할때, 속성의 배열을 모델의 생성자에게 전달합니다. 이 속성들은 그러면 대량-할당을 통해 모델에 배정됩니다. 이는 편리하지만, 사용자의 입력을 모델에 마구잡이로 전달 할 경우 **심각한** 보안이 염려될 수 있습니다. 만약 사용자 입력이 마구잡이로 모델에 전달된다면, 사용자는 모델의 **모든** 속성을 자유롭게 수정 할 수 있습니다. 이러한 이유로, 모든 엘로퀀트 모델은 기본적으로 대량-할당에 대해 보호합니다.

시작하려면, 여러분의 모델에 `fillable` 또는 `guarded` 속성을 설정하세요.

#### 모델에 채울 수 있는 속성들을 정의

`fillable` 속성(property)은 어떤 속성들(attributes)이 대량-할당 될 수 있는지 명시 합니다. 이는 클래스 또는 인스턴스 단계에서 설정 할 수 있습니다.

    class User extends Model {

        protected $fillable = ['first_name', 'last_name', 'email'];

    }

이 예제에서는, 위에 나열된 3개의 속성들만 대량-할당 될 수 있습니다.

#### 모델에 보호 되어야 할 속성들을 정의

`fillable`의 반대는 `guarded`이며, "화이트-리스트" 대신 "블랙-리스트"처럼 이용됩니다:

    class User extends Model {

        protected $guarded = ['id', 'password'];

    }

> **주의:** `guarded`를 사용 할 때, 보호되지 않은 컬럼이 업데이트 될 수 있으므로 여러분은 절대로 `save` 또는 `update` 메서드에 사용자가 통제하는 `Input::get()` 또는 원시 배열을 전달 하면 안됩니다.

#### 대량 할당으로부터 모든 속성을 차단

위의 예제에서, `id`와 `password` 속성은 대량 할당 될 수 **없습니다**. 다른 모든 속성들은 대량 할당 될 수 있습니다. 여러분은 또한 보호 속성을 사용하여 대량 할당으로부터 **모든** 속성들을 차단 할 수도 있습니다:

    protected $guarded = ['*'];

<a name="insert-update-delete"></a>
## 삽입, 업데이트, 삭제

모델에서 데이터베이스에 새로운 레코드를 생성하려면, 간단하게 새로운 모델 인스턴스를 생성하고 `save` 메서드를 호출 하세요.

#### 새로운 모델 저장

    $user = new User;

    $user->name = 'John';

    $user->save();

> **주의:** 일반적으로, 여러분의 엘로퀀트 모델은 자동-증가 키를 갖고 있습니다. 그렇지만, 여러분이 직접 키를 명시하고 싶다면, 모델의 `incrementing` 속성을 `false`로 설정하세요.

또한 `create` 메서드를 사용하여 한줄에서 바로 새로운 모델을 저장 할 수도 있습니다. 입력된 모델 인스턴스가 메서드로부터 반환됩니다. 하지만, 이렇게 하기 전에, 여러분은 모든 엘로퀀트 모델들이 대량-할당으로부터 보호되기 위해 모델에 `fillable` 또는 `guarded` 속성 중 하나를 명시해야 합니다.

자동-증가 ID를 사용하는 새로운 모델을 저장하거나 생성한 뒤에, 여러분은 객체의 `id` 속성을 액세스하여 해당 ID를 조회 할 수 있습니다:

    $insertedId = $user->id;

#### 모델에 보호 되어야 할 속성들을 설정

    class User extends Model {

        protected $guarded = ['id', 'account_id'];

    }

#### 모델 생성 메서드 사용

    // 데이터베이스에 새로운 사용자를 생성
    $user = User::create(['name' => 'John']);

    // 속성을 기반으로 해당 유저를 조회하거나, 없을 경우 새로 생성 (데이터베이스에 저장함)
    $user = User::firstOrCreate(['name' => 'John']);

    // 속성을 기반으로 해당 유저를 조회하거나, 새로운 인스턴스를 생성 (데이터베이스에 저장하지 않음)
    $user = User::firstOrNew(['name' => 'John']);

#### 조회한 모델을 업데이트

모델을 업데이트 하려면, 조회 한 다음, 속성을 변경하고, 그다음 `save` 메서드를 사용합니다:

    $user = User::find(1);

    $user->email = 'john@foo.com';

    $user->save();

#### 모델과 관계 저장

떄로는 해당 모델 뿐만 아니라 관계된 모든 모델을 저장해야 할 때도 있습니다. 이렇게 하려면, `push` 메서드를 사용하세요:

    $user->push();

또한 모델 세트에 대해서 쿼리로 업데이트를 실행 할 수도 있습니다:

    $affectedRows = User::where('votes', '>', 100)->update(['status' => 2]);

> **주의:** 엘로퀀트 쿼리 빌더를 통하여 모델 세트를 업데이트 할 때는 모델 이벤트가 발생하지 않습니다.

#### 기존 모델을 삭제

모델을 삭제하려면, 간단하게 모델 인스턴스에서 `delete` 메서드를 호출 하세요:

    $user = User::find(1);

    $user->delete();

#### 키를 기반으로 기존 모델을 삭제

    User::destroy(1);

    User::destroy([1, 2, 3]);

    User::destroy(1, 2, 3);

물론, 여러분은 모델 세트에 대해서 삭제 쿼리를 실행 할 수도 있습니다:

    $affectedRows = User::where('votes', '>', 100)->delete();

#### 모델의 타임스템프만 업데이트

만약 모델의 타임스템프만 간단하게 업데이트 하길 원한다면, `touch` 메서드를 사용하세요:

    $user->touch();

<a name="soft-deleting"></a>
## 일시 삭제

모델을 일시 삭제 할 때, 그것은 실제로 데이터베이스에서 제거 되지는 않습니다. 대신, 해당 레코드에 `deleted_at` 타임스탬프가 설정됩니다. 모델의 일시 삭제를 활성화 시키려면, 모델에 `SoftDeletes`를 사용하세요:

    use Illuminate\Database\Eloquent\SoftDeletes;

    class User extends Model {

        use SoftDeletes;

        protected $dates = ['deleted_at'];

    }

테이블에 `deleted_at` 컬럼을 추가하려면 마이그레이션에서 `softDeletes` 메서드를 사용하면 됩니다:

    $table->softDeletes();

이제, 모델에서 `delete` 메서드를 호출하면, `deleted_at` 컬럼이 현재의 타임스탬프로 설정됩니다. 일시 삭제를 사용하는 모델을 쿼리 할때, "삭제"된 모델은 쿼리 결과에 포함되지 않습니다.

#### 일시 삭제된 모델들을 강제로 결과에 포함시키기

일시 삭제된 모델들을 결과 세트에 나타나도록 하려면, 쿼리에서 `withTrashed` 메서드를 사용합니다:

    $users = User::withTrashed()->where('account_id', 1)->get();

`withTrashed` 메서드는 관계를 정의 할때도 사용될 수 있습니다:

    $user->posts()->withTrashed()->get();

만약 일시 삭제된 모델들**만** 결과로 받으려면, `onlyTrashed` 메서드를 사용하비다.:

    $users = User::onlyTrashed()->where('account_id', 1)->get();

일시 삭제된 모델을 활성 상태로 복구하려면, `restore` 메서드를 사용합니다:

    $user->restore();

쿼리에서 `restore` 메서드를 사용 할 수도 있습니다:

    User::withTrashed()->where('account_id', 1)->restore();

`withTrashed`처럼, `restore` 메서드 역시 관계형에서 쓰일 수 있습니다.:

    $user->posts()->restore();

만약 모델을 데이터베이스에서 실제로 제거하고 싶다면, `forceDelete` 메서드를 사용합니다:

    $user->forceDelete();

`forceDelete` 메서드 역시 관계형에서 작동 합니다:

    $user->posts()->forceDelete();

해당 모델의 인스턴스가 일시 삭제됬는지 확인하려면, `trashed` 메서드를 사용합니다:

    if ($user->trashed())
    {
        //
    }

<a name="timestamps"></a>
## 타임스탬프

기본적으로, 엘로퀀트가 데이터베이스 테이블의 `created_at`과 `updated_at` 컬럼을 자동으로 관리합니다. 간단하게 `timestamp` 컬럼들을 테이블에 추가하세요. 그러면 엘로퀀트가 나머지를 알아서 할겁니다. 만약 이 컬럼들을 엘로퀀트가 관리하길 원하지 않는 다면 다음의 속성을 모델에 추가하세요:

#### 타임스탬프 자동 관리 비활성화

    class User extends Model {

        protected $table = 'users';

        public $timestamps = false;

    }

#### 사용자 정의 타임스탬프 형식 제공

만약 타임스탬프의 형식을 사용자 지정하려면, 모델의 `getDateFormat` 메서드를 재정의 하세요:

    class User extends Model {

        protected function getDateFormat()
        {
            return 'U';
        }

    }

<a name="query-scopes"></a>
## 쿼리 스코프

#### 쿼리 스코프 정의

스코프는 모델에서 쿼리 로직을 쉽게 재사용 할수 있도록 해줍니다. 스코프를 정의하려면, 간단히 모델의 메서드에 `scope` 접두사를 추가하세요:

    class User extends Model {

        public function scopePopular($query)
        {
            return $query->where('votes', '>', 100);
        }

        public function scopeWomen($query)
        {
            return $query->whereGender('W');
        }

    }

#### 쿼리 스코프 사용

    $users = User::popular()->women()->orderBy('created_at')->get();

#### 동적 스코프

때로는 매개변수를 받는 스코프를 정의해야 할 수도 있습니다. 단순히 스코프 함수에 매개변수를 추가하세요:

    class User extends Model {

        public function scopeOfType($query, $type)
        {
            return $query->whereType($type);
        }

    }

그런 다음 스코프 호출에 매개변수를 전달 하세요:

    $users = User::ofType('member')->get();

<a name="global-scopes"></a>
## 전역 스코프

때때로 모델에서 실행되는 모든 쿼리에 적용되는 스코프를 정의 해야 할 수도 있습니다. 본질적으로, 이것이 어떻게 라라벨의 "일시 삭제" 기능이 작동 하는지 보여줍니다. 전역 스코프는 PHP 트레이트와 `Illuminate\Database\Eloquent\ScopeInterface` 구현 클래스를 혼합하여 정의됩니다.

첫번째로, 트레이트를 정의합니다. 이 예제에서, 우리는 라라벨에 포함된 `SoftDeletes`를 사용합니다:

    trait SoftDeletes {

        /**
         * 모델의 일시 삭제 트레이트를 부팅
         *
         * @return void
         */
        public static function bootSoftDeletes()
        {
            static::addGlobalScope(new SoftDeletingScope);
        }

    }

만약 엘로퀀트 모델이 `bootNameOfTrait` 명명 규칙과 일치하는 메서드를 가지고있는 트레이트를 사용한다면, 해당 트레이트는 엘로퀀트 모델이 부팅될때 호출되고, 전역 스코프를 등록하도록, 또는 여러분이 원하는 어떠한 일이든 할수 있는 기회를 제공합니다. 스코프는 무조건 `apply`와 `remove`를 명시하고 있는 `ScopeInterface`를 구현해야만 합니다.

`apply` 메서드는 `Illuminate\Database\Eloquent\Builder` 쿼리 빌더와 적용할 `Model`을 수신하고, 스코프에 추가하고자 하는 `where`절을 추가하는 역할을 합니다. `remove` 메서드 역시 `Builder` 객체와 `Model`를 수신하고, `apply` 메서드가 취한 액션을 되돌리는 역할을 합니다. 다시 말하면, `remove`는 추가된 `where` 절 (또는 다른 절)을 제거하는데 사용됩니다. 그러므로, `SoftDeletingScope`에서 해당 메서드들은 다음과 같습니다:

    /**
     * 주어진 엘로퀀트 쿼리 빌더에 스코프를 적용.
     *
     * @param  \Illuminate\Database\Eloquent\Builder  $builder
     * @param  \Illuminate\Database\Eloquent\Model  $model
     * @return void
     */
    public function apply(Builder $builder, Model $model)
    {
        $builder->whereNull($model->getQualifiedDeletedAtColumn());

        $this->extend($builder);
    }

    /**
     * 주어진 엘로퀀트 쿼리 빌더에 스코프를 제거
     *
     * @param  \Illuminate\Database\Eloquent\Builder  $builder
     * @param  \Illuminate\Database\Eloquent\Model  $model
     * @return void
     */
    public function remove(Builder $builder, Model $model)
    {
        $column = $model->getQualifiedDeletedAtColumn();

        $query = $builder->getQuery();

        foreach ((array) $query->wheres as $key => $where)
        {
            // 만약 where 절이 일시 삭제 날짜 제약 조건이라면, 쿼리로부터 제거하고
            // where 배열 키를 리셋 합니다. 이는 개발자가 레이지 로드된
            // 관계형 결과 세트에서 삭제된 모델을 포함할 수 있도록 해줍니다.
            if ($this->isSoftDeleteConstraint($where, $column))
            {
                unset($query->wheres[$key]);

                $query->wheres = array_values($query->wheres);
            }
        }
    }

<a name="relationships"></a>
## 관계

물론, 여러분의 테이블들은 아마도 다른 테이블과 연관되어 있을겁니다. 예를 들어, 하나의 블로그 포스트는 여러개의 코멘트를 갖고 있을 수도 있고, 하나의 주문은 주문을 한 사용자와 연관되어 있을 수도 있습니다. 엘로퀀트는 이러한 관계를 쉽게 관리하고, 작동 할 수 있도록 해줍니다. 라라벨은 여러 타입의 관계형을 제공합니다:

- [One To One](#one-to-one)
- [One To Many](#one-to-many)
- [Many To Many](#many-to-many)
- [Has Many Through](#has-many-through)
- [Polymorphic Relations](#polymorphic-relations)
- [Many To Many Polymorphic Relations](#many-to-many-polymorphic-relations)

<a name="one-to-one"></a>
### One To One

#### One To One 관계 정의

One To One 관계는 가장 기본적인 관계입니다. 예를 들어, 하나의 `User` 모델은 하나의 `Phone`을 갖고 있을 수 있습니다. 이런 관계를 엘로퀀트에서는 다음과 같이 정의합니다:

    class User extends Model {

        public function phone()
        {
            return $this->hasOne('App\Phone');
        }

    }

'hasOne` 메서드에 전달된 첫번째 인수는 연관된 모델명 입니다. 이러한 관계가 정의되고 나면, 엘로퀀트의 [동적 속성](#dynamic-properties)을 사용하여 그것을 조회 할 수 있습니다.(#dynamic-properties):

    $phone = User::find(1)->phone;

위의 통해 수행되는 SQL 문은 다음과 같습니다:

    select * from users where id = 1

    select * from phones where user_id = 1

엘로퀀트는 모델 명을 기반으로 관계된 외래 키를 가정하는것에 주의하세요. 이 경우, `Phone` 모델은 `user_id` 외래 키를 사용 한다고 가정합니다. 만약 이러한 규칙을 재정의 하고 싶다면, `hasOne` 메서드의 두번째 인수로 외래 키로 사용할 컬럼명을 전달합니다. 나아가, 세번째 인수에 어떤 로컬 컬럼이 연결에 사용될지도 명시를 할 수 있습니다:

    return $this->hasOne('App\Phone', 'foreign_key');

    return $this->hasOne('App\Phone', 'foreign_key', 'local_key');

#### 역관계 정의

`Phone` 모델에 역관계를 정의하려면, `belongsTo`를 사용합니다:

    class Phone extends Model {

        public function user()
        {
            return $this->belongsTo('App\User');
        }

    }

위의 예제에서, 엘로퀀트는 `phone` 테이블에서 `user_id`를 찾습니다. 만약 다른 외래 키를 정의하고 싶다면, `belongsTo` 메서드의 두번째 인수에 전달합니다.:

    class Phone extends Model {

        public function user()
        {
            return $this->belongsTo('App\User', 'local_key');
        }

    }

추가적으로, 부모 테이블과 연결된 컬럼 명을 새번째 인수에 전달 할 수도 있습니다:

    class Phone extends Model {

        public function user()
        {
            return $this->belongsTo('App\User', 'local_key', 'parent_key');
        }

    }

<a name="one-to-many"></a>
### One To Many

One To Many 관계는 "여러개"의 코멘트를 갖고있는 하나의 블로그 포스트를 예로 들 수 있습니다. 이러한 관계는 아래처럼 정의 할 수 있습니다:

    class Post extends Model {

        public function comments()
        {
            return $this->hasMany('App\Comment');
        }

    }

이제 우리는 [동적 속성](#dynamic-properties)을 통해 포스트의 코멘트들을 액세스 할 수 있습니다:

    $comments = Post::find(1)->comments;

만약 어떠한 코멘트들을 조회할 것인지 추가적인 제약 조건이 필요하다면, `comments` 메서드를 호출하여 조건들을 계속 체이닝 할 수도 있습니다:

    $comments = Post::find(1)->comments()->where('title', '=', 'foo')->first();

또, `hasMany` 메서드에 두번째 인수를 전달하여, 외래 키 규칙을 재정의 할 수도 있습니다. 그리고, `hasOne` 관계처럼, 로컬 컬럼 키 또한 명시 될 수 있습니다:

    return $this->hasMany('App\Comment', 'foreign_key');

    return $this->hasMany('App\Comment', 'foreign_key', 'local_key');

#### 역관계 정의

`Comment` 모델에 역관계를 정의하려면, `belongsTo` 메서드를 사용합니다:

    class Comment extends Model {

        public function post()
        {
            return $this->belongsTo('App\Post');
        }

    }

<a name="many-to-many"></a>
### Many To Many

Many To Many 관계는 좀 더 복잡한 관계 유형입니다. 이러한 관계는 많은 역할들을 갖고 있는 사용자와 또한 해당 역할이 다른 사용자들과도 공유되고 있는 상황을 예로 들 수 있습니다. 예를 들어, 여러명의 사용자가 "관리자" 역할을 갖고 있을 수 있습니다. 이러한 관계에는 `users`, `roles`, 그리고 `role_user` 3개의 데이터베이스 테이블이 필요합니다. `role_user` 테이블은 연관된 모델명들의 알파벳 순서에 의해 테이블명이 지어졌고, `user_id`와 `role_id` 컬럼을 포함하고 있어야만 합니다.

우리는 `belongsToMany` 메서드를 사용하여 Many To Many 관계를 정의 할 수 있습니다:

    class User extends Model {

        public function roles()
        {
            return $this->belongsToMany('App\Role');
        }

    }

이제, 우리는 `User` 모델을 통해 해당 역할을 조회 할 수 있습니다:

    $roles = User::find(1)->roles;

만약 피벗 테이블에 불규칙한 테이블 명을 사용하고 싶다면, `belongsToMany` 메서드의 두번째 인수에 테이블 명을 전달 하세요:

    return $this->belongsToMany('App\Role', 'user_roles');

규칙된 관련 키들 또한 재정의 할 수 있습니다:

    return $this->belongsToMany('App\Role', 'user_roles', 'user_id', 'foo_id');

물론, `Role`모델에 역관계를 정의 할 수도 있습니다:

    class Role extends Model {

        public function users()
        {
            return $this->belongsToMany('App\User');
        }

    }

<a name="has-many-through"></a>
### Has Many Through

"has many through" 관계는 중간 관계를 통해 떨어져 있는 관계를 액세스 할 수 있도록 편리한 바로 가기를 제공합니다. 예를 들어, 하나의 `Country` 모델은 `User` 모델을 통해 많은 `Post`를 갖고 있을 수 있습니다. 이러한 관계의 테이블들은 다음과 같습니다:

    countries
        id - integer
        name - string

    users
        id - integer
        country_id - integer
        name - string

    posts
        id - integer
        user_id - integer
        title - string

`posts` 테이블이 `country_id` 컬럼을 갖고 있지 않더라고, `hasManyThrough` 관계는 `$country->posts`를 통해 국가의 포스트들을 액세스 할 수 있도록 해줍니다. 이러한 관계를 정의 해봅시다:

    class Country extends Model {

        public function posts()
        {
            return $this->hasManyThrough('App\Post', 'App\User');
        }

    }

관계된 키들을 수동으로 명시하려면, 해당 메서드의 세번째, 네번째 인수로 전달합니다:

    class Country extends Model {

        public function posts()
        {
            return $this->hasManyThrough('App\Post', 'App\User', 'country_id', 'user_id');
        }

    }

<a name="polymorphic-relations"></a>
### Polymorphic 관계

Polymorphic 관계는 한번의 연결로 하나 이상의 다른 모델에 속할 수 있도록 해줍니다. 예를 들어, 스태프 모델이나 주문 모델에 속하는 사진 모델이 있을 수 있습니다. 이러한 모델은 다음과 같이 정의 합니다:

    class Photo extends Model {

        public function imageable()
        {
            return $this->morphTo();
        }

    }

    class Staff extends Model {

        public function photos()
        {
            return $this->morphMany('App\Photo', 'imageable');
        }

    }

    class Order extends Model {

        public function photos()
        {
            return $this->morphMany('App\Photo', 'imageable');
        }

    }

#### Polymorphic 관계 조회

이제 스태프 맴버 또는 주문의 사진을 조회 할 수 있습니다:

    $staff = Staff::find(1);

    foreach ($staff->photos as $photo)
    {
        //
    }

#### Polymorphic 관계의 소유자 조회

그렇지만, 잔짜 "polymorphic" 매직은 `Photo` 모델에서 스태프 또는 주문을 엑세스 할 때 일어납니다:

    $photo = Photo::find(1);

    $imageable = $photo->imageable;

`Photo` 모델의 `imageable` 관계는 어떤 유형의 모델이 해당 사진을 소유했는지에 따라 `Staff` 또는 `Order` 인스턴스를 반환합니다.

#### Polymorphic 관계 테이블 구조

어떻게 이런 일이 가능한지 이해를 돕기 위해, polymorphic 관계의 데이터베이스 구조를 살펴봅시다:

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

여기서 주목해야할 핵심 필드는 `photo` 테이블의 `imageable_id`와 `imageable_type` 입니다. ID는 ID값을 포함(이 예제에서는 소유자 스태프 또는 주문)하고 type은 소유 모델의 클래스명을 포함합니다. 이것이 `imageable` 관계를 액세스 할 때, ORM이 어떤 유형의 소유 모델을 반환해야 하는지 결정 해줍니다.

<a name="many-to-many-polymorphic-relations"></a>
### Many To Many Polymorphic 관계

#### Polymorphic Many To Many 관계 테이블 구조

기존의 polymorphic 관계뿐만 아니라, 여러분은 many-to-many polymorphic 관계 또한 지정 할 수 있습니다. 예를 들어, 하나의 블로그 `Post`와 `Video` 모델은 하나의 `Tag` 모델에 polymorphic 관계를 공유 할 수 있습니다. 첫번째로, 테이블 구조를 살펴봅시다:

    posts
        id - integer
        name - string

    videos
        id - integer
        name - string

    tags
        id - integer
        name - string

    taggables
        tag_id - integer
        taggable_id - integer
        taggable_type - string

다음으로, 우리는 모델에 관계를 설정 할 준비가 되었습니다. `Post`와 `Video` 두 모델은 `tags` 메서드를 통해 `morphToMany` 관계를 가지게 됩니다:

    class Post extends Model {

        public function tags()
        {
            return $this->morphToMany('App\Tag', 'taggable');
        }

    }

`Tag` 모델은 관계에 대한 각각의 메서드를 정의 할 수 있습니다:

    class Tag extends Model {

        public function posts()
        {
            return $this->morphedByMany('App\Post', 'taggable');
        }

        public function videos()
        {
            return $this->morphedByMany('App\Video', 'taggable');
        }

    }

<a name="querying-relations"></a>
## 관계 쿼리

#### SELECT문 관계를 쿼리

모델의 레코드를 액세스 할 때, 관계의 존재 여부에 따라 결과를 제한 시키고 싶을 수 도 있습니다. 예를 들어, 최소한 한개의 코멘트를 갖고 있는 모든 블로그 포스트를 조회 해야 할 때도 있습니다. 이렇게 하려면, `has` 메서드를 사용 합니다:

    $posts = Post::has('comments')->get();

여러분은 또한 연산자와 카운트를 지정 할 수도 있습니다:

    $posts = Post::has('comments', '>=', 3)->get();

"도트" 표기법을 사용하여 중첩된 `has` 문 또한 사용 할 수 있습니다:

    $posts = Post::has('comments.votes')->get();

더 많은 기능을 원한다면, `has` 쿼리에 "where" 조건을 추가해주는 `whereHas`와 `orWhereHas` 메서드를 사용 할 수도 있습니다:

    $posts = Post::whereHas('comments', function($q)
    {
        $q->where('content', 'like', 'foo%');

    })->get();

<a name="dynamic-properties"></a>
### 동적 속성

엘로퀀트는 동적 속성을 통해 여러분의 관계들을 엑세스 할 수 있도록 해줍니다. 엘로퀀트는 여러분을 위해 자동으로 관계를 로드하고, 심지어 `get` (one-to-many 관계의 경우) 또는 `first` (one-to-one  관계의 경우) 중 어떤 메서드를 호출해야 할지 알만큼 똑똑합니다. 그런 다음 관계와 똑같은 이름을 사용하는 동적 속성을 통해 엑세스 될 수 있습니다. 예를 들어, 다음의 `$phone` 모델을 보시죠:

    class Phone extends Model {

        public function user()
        {
            return $this->belongsTo('App\User');
        }

    }

    $phone = Phone::find(1);

사용자의 이메일을 아래와 같이 표시하는 것 보다:

    echo $phone->user()->first()->email;

아래처럼 더 단순하게 단축 될 수 있습니다:

    echo $phone->user->email;

> **주의:** 많은 결과를 반환하는 관계는 `Illuminate\Database\Eloquent\Collection` 클래스 인스턴스를 반환합니다.

<a name="eager-loading"></a>
## 즉시 로드 (Eager Loading)

즉시 로드는 N + 1 쿼리 문제를 개선하기 위해 존재 합니다. 예를 들어, `Author` 모델에 연관되어 있는 `Book` 모델을 생각해보세요. 관계는 다음처럼 정의 될 수 있습니다:

    class Book extends Model {

        public function author()
        {
            return $this->belongsTo('App\Author');
        }

    }

이제 다음의 코드를 생각해보세요:

    foreach (Book::all() as $book)
    {
        echo $book->author->name;
    }

이 반복은 테이블의 모든 책을 조회하는 1개의 쿼리를 실행하고, 저자를 조회하는 또 다른 쿼리를 각각의 책마다 실행합니다. 그러므로, 만약 25의 책이 있다면, 해당 반복은 26개의 쿼리를 실행하게 됩니다.

고맙게도, 우리는 쿼리의 수를 크게 감소 시켜주는 즉시 로딩을 사용 할 수 있습니다. 즉시 로드 되어야할 관계는 `with` 메서드를 통해 지정 할 수 있습니다:

    foreach (Book::with('author')->get() as $book)
    {
        echo $book->author->name;
    }

위의 반복에서는, 오직 2개의 쿼리가 실행됩니다:

    select * from books

    select * from authors where id in (1, 2, 3, 4, 5, ...)

즉시 로드의 현명한 사용은 여러분의 어플리케이션 성능을 크게 증가 시킬 수 있습니다.

물론, 여러분은 한번에 여러개의 관계를 즉시 로드 할 수도 있습니다:

    $books = Book::with('author', 'publisher')->get();

중첩된 관계까지도 즉시 로드 할 수 있습니다:

    $books = Book::with('author.contacts')->get();

위의 예제에서, `author` 관계가 즉시 로드되고, 저자의 `contacts` 관계 역시 로드 됩니다.

### 즉시 로드 제약 조건

때때로 여러분은 관계를 즉시 로드하길 원하는 동시에, 조건을 지정해야 할 때도 있습니다. 아래에 예제가 있습니다:

    $users = User::with(['posts' => function($query)
    {
        $query->where('title', 'like', '%first%');

    }])->get();

이 예제에서, 우리는 사용자의 포스트를 즉시 로드하지만, 포스트의 제목이 "first" 단어를 포함하고 있는 포스트만 조회합니다.

물론, 즉시 로드 클로저가 "제약 조건"에만 한정된것은 아닙니다. 여러분은 순서 또한 적용 할 수 있습니다:

    $users = User::with(['posts' => function($query)
    {
        $query->orderBy('created_at', 'desc');

    }])->get();

### 지연된 즉시 로드 (Lazy Eager Loading)

이미 존재하는 모델 콜렉션에서 직접 연관된 모델들을 즉시 로드 하는 일 또한 가능합니다. 이는 연관된 모델들의 로드 여부를 동적으로 결정 할때나 캐싱과 함께 동작 할 때 유용합니다.

    $books = Book::all();

    $books->load('author', 'publisher');

여러분은 또한 쿼리에 제약 조건을 설정하는 클로저를 전달 할 수도 있습니다:

    $books->load(['author' => function($query)
    {
        $query->orderBy('published_date', 'asc');
    }]);

<a name="inserting-related-models"></a>
## 관련 모델 삽입

#### 관련 모델 부여

여러분은 종종 새로운 관련 모델을 삽입 해야 할 때가 있습니다. 예를 들어, 포스트에 새로운 코멘트를 삽입 해야 할 수도 있습니다. 코멘트 모델에 `post_id` 외래 키를 직접 설정 하는일 대신, 해당 모델의 부모 `Post` 모델에서 직접 새로운 코멘트를 삽입 할 수 있습니다:

    $comment = new Comment(['message' => 'A new comment.']);

    $post = Post::find(1);

    $comment = $post->comments()->save($comment);

이 예제에서, `post_id` 필드는 삽입된 코멘트 레코드에 자동으로 설정됩니다.

만약 여러개의 관련 모델을 저장해야 한다면 아래처럼 할 수 있습니다:

    $comments = [
        new Comment(['message' => 'A new comment.']),
        new Comment(['message' => 'Another comment.']),
        new Comment(['message' => 'The latest comment.'])
    ];

    $post = Post::find(1);

    $post->comments()->saveMany($comments);

### 연관 모델들 (Belongs To)

`belongsTo` 관계를 업데이트 할 때는, `associate` 메서드를 사용합니다. 이 메서드는 자식 모델에 외래 키를 설정합니다:

    $account = Account::find(10);

    $user->account()->associate($account);

    $user->save();

### 관련 모델들 삽입 (Many To Many)

Many-to-many 관계를 작업 할때, 관련 모델들을 삽입 해야 할 수도 있습니다. `User`와 `Role` 모델을 예제로 계속 사용 하겠습니다. 우리는 `attach` 메서드를 사용하여 쉽게 사용자에게 새로운 역할을 부여 할 수 있습니다:

#### Many To Many 모델 부여

    $user = User::find(1);

    $user->roles()->attach(1);

여러분은 또한 피벗 테이블의 해당 관계에 저장되는 속성을 배열로 전달 할 수도 있습니다:

    $user->roles()->attach(1, ['expires' => $expires]);

물론, `attach`의 반대는 `detach` 입니다:

    $user->roles()->detach(1);

`attach`와 `detach` 두가지 모두 역시 ID 배열을 받을 수도 있습니다:

    $user = User::find(1);

    $user->roles()->detach([1, 2, 3]);

    $user->roles()->attach([1 => ['attribute1' => 'value1'], 2, 3]);

#### 동기화를 사용하여 Many To Many 모델 부여

여러분은 `sync` 메서드를 사용하여 관련 모델들을 부여 할 수도 있습니다. `sync` 메서드는 피벗 테이블에 넣을 ID를 배열로 받습니다. 이 연산이 끝나고나면, 배열에 있는 ID 들만 해당 모델의 피벗 테이블에 존재하게 됩니다:

    $user->roles()->sync([1, 2, 3]);

#### 동기화 할때 피벗 데이터 추가

여러분은 또한 주어진 ID들의 피벗 테이블 데이터를 연결 할 수도 있습니다:

    $user->roles()->sync([1 => ['expires' => true]]);

때때로, 한번의 커맨드로 새로운 관련 모델을 생성하고 부여하길 원할 수도 있습니다. 이러한 연산은, `save` 메서드를 사용합니다:

    $role = new Role(['name' => 'Editor']);

    User::find(1)->roles()->save($role);

이 예제에서, 새로운 `Role` 모델은 데이터베이스에 저장되고 사용자 모델에 부여 됩니다. 또한 속성 배열을 전달하여 피벗 테이블에 데이터를 저장 할 수 있습니다:

    User::find(1)->roles()->save($role, ['expires' => $expires]);

<a name="touching-parent-timestamps"></a>
## 부모 타임스탬프 터치

하나의 모델이 다른 모델에 `속할 때(belongsTo)`, `Post`에 속하는 `Comment`처럼, 가끔 자식 모델을 업데이트 할 때 부모의 타임스탬프를 업데이트 하는것이 도움이 될 때도 있습니다. 예를 들어, `Comment` 모델이 업데이트 되면, 부모 `Post`의 `updated_at` 타임스탬프 값이 자동으로 업데이트 되길 바랄 수도 있습니다. 엘로퀀트는 이 일을 쉽게 만들어줍니다. 그냥 자식 모델에 관련 모델 명을 포함하고 있는 `touches` 속성을 추가하기만 하세요:

    class Comment extends Model {

        protected $touches = ['post'];

        public function post()
        {
            return $this->belongsTo('App\Post');
        }

    }

이제, `Comment`를 업데이트 할 때마다, 해당 코멘트를 소유하고 있는 `Post`는 `updated_at` 컬럼을 항상 업데이트 합니다:

    $comment = Comment::find(1);

    $comment->text = 'Edit to this comment!';

    $comment->save();

<a name="working-with-pivot-tables"></a>
## 피벗 테이블과 작업

여러분이 이미 배운바와 같이, many-to-many 관계를 사용 할때는 서로 연결해주는 중간 테이블이 필요합니다. 엘로퀀트는 이 테이블과 상호작용 하는데 매우 도움이 되는 여러 방법들을 제공합니다. 예를 들어, `User` 객체는 연관된 많은 `Role` 객체를 가지고 있다고 가정하겠습니다. 이 관계를 엑세스 한 뒤, 우리는 모델의 `pivot` 테이블을 액세스 할 수도 있습니다:

    $user = User::find(1);

    foreach ($user->roles as $role)
    {
        echo $role->pivot->created_at;
    }

우리가 조회한 각각의 `Role` 모델은 자동으로 `pivot` 속성이 부여되어 있다는걸 명심하세요. 이 속성은 연결 테이블 모델을 나타내며, 다른 엘로퀀트 모델처럼 사용 될 수 있습니다.

기본적으로, `pivot` 객체에 키들만 나타나게 됩니다. 만약 피벗 테이블의 추가 속성들을 포함하길 원한다면, 관계를 정의 할때 속성들을 명시해야만 합니다:

    return $this->belongsToMany('App\Role')->withPivot('foo', 'bar');

이제 `Role`모델의 `pivot` 객체에서 `foo`와 `bar` 속성들을 액세스 할 수 있습니다.

만약 피벗 테이블의 `created_at`과 `updated_at` 타임스탬프가 자동으로 관리되길 원한다면, 관계를 정의 할 때 `withTimestamps` 메서드를 사용하세요:

    return $this->belongsToMany('App\Role')->withTimestamps();

#### 피벗 테이블의 레코드를 삭제

해당 모델의 모든 피벗 테이블 레코드를 삭제하려면, `detach` 메서드를 사용합니다:

    User::find(1)->roles()->detach();

이 연산은 `roles` 테이블의 레코드를 삭제하는것이 아니라, 피벗 테이블의 레코드만 삭제한다는걸 명심하세요.

#### 피벗 테이블의 레코드를 업데이트

때로는 피벗 테이블을 삭제하지않고, 업데이트만 해야 할 숟 욌습니다. 만약 피벗 테이블을 업데이트하길 원한다면, 아래처럼 `updateExistingPivot` 메서드를 사용합니다:

    User::find(1)->roles()->updateExistingPivot($roleId, $attributes);

#### 사용자 정의 피벗 테이블 정의

라라벨은 또한 여러분이 사용자 정의 피벗 테이블을 정의 할 수 있도록 해줍니다. 사용자 정의 모델을 정의하려면, 첫번째로 `Eloquent`를 확장하는 여러분의 "기본" 모델을 생성하세요. 여러분의 다른 엘로퀀트 모델들에서, 기본 `Eloquent` 대신 해당 사용자 정의 기본 모델을 확장 하세요. 여러분의 기본 모델에서, 사용자 정의 피벗 모델을 인스턴스를 반환하는 다음의 함수를 추가하세요:

    public function newPivot(Model $parent, array $attributes, $table, $exists)
    {
        return new YourCustomPivot($parent, $attributes, $table, $exists);
    }

<a name="collections"></a>
## 컬렉션

`get` 메서드나 `relationship`을 통해 엘로퀀트로부터 반환되는 모든 다중-결과 세트는, 컬렉션 객체를 반환합니다. 이 객체는 PHP의 `IteratorAggregate` 인터페이스를 구현하므로 배열처럼 반복될 수 있습니다. 이 외에도, 이 객체는 결과 세트와 작업 할 수 있는 많은 유용한 메서드들 또한 포함하고 있습니다.

#### 컬렉션이 키를 포함하고 있는지 확인

예를 들어, 우리는 `contains` 메서드를 사용하여 주어진 기본 키가 결과 세트에 포함되어있는지 확인 할 ㅅ 윘습니다:

    $roles = User::find(1)->roles;

    if ($roles->contains(2))
    {
        //
    }

콜렉션은 또한 배열이나 JSON으로 변환 될 수도 있습니다:

    $roles = User::find(1)->roles->toArray();

    $roles = User::find(1)->roles->toJson();

만약 콜렉션이 문자열로 캐스팅된다면, 해당 콜렉션은 JSON으로 반환됩니다:

    $roles = (string) User::find(1)->roles;

#### 콜렉션 반복

엘로퀀트 콜렉션은 각 아이템들을 반복하고 필터를 해주는 몇개의 유용한 메서드를 포함하고 있습니다:

    $roles = $user->roles->each(function($role)
    {
        //
    });

#### 콜렉션 필터링

콜렉션을 필터링 할 때, 제공된 콜백은 [array_filter](http://php.net/manual/en/function.array-filter.php)의 콜백으로 사용됩니다.

    $users = $users->filter(function($user)
    {
        return $user->isAdmin();
    });

> **주의:** 콜렉션을 필터링하고 JSON으로 변환할 때, `values` 함수를 먼저 호출하여 배열의 키들을 초기화 할수 있도록 하세요.

#### 각각의 콜렉션 객체에 콜백을 적용

    $roles = User::find(1)->roles;

    $roles->each(function($role)
    {
        //
    });

#### 콜렉션 값을 기준으로 정렬

    $roles = $roles->sortBy(function($role)
    {
        return $role->created_at;
    });

#### 콜렉션 값을 기준으로 정렬

    $roles = $roles->sortBy('created_at');

#### 사용자 정의 콜렉션 타입 반환

때때로, 여러분이 추가한 메서드와 함께 사용자 정의 콜렉션 객체를 반환하길 원할수도 있습니다. 여러분의 엘로퀀트 모델에 `newCollection` 메서드를 재정의하여 이를 지정 할 수 있습니다:

    class User extends Model {

        public function newCollection(array $models = [])
        {
            return new CustomCollection($models);
        }

    }

<a name="accessors-and-mutators"></a>
## 접근자 & 설정자

#### 접근자 정의

엘로퀀트는 모델의 속성에 접근하거나 설정 할 때 해당 속성을 변형시켜주는 편리한 방법을 제공합니다. 간단히 모델에 `getFooAttribute` 메서드를 정의하여 접근자를 선언하세요. 데이터베이스의 컬럼이 snake 표기법 일지라도 메서드는 camel 표기법을 따라야 한다는 것을 명심하세요:

    class User extends Model {

        public function getFirstNameAttribute($value)
        {
            return ucfirst($value);
        }

    }

위의 예제에서, `first_name` 컬럼은 접근자 메서드를 갖고 있습니다. 해당 속성의 값이 접근자의 인수로 전달됩니다.

#### 설정자 정의

설정자 역시 같은 형식으로 선언 됩니다:

    class User extends Model {

        public function setFirstNameAttribute($value)
        {
            $this->attributes['first_name'] = strtolower($value);
        }

    }

<a name="date-mutators"></a>
## 날짜 설정자

기본적으로, 엘로퀀트는 `created_at`과 `updated_at` 컬럼을 유용한 메서드들을 갖고 있으며 네이티브 PHP `DateTime` 클래스를 확장한 [Carbon](https://github.com/briannesbitt/Carbon)의 인스턴스로 변환시킵니다.

여러분은 어떤 필드들이 자동으로 설정될 것인지 커스터마이징 할 수 있으며, 또한 모델의 `getDates` 메서드를 재정의 하여 설정자을 완전이 비활성화 시킬수도 있습니다:

    public function getDates()
    {
        return ['created_at'];
    }

컬럼이 날짜라고 생각되면, 여러분은 UNIX 타임스탬프, date 문자열 (`Y-m-d`), date-time 문자열, 그리고 `DateTime` / `Carbon` 인스턴스 중 하나로 설정 할 수 있습니다.

날짜 설정자를 완전히 비활성화 시키려면, `getDates` 메서드에서 빈 배열을 반환하세요:

    public function getDates()
    {
        return [];
    }

<a name="attribute-casting"></a>
## 속성 캐스팅

만약 항상 다른 데이터 형식으로 변환 시키고자 하는 속성들이 있다면, 모델에 `casts` 속성을 추가하면 됩니다. 그렇지않으면, 여러분은 각각의 속성들마다 설정자를 정의하는데 시간을 낭비해야 합니다. 아래에 `casts` 속성의 사용 예제가 있습니다:

    /**
     * 기본 유형으로 캐스팅 되어야 하는 속성들
     *
     * @var array
     */
    protected $casts = [
        'is_admin' => 'boolean',
    ];

이제 여러분이 `is_admin` 속성을 액세스 할때, 이 속성이 데이터베이스에 integer로 저장되어 있더라도 항상 boolean으로 캐스팅 됩니다. 지원하는 다른 캐스트 형식은 다음과 같습니다: `integer`, `real`, `float`, `double`, `string`, `boolean`, `object` 그리고 `array`.

`array` 캐스트는 직렬화(serialized)되어 저장된 JSON 컬럼을 사용 할 때 유용합니다. 예를 들어, 만약 여러분의 데이터베이스가 시리얼라이즈된 JSON을 포함하고 있는 TEXT 형식을 갖고 있다면, 그 속성에 `array` 캐스팅을 추가하면 여러분이 엘로퀀트 모델에서 해당 속성을 액세스 할때 이 속성은 자동으로 PHP 배열 형태로 역직렬화(deserialize)됩니다:

    /**
     * 기본 유형으로 캐스팅 되어야 하는 속성들
     *
     * @var array
     */
    protected $casts = [
        'options' => 'array',
    ];

이제, 엘로퀀트 모델을 아래처럼 사용할 수 있습니다:

    $user = User::find(1);

    // $options는 배열 형태입니다...
    $options = $user->options;

    // options는 자동으로 JSON 형태로 직렬화 됩니다...
    $user->options = ['foo' => 'bar'];

<a name="model-events"></a>
## 모델 이벤트

엘로퀀트 모델은 다음의 메서드들을 사용하여 모델의 다양한 라이프사이클 포인트를 후크(hook)할 수 있도록 여러개의 이벤트들을 실행합니다: `creating`, `created`, `updating`, `updated`, `saving`, `saved`, `deleting`, `deleted`, `restoring`, `restored`.

새로운 아이템이 처음으로 저장될 때, `creating`과 `created` 이벤트들이 실행 됩니다. 만약 새로운 아이템이 아니고 `save` 메서드가 호출되면, `updating` / `updated` 이벤트들이 실행됩니다. 두가지 케이스 모두 `saving` / `saved` 이벤트들이 실행됩니다.

#### 이벤트를 통한 저장 연산 취소

만약 `creating`, `updating`, `saving`, 또는 `deleting` 이벤트들로부터 `false`가 반환되면, 해당 액션은 취소 됩니다:

    User::creating(function($user)
    {
        if ( ! $user->isValid()) return false;
    });

#### 이벤트 리스너는 어디에 등록 하는가

`EventServiceProvider` 공급자는 여러분의 모델 이벤트 바인딩들을 등록할 수 있는 편리한 장소를 제공합니다. 예를 들어:

    /**
     * 여러분 어플리케이션의 어떠한 이벤트라도 등록
     *
     * @param  \Illuminate\Contracts\Events\Dispatcher  $events
     * @return void
     */
    public function boot(DispatcherContract $events)
    {
        parent::boot($events);

        User::creating(function($user)
        {
            //
        });
    }

<a name="model-observers"></a>
## 모델 관찰자

모델 이벤트들의 처리를 통합하기 위해, 여러분은 모델 관찰자를 등록 할 수 있습니다. 관찰자 클래스는 다양한 모델 이벤트들에 해당하는 메서드들을 갖고 있습니다. 예를 들어, `creating`, `updating`, `saving` 메서드들은 다른 모델의 이벤트 이름에서뿐만 아니라, 관찰자에 있을 수도 있습니다.

그러므로, 예를 들면, 모델 관찰자는 아래처럼 보일 수 있습니다:

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

여러분은 `observe` 메서드를 사용하여 관찰자 인스턴스를 등록 할 수 있습니다:

    User::observe(new UserObserver);

<a name="model-url-generation"></a>
## 모델 URL 생성

모델을 `route` 또는 `action` 메서드로 전달 할때, 모델의 기본 키가 생성된 URI에 추가됩니다. 예제를 봅시다:

    Route::get('user/{user}', 'UserController@show');

    action('UserController@show', [$user]);

이 예제에서 `$user->id` 속성이 생성된 URL의 `{user}` 자리표시자에 삽입됩니다. 하지만, 만약 ID 대신 다른 속성을 사용하고 싶다면, 모델의 `getRouteKey` 메서드를 재정의 하면됩니다:

    public function getRouteKey()
    {
        return $this->slug;
    }

<a name="converting-to-arrays-or-json"></a>
## 배열 / JSON으로 변환

#### 모델을 배열로 변환

JSON API를 빌드 할때, 여러분은 종종 모델 그리고 관련된 모델들을 배열이나 JSON으로 변환해야 할 때가 있습니다. 그러므로, 엘로퀀트는 이러한 일들을 위한 메서드를 포함하고 있습니다. 모델과 모델에 로드된 관계들을 배열로 변환하려면, `toArray` 메서드를 사용합니다:

    $user = User::with('roles')->first();

    return $user->toArray();

모델 콜렉션 전체 또한 배열로 변환 될 수 있습니다:

    return User::all()->toArray();

#### 모델을 JSON으로 변환

모델을 JSON으로 변환하려면, `toJson` 메서드를 사용합니다:

    return User::find(1)->toJson();

#### 라우트에서 모델을 반환

모델이나 콜렉션이 문자열로 캐스팅될때, JSON으로 변환 된다는것을 명심하세요. 다시말하면, 여러분의 어플리케이션 라우트에서 엘로퀀트 객체를 직접 반환 할 수도 있습니다!

    Route::get('users', function()
    {
        return User::all();
    });

#### 배열 또는 JSON 변환에서 특정 속성들을 숨기기

때때로 비밀번호같이 모델의 배열이나 JSON에 포함되는 속성에 제한을 걸어야 할 수도 있습니다. 이렇게 하려면, 여러분의 모델에 `hidden` 속성을 정의 하세요:

    class User extends Model {

        protected $hidden = ['password'];

    }

> **주의:** 관계 모델의 속성을 숨길때는, 동적 접근자 명이 아닌 관계의 **메서드**명을 사용하세요.

다른 한편으로는, `visible` 속성을 사용하여 화이트리스트를 지정 할 수 있습니다:

    protected $visible = ['first_name', 'last_name'];

<a name="array-appends"></a>
가끔, 데이터베이스에 일치하지 않는 컬럼 속성을 배열에 추가해야 할 때도 있습니다. 이렇게 하려면, 해당 값의 접근자를 정의하세요:

    public function getIsAdminAttribute()
    {
        return $this->attributes['admin'] == 'yes';
    }

접근자를 생성하고 나면, 모델의 `appends` 속성에 해당 값을 추가하세요:

    protected $appends = ['is_admin'];

해당 속성이 `appends` 목록에 추가 되고 나면, 모델의 배열과 JSON 폼에 포함됩니다. `appends` 목록에 있는 속성들은 모델의 `visible`과 `hidden` 설정을 따릅니다.
