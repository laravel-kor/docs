# 기본 데이터베이스 사용법

- [구성](#configuration)
- [읽기 / 쓰기 커넥션](#read-write-connections)
- [쿼리 실행](#running-queries)
- [데이터베이스 트랜잭션](#database-transactions)
- [커넥션 엑세스](#accessing-connections)
- [쿼리 로깅](#query-logging)

<a name="configuration"></a>
## 구성

라라벨은 데이터베이스 연결과 쿼리 실행을 매우 쉽게 만들어 줍니다. 데이터베이스 구성 파일은 `config/database.php`입니다. 이 파일에서 여러분은 모든 데이터베이스 연결들을 정의하고 어떤 연결이 기본으로 사용되어야 하는지 지정합니다. 지원하는 모든 데이터베이스 시스템의 예제가 이 파일에 제공 되어있습니다.

현재 라라벨은 다음 4종류의 데이터베이스 시스템을 지원합니다: MySQL, Postgres, SQLite, 그리고 SQL Server.

<a name="read-write-connections"></a>
## 읽기 / 쓰기 커넥션

때때로 여러분은 SELECT 구문과 INSERT, UPDATE, DELETE 구문에 각각 다른 데이터베이스 커넥션을 사용해야 할 수도 있습니다. 라라벨은 이를 매우 쉽게 할 수 있도록 해주며, 원시 쿼리, 쿼리 빌더, 또는 엘로퀀트 ORM 중 무엇을 사용하든지 항상 올바른 연결이 사용될 수 있도록 해줍니다.

읽기 / 쓰기 연결이 어떻게 구성 되는지 보려면 다음의 예제를 살펴봅시다:

    'mysql' => [
        'read' => [
            'host' => '192.168.1.1',
        ],
        'write' => [
            'host' => '196.168.1.2'
        ],
        'driver'    => 'mysql',
        'database'  => 'database',
        'username'  => 'root',
        'password'  => '',
        'charset'   => 'utf8',
        'collation' => 'utf8_unicode_ci',
        'prefix'    => '',
    ],

구성 배열에 `read`와 `write` 2개의 키가 추가된것을 확인하세요. 이들 키 모두 하나의 `host` 키를 포함하고 있는 배열을 갖고 있습니다. `read`와 `write` 연결의 나머지 데이터베이스 옵션들은 메인 `mysql` 배열로부터 병합됩니다. 그러므로, 우리는 메인 배열로부터 재정의 할 옵션들만 `read`와 `write` 배열에 추가하면 됩니다. 그러므로, 이 케이스에서는 `192.168.1.1` 아이피가 "읽기" 연결로 사용되며, `192.168.1.2` 아이피는 "쓰기" 연결로 사용됩니다. 메인 `mysql` 배열의 데이터베이스 자격증명, 접두사, 케릭터 세트, 그리고 모든 다른 옵션들이 두개의 연결을 통하여 공유됩니다.

<a name="running-queries"></a>
## 쿼리 실행

데이터베이스 연결을 구성하고 나면, `DB` 파사드를 사용하여 쿼리를 실행 할 수 있습니다.

#### Select 쿼리 실행

    $results = DB::select('select * from users where id = ?', [1]);

`select` 메서드는 항상 `array`를 결과로 반환합니다.

#### Insert 구문 실행

    DB::insert('insert into users (id, name) values (?, ?)', [1, 'Dayle']);

#### Update 구문 실행

    DB::update('update users set votes = 100 where name = ?', ['John']);

#### Delete 구문 실행

    DB::delete('delete from users');

> **주의:** `update`와 `delete` 구문은 연산에 영향받은 행의 갯수를 반환합니다.

#### 일반 구문 실행

    DB::statement('drop table users');

#### 쿼리 이벤트 수신

`DB::listen` 메서드를 사용하여 쿼리 이벤트를 수신 할 수 있습니다:

    DB::listen(function($sql, $bindings, $time)
    {
        //
    });

<a name="database-transactions"></a>
## 데이터베이스 트랜잭션

데이터베이스 트랜잭션 내에서 연산을 실행하려면 `transaction` 메서드를 사용합니다:

    DB::transaction(function()
    {
        DB::table('users')->update(['votes' => 1]);

        DB::table('posts')->delete();
    });

> **주의:** `transaction` 클로저에서 던져진 모든 예외는 트랜잭션이 자동으로 롤백되도록 합니다.

어떤 경우에는 직접 트랜잭션을 시작 해야 할 때도 있습니다:

    DB::beginTransaction();

`rollback` 메서드를 사용하여 트랜잭션을 롤백 할 수 있습니다:

    DB::rollback();

마지막으로 `commit` 메서드를 통해 트랜잭션을 커밋 할 수 있습니다:

    DB::commit();

<a name="accessing-connections"></a>
## 커넥션 엑세스

여러개의 커넥션을 사용 할때, `DB::connection` 메서드를 사용하여 해당 커넥션을 엑세스 할 수 있습니다.:

    $users = DB::connection('foo')->select(...);

또한 여러분은 기본 PDO 인스턴스를 엑세스 할 수도 있습니다:

    $pdo = DB::connection()->getPdo();

때때로 여러분은 주어진 데이터베이스로 재연결을 해야 할 수도 있습니다:

    DB::reconnect('foo');

만약 여러분이 기본 PDO 인스턴스의 `max_connections` 제한때문에 연결된 데이터베이스로부터 연결을 해제해야 한다면, `disconnect` 메서드를 사용하세요:

    DB::disconnect('foo');

<a name="query-logging"></a>
## 쿼리 로깅

라라벨은 옵션으로 현재 요청에 실행된 모든 쿼리를 메모리에 로그 할 수 있습니다. 많은 수의 행을 입력할때처럼 어떤 경우에는 어플리케이션이 과도한 메모리를 사용할 수 있다는 점에 유의하세요. 로그를 허용하려면 `enableQueryLog` 메서드를 사용하세요:

    DB::connection()->enableQueryLog();

실행된 쿼리 배열을 얻으려면, `getQueryLog` 메서드를 사용하세요:

    $queries = DB::getQueryLog();
