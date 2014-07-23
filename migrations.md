# 마이그레이션 & 씨딩

- [소개](#introduction)
- [마이그레이션 생성](#creating-migrations)
- [마이그레이션 실행](#running-migrations)
- [마이그레이션 롤백](#rolling-back-migrations)
- [데이터베이스 씨딩](#database-seeding)

<a name="introduction"></a>
## 소개

마이그레이션은 데이터베이스 버전 컨트롤의 한 종류 입니다. 개발팀이 데이터베이스 스키마를 수정하고 현재 스키마에 대한 최신 상태를 유지 할 수 있게 해줍니다. 마이그레이션은 어플리케이션의 스키마를 쉽게 다루기 위해 보통 [스키마 빌더](/docs/schema)와 같이 이루어져 있습니다.

<a name="creating-migrations"></a>
## 마이그레이션 생성

마이그레이션을 생성하려면 Artisan CLI에서 `migrate:make` 커맨드를 사용합니다.:

**마이그레이션 생성**

    php artisan migrate:make create_users_table

마이그레이션은 `app/database/migrations` 폴더에 위치하며, 프레임워크가 마이그레이션의 순서를 결정할수 있도록 파일명에 타임스탬프를 포함하고 있습니다.

마이그레이션을 생성할때 `--path` 옵션을 지정 할 수도 있습니다. 경로는 루트 디렉토리에 상대적 이어야합니다.:

	  php artisan migrate:make foo --path=app/migrations

또한 `--table`과 `--create` 옵션을 사용하여 테이블명과 마이그레이션이 새로운 테이블을 만드는지 여부를 나타내는 옵션을 사용 할 수 있습니다.

	  php artisan migrate:make create_users_table --table=users --create

<a name="running-migrations"></a>
## 마이그레이션 실행

**처리되지 않은 모든 마이그레이션 실행**

	php artisan migrate

**해당경로의 처리되지 않은 모든 마이그레이션 실행**

	php artisan migrate --path=app/foo/migrations

**해당 패키지의 처리되지 않은 모든 마이그레이션 실행**

	php artisan migrate --package=vendor/package

> **노트:** 마이그레이션을 실행하는 동안 "class not found" 에러가 발생한다면, `composer update` 커맨드를 실행한 후, 재시도 해보세요

<a name="rolling-back-migrations"></a>
## 마이그레이션 롤백

**마지막 마이그레이션 롤백**

	php artisan migrate:rollback

**모든 마이그레이션 롤백**

	php artisan migrate:reset

**모든 마이그레이션 롤백 후, 모든 마이그레이션 실행**

	php artisan migrate:refresh

	php artisan migrate:refresh --seed

<a name="database-seeding"></a>
## 데이터베이스 씨딩

Laravel은 seed 클래스를 사용하여 데이터베이스에 테스트 데이터를 씨드하는 간단한 방법을 포함하고 있습니다. 모든 seed 클래스는 `app/database/seeds` 폴더에 저장되어 있습니다. Seed 클래스는 원하는 어떠한 이름을 사용 하여도 상관 없지만, `UserTableSeeder`와 같이 분별있는 규칙성을 따르는것이 좋습니다. 기본적으로 `DatabaseSeeder` 클래스가 정의 되어있습니다. 이 클래스에서 씨딩 순서를 조절 할 수 있도록 해주는 `call` 메소드를 사용하여 다른 seed 클래스를 실행할 수 있습니다.

**데이터베이스 seed 클래스 예제**

	class DatabaseSeeder extends Seeder {

		public function run()
		{
			$this->call('UserTableSeeder');

			$this->command->info('User table seeded!');
		}

	}

	class UserTableSeeder extends Seeder {

		public function run()
		{
			DB::table('users')->delete();

			User::create(array('email' => 'foo@bar.com'));
		}

	}

Artisan CLI 에서 `db:seed` 커맨드를 사용하여 데이터베이스를 씨딩 할 수 있습니다.:

	php artisan db:seed

또한 모든 마이그레이션을 록백하고 다시 실행하는 `migrate:refresh` 커맨드를 사용하여 테이터베이스를 씨딩 할 수 있습니다.:

	php artisan migrate:refresh --seed
