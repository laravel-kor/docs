# 쿼리 빌더

- [소개](#introduction)
- [조회(Selects)](#selects)
- [조인(Joins)](#joins)
- [고급 where 절](#advanced-wheres)
- [집계](#aggregates)
- [가공되지 않은 표현식](#raw-expressions)
- [삽입(Inserts)](#inserts)
- [수정(Updates)](#updates)
- [삭제(Deletes)](#deletes)
- [조합(Unions)](#unions)
- [쿼리 캐시](#caching-queries)

<a name="introduction"></a>
## 소개

데이터베이스 쿼리 빌더는 데이터베이스 쿼리를 만들고 실행하는 편리하고 능수능란한 인터페이스를 제공합니다. 어플리케이션에서 대부분의 데이터베이스 작업을 수행하는데 사용될 수 있으며, 지원되는 모든 데이터베이스 시스템에서 작동합니다.

> **노트:** Laravel 쿼리 빌더는 SQL injection 공격으로부터 어플리케이션을 보호하기 위해 PDO의 매개 변수 바인딩을 사용합니다. 그러므로 바인딩되는 문자열을 다듬을 필요가 없습니다.

<a name="selects"></a>
## 조회(Selects)

**테이블의 모든 행 조회**

    $users = DB::table('users')->get();
  
  	foreach ($users as $user)
  	{
  		var_dump($user->name);
  	}

**테이블의 단일 행 조회**

	$user = DB::table('users')->where('name', 'John')->first();

	var_dump($user->name);

**단일 행에서 단일 항목 조회**

	$name = DB::table('users')->where('name', 'John')->pluck('name');

**컬럼 값 리스트 조회**

	$roles = DB::table('roles')->lists('title');

이 메소드는 role title을 배열로 반환합니다. 또한 사용자 정의 컬럼 키를 배열키로 지정 할 수도 있습니다.:

	$roles = DB::table('roles')->lists('title', 'name');

**조회절(select) 지정**

	$users = DB::table('users')->select('name', 'email')->get();

	$users = DB::table('users')->distinct()->get();

	$users = DB::table('users')->select('name as user_name')->get();

**이미 만든 쿼리에 select 절 추가**

	$query = DB::table('users')->select('name');

	$users = $query->addSelect('age')->get();

**where 사용**

	$users = DB::table('users')->where('votes', '>', 100)->get();

**or where 사용**

	$users = DB::table('users')
	                    ->where('votes', '>', 100)
	                    ->orWhere('name', 'John')
	                    ->get();

**where between 사용**

	$users = DB::table('users')
	                    ->whereBetween('votes', array(1, 100))->get();

**배열과 함께 where in 사용**

	$users = DB::table('users')
	                    ->whereIn('id', array(1, 2, 3))->get();

	$users = DB::table('users')
	                    ->whereNotIn('id', array(1, 2, 3))->get();

**값이 지정되지 않은 레코드를 찾는 where null 사용**

	$users = DB::table('users')
	                    ->whereNull('updated_at')->get();

**Order By, Group By, Having**

	$users = DB::table('users')
	                    ->orderBy('name', 'desc')
	                    ->groupBy('count')
	                    ->having('count', '>', 100)
	                    ->get();

**Offset & Limit**

	$users = DB::table('users')->skip(10)->take(5)->get();

<a name="joins"></a>
## 조인(Joins)

또한 쿼리 빌더는 조인(join)문을 작성하는데 사용할 수 있습니다. 다음의 예제를 보세요.:

**기본적인 조인(join)문**

	DB::table('users')
	            ->join('contacts', 'users.id', '=', 'contacts.user_id')
	            ->join('orders', 'users.id', '=', 'orders.user_id')
	            ->select('users.id', 'contacts.phone', 'orders.price');

또한 좀더 고급의 조인(join)절을 명시할 수 있습니다.:

	DB::table('users')
	        ->join('contacts', function($join)
	        {
	        	$join->on('users.id', '=', 'contacts.user_id')->orOn(...);
	        })
	        ->get();

<a name="advanced-wheres"></a>
## 고급 where 절

"where exists"나 중첩된 매개 변수를 그룹화하는 좀더 고급의 where 절을 사용할 수도 있습니다. Laravel 쿼리 빌더는 이러한 것들 또한 처리할 수 있습니다.:

**매개 변수 그룹화**

	DB::table('users')
	            ->where('name', '=', 'John')
	            ->orWhere(function($query)
	            {
	            	$query->where('votes', '>', 100)
	            	      ->where('title', '<>', 'Admin');
	            })
	            ->get();

위의 쿼리는 다음과 같은 SQL을 생성합니다.:

	select * from users where name = 'John' or (votes > 100 and title <> 'Admin')

**Exists절**

	DB::table('users')
	            ->whereExists(function($query)
	            {
	            	$query->select(DB::raw(1))
	            	      ->from('orders')
	            	      ->whereRaw('orders.user_id = users.id');
	            })
	            ->get();

위의 쿼리는 다음과 같은 SQL을 생성합니다.:

	select * from users
	where exists (
		select 1 from orders where orders.user_id = users.id
	)

<a name="aggregates"></a>
## 집계

또한 쿼리빌더는 `count`, `max`, `min`, `avg`, `sum` 같이 다양한 집계 메소드를 제공합니다.

**집계 메소드 사용**

	$users = DB::table('users')->count();

	$price = DB::table('orders')->max('price');

	$price = DB::table('orders')->min('price');

	$price = DB::table('orders')->avg('price');

	$total = DB::table('users')->sum('votes');

<a name="raw-expressions"></a>
## 가공되지 않은 표현식

가끔 쿼리에서 가공되지 않은 표현식을 사용하길 원할수도 있습니다. 이러한 표현식은 쿼리에 문자열로 주입되므로 SQL injection 지점를 만들지 않도록 주의하십시오! 가공되지 않은 표현식을 만들려면, `DB::raw` 메소드를 사용합니다.:

**가공되지 않은 표현식 사용**

	$users = DB::table('users')
	                     ->select(DB::raw('count(*) as user_count, status'))
	                     ->where('status', '<>', 1)
	                     ->groupBy('status')
	                     ->get();

**항목의 값을 증가 또는 감소**

	DB::table('users')->increment('votes');

	DB::table('users')->increment('votes', 5);

	DB::table('users')->decrement('votes');

	DB::table('users')->decrement('votes', 5);

또한, 추가적으로 업데이트 할 컬럼들 또한 명시 할 수 있습니다.:

	DB::table('users')->increment('votes', 1, array('name' => 'John'));

<a name="inserts"></a>
## 삽입(Inserts)

**테이블에 레코드 삽입**

	DB::table('users')->insert(
		array('email' => 'john@example.com', 'votes' => 0)
	);

만약 테이블에 auto-incrementing id가 있다면, `insertGetId` 메소드를 사용하여 레코드를 삽입하고 id를 조회할 수 있습니다.:

**Auto-Increamenting ID와 함께 테이블에 레코드 삽입**

	$id = DB::table('users')->insertGetId(
		array('email' => 'john@example.com', 'votes' => 0)
	);

> **노트:** PostgreSQL를 사용할 경우, insertGetId 메소드는 auto-increamting 항목 이름이 "id"로 명명되길 요구합니다.

**테이블에 여러개의 레코드 삽입**

	DB::table('users')->insert(array(
		array('email' => 'taylor@example.com', 'votes' => 0),
		array('email' => 'dayle@example.com', 'votes' => 0),
	));

<a name="updates"></a>
## 수정(Updates)

**테이블의 레코드 수정**

	DB::table('users')
	            ->where('id', 1)
	            ->update(array('votes' => 1));

<a name="deletes"></a>
## 삭제(Deletes)

**테이블의 레코드 삭제**

	DB::table('users')->where('votes', '<', 100)->delete();

**테이블의 모든 레코드 삭제**

	DB::table('users')->delete();

**truncate을 사용하여 모든 레코드 삭제**

	DB::table('users')->truncate();

<a name="unions"></a>
## 조합(Unions)

쿼리 빌더는 두 쿼리를 함께 "조합" 해주는 빠른 방법을 제공합니다.:

**쿼리 조합 수행**

	$first = DB::table('users')->whereNull('first_name');

	$users = DB::table('users')->whereNull('last_name')->union($first)->get();

`unionAll` 메소드 역시 제공되며, `union` 과 같은 특징을 갖고 있습니다.`.

<a name="caching-queries"></a>
## 쿼리 캐시

`remember` 메소드를 사용하여 쿼리 결과를 쉽게 캐싱 할 수 있습니다.:

**쿼리 결과 캐싱**

	$users = DB::table('users')->remember(10)->get();

이 예제에서, 쿼리 결과는 10분 동안 캐시됩니다. 결과가 캐시됐을 경우, 데이터베이스에 쿼리를 수행하지 않으며, 결과는 어플리케이션에 명시된 기본 캐시 드라이버에서 불러집니다.