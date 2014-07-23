# 스키마 빌더

- [소개](#introduction)
- [테이블 생성 & 삭제](#creating-and-dropping-tables)
- [컬럼 추가](#adding-columns)
- [컬럼명 변경](#renaming-columns)
- [컬럼 삭제](#dropping-columns)
- [존재 여부 확인](#checking-existence)
- [인덱스 추가](#adding-indexes)
- [왜래키](#foreign-keys)
- [인덱스 삭제](#dropping-indexes)
- [스토리지 엔진](#storage-engines)

<a name="introduction"></a>
## 소개

Laravel `Schema` 클래스는 테이블 생성 하는데 관대한 방법을 제공합니다. Laravel에서 제공하는 모든 데이터베이스와 잘 작동하며 통합된 API을 제공합니다.

<a name="creating-and-dropping-tables"></a>
## 테이블 생성 & 삭제

`Schema::create` 메소드를 사용하여 새로운 데이터베이스 테이블을 생성합니다.:

    Schema::create('users', function($table)
  	{
  		$table->increments('id');
  	});

`create` 메소드에 전달되는 첫번째 인수는 테이블의 이름이며, 두번째는 새로운 테이블을 정의하는 `Blueprint` 객체를 받는 `Closure` 입니다.

이미 존재하는 데이터베이스 테이블명을 변경하려면, `rename` 메소드를 사용합니다.:

	Schema::rename($from ,$to);

`Schema::connection` 메소드를 사용하여 스키마 작업이 어떤 커넥션을 사용할 지 지정 할 수 있습니다.:

	Schema::connection('foo')->create('users', function($table)
	{
		$table->increments('id');
	});

`Schema::drop` 메소드를 사용 하여 테이블을 삭제 할 수 있습니다.:

	Schema::drop('users');

	Schema::dropIfExists('users');

<a name="adding-columns"></a>
## 컬럼 추가

이미 존재하는 테이블을 수정하려면 `Schema::table` 메소드를 사용합니다.:

	Schema::table('users', function($table)
	{
		$table->string('email');
	});

테이블 빌더는 테이블을 빌드 할때 사용할 수 있는 다양한 컬럼 타입들을 포함하고 있습니다.:

커맨드  | 설명
------------- | -------------
`$table->increments('id');`  |  테이블의 Incrementing ID (primary key).
`$table->bigIncrements('id');`  |  테이블의 "big integer"와 동일한 Incrementing ID.
`$table->string('email');`  |  VARCHAR 컬럼과 동일
`$table->string('name', 100);`  |  길이가 지정된 VARCHAR 컬럼과 동일
`$table->integer('votes');`  |  테이블의 INTEGER와 동일
`$table->bigInteger('votes');`  |  테이블의 BIGINT와 동일
`$table->smallInteger('votes');`  |  테이블의 SMALLINT와 동일
`$table->float('amount');`  |  테이블의 FLOAT과 동일
`$table->decimal('amount', 5, 2);`  |  precision과 scale이 지정된 DECIMAL과 동일
`$table->boolean('confirmed');`  |  테이블의 BOOLEAN과 동일
`$table->date('created_at');`  |  테이블의 DATE와 동일
`$table->dateTime('created_at');`  |  테이블의 DATETIME과 동일
`$table->time('sunrise');`  |  테이블의 TIME과 동일
`$table->timestamp('added_on');`  |  테이블의 TIMESTAMP와 동일
`$table->timestamps();`  |  **created\_at** 과 **updated\_at** 컬럼 추가
`$table->softDeletes();`  |  소프트 삭제에 쓰이는 **deleted\_at** 컬럼 추가
`$table->text('description');`  |  테이블의 TEXT와 동일
`$table->binary('data');`  |  테이블의 BLOB과 동일
`$table->enum('choices', array('foo', 'bar'));` | 테이블의 ENUM과 동일
`->nullable()`  |  컬럼이 NULL 값을 허용하도록 지정
`->default($value)`  |  컬럼의 기본값 선언
`->unsigned()`  |  UNSIGNED INTEGER 설정

만약 MySQL 데이터베이스를 사용하고 있다면, `after` 메소드를 사용하여 컬럼 순서를 지정 할 수 있습니다.:

**MySQL에서 after 사용**

	$table->string('name')->after('email');

<a name="renaming-columns"></a>
## 컬럼명 변경

스키마 빌더의 `renameColumn` 메소드를 사용하여 컬럼명을 변경 할 수 있습니다.:

**컬럼명 변경**

	Schema::table('users', function($table)
	{
		$table->renameColumn('from', 'to');
	});

> **노트:** `enum` 타입의 컬럼명 변경은 제공되지 않습니다.

<a name="dropping-columns"></a>
## 컬럼 삭제

**데이터베이스 테이블에서 컬럼 삭제**

	Schema::table('users', function($table)
	{
		$table->dropColumn('votes');
	});

**데이터베이스 테이블에서 다수의 컬럼 삭제**

	Schema::table('users', function($table)
	{
		$table->dropColumn('votes', 'avatar', 'location');
	});

<a name="checking-existence"></a>
## 존재 여부 확인

`hasTable`와 `hasColumn` 메소드를 사용하여 테이블이나 컬럼이 존재하는지 쉽게 확인 할 수 있습니다.:

**테이블의 존재 여부 확인**

	if (Schema::hasTable('users'))
	{
		//
	}

**컬럼의 존재 여부 확인**

	if (Schema::hasColumn('users', 'email'))
	{
		//
	}

<a name="adding-indexes"></a>
## 인덱스 추가

스키마 빌더는 여러 종류의 인덱스를 제공 합니다. 인덱스를 추가하는데는 두가지 방법이 있습니다. 컬럼을 정의할때 같이 정의하거나 아니면 따로 추가 할수도 있습니다.:

**컬럼과 인덱스 생성**

	$table->string('email')->unique();

또는 분리된 라인에 인덱스를 추가하도록 선택할 수도 있습니다. 아래는 사용 가능한 인덱스 타입입니다.:

커맨드  | 설명
------------- | -------------
`$table->primary('id');`  |  primary key 추가
`$table->primary(array('first', 'last'));`  |  composite keys 추가
`$table->unique('email');`  |  unique index 추가
`$table->index('state');`  |  basic index 추가

<a name="foreign-keys"></a>
## 외래키

라라벨은 테이블에 외래키 제약조건 추가를 제공합니다:

**테이블에 외래키 추가**

	$table->foreign('user_id')->references('id')->on('users');

이 예제는, `user_id` 컬럼이 `users` 테이블의 `id` 컬럼을 참조한다고 명시합니다.

또한, "on delete"와 "on update" 제약 조건을 옵션으로 명시 할 수도 있습니다.:

	$table->foreign('user_id')
          ->references('id')->on('users')
          ->onDelete('cascade');

외래키를 삭제하려면, `dropForeign` 메소드를 사용합니다. 다른 인덱스들에 사용된것과 비슷한 명명규칙이 외래키에 사용됩니다.:

	$table->dropForeign('posts_user_id_foreign');

> **노트:** incrementing integer를 참조하는 외래키를 만들때, 외래키 컬럼은 항상 `unsigned`여야 합니다.

<a name="dropping-indexes"></a>
## 인덱스 삭제

인덱스를 삭제하려면 반드시 인덱스의 이름을 명시해야 합니다. Laravel은 적절한 이름을 기본적으로 인덱스에 부여합니다. 간단히 테이블명, 인덱스의 컬럼명, 인텍스 타입을 연결하여 이름을 부여합니다. 아래의 예를 보세요.:

커맨드  | 설명
------------- | -------------
`$table->dropPrimary('users_id_primary');`  |  "users" 테이블의 primary key 삭제
`$table->dropUnique('users_email_unique');`  |  "users" 테이블의 unique index 삭제
`$table->dropIndex('geo_state_index');`  |  "geo" 테이블의 basic index 삭제

<a name="storage-engines"></a>
## 스토리지 엔진

테이블의 스토리지 엔진을 설정 하려면, 스키마 빌더의 `engine` 속성을 설정하면 됩니다.:

    Schema::create('users', function($table)
    {
        $table->engine = 'InnoDB';

        $table->string('email');
    });
