# 페이지네이션

- [설정](#configuration)
- [사용법](#usage)
- [페이지네이션 링크에 추가](#appending-to-pagination-links)

<a name="configuration"></a>
## 설정

다른 프레임워크에서 페이지네이션은 매우 성가신 일이지만, Laravel에서는 식은 죽 먹기 입니다. `app/config/view.php` 파일 안에는 단 한개의 설정 옵션만 존재합니다. `pagination` 옵션은 어떤 뷰를 사용하여 페이지네이션 링크를 생성할지 지정합니다. 기본적으로 Laravel은 2가지의 뷰를 포함하고 있습니다.

`pagination::simple` 뷰가 간단하게 "이전"과 "다음" 버튼을 보여주는 반면, `pagination::slider` 뷰는 현재 페이지를 근거로 링크의 "범위"를 지능적으로 보여줍니다. **두개의 뷰 모두 트위터 부트스트랩에 특화되어 있습니다.**

<a name="usage"></a>
## 사용법

아이템을 페이징 하는데는 여러가지 방법이 있습니다. 가장 쉬운 방법은 쿼리 빌더나 엘로퀀트 모델에 `paginate` 메소드를 사용하는것 입니다.

**데이터베이스 결과 페이징**

    $users = DB::table('users')->paginate(15);

[Eloquent(엘로퀀트)](/docs/eloquent) 모델을 페이징 할수도 있습니다.:

**엘로퀀트 모델 페이징**

	$users = User::where('votes', '>', 100)->paginate(15);

`paginate` 메소드에 전달되는 인수는 페이지마다 표시될 아이템의 갯수 입니다. 결과가 조회된 후에는 뷰에 아이템들을 표시하고 `links` 메소드를 사용하여 페이지네이션 링크를 생성할 수 있습니다.:

	<div class="container">
		<?php foreach ($users as $user): ?>
			<?php echo $user->name;
		<?php endforeach; ?>
	</div>

	<?php echo $users->links(); ?>

페이지네이션 시스템을 만드는 필요한게 이것 뿐입니다! 프레임워크에게 현재 페이지를 알려줄 필요가 없었다는 것을 기억하세요. Laravel이 자동으로 알아낼겁니다.

또한 다음의 메소드를 사용해 페이지네이션의 추가 정보를 액세스 할 수 있습니다.:

- `getCurrentPage`
- `getLastPage`
- `getPerPage`
- `getTotal`
- `getFrom`
- `getTo`

가끔 수동으로 아이템 배열을 첫번째 인수로 전달하여 페이지네이션을 생성하고 싶을때도 있을겁니다. 이럴땐 `Paginator::make` 메소드를 사용합니다.:

**수동으로 페이지네이션 생성**

	$paginator = Paginator::make($items, $totalItems, $perPage);

**페이지네이터 URI 커스터 마이징**

`setBaseUrl` 메소드를 사용하여 페이지네이터에서 사용되는 URI를 커스터 마이징 할수도 있습니다.:

	$users = User::paginate();

	$user->setBaseUrl('custom/url');

이 예제는 다음과 같은 URL을 생성합니다.: http://example.com/custom/url?page=2

<a name="appending-to-pagination-links"></a>
## 페이지네이션 링크에 추가

페이지네이터의 `appends` 메소드를 사용하여 페이지네이션 링크에 쿼리 스트링을 추가 할 수 있습니다.:

	<?php echo $users->appends(array('sort' => 'votes'))->links(); ?>

이는 아래와 같은 URL을 생성합니다.:

	http://example.com/something?page=2&sort=vote