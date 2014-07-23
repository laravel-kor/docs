# 검증

- [기본적인 사용법](#basic-usage)
- [에러 메시지와 함께 작업](#working-with-error-messages)
- [에러 메시지와 뷰](#error-messages-and-views)
- [사용 가능한 검증 규칙](#available-validation-rules)
- [사용자 정의 에러 메시지](#custom-error-messages)
- [사용자 정의 검증 규칙](#custom-validation-rules)

<a name="basic-usage"></a>
## 기본적인 사용법

Laravel은 `Validation` 클래스를 통해 데이터를 검증하고 에러 메시지를 조회하는 간편하고 편리한 기능을 포함하고 있습니다.

**기본적인 검증 예제**

    $validator = Validator::make(
  		array('name' => 'Dayle'),
  		array('name' => 'required|min:5')
  	);

`make` 메소드의 첫번째 인수는 검증할 데이터 이며, 두번째 인수는 데이터에 적용될 규칙들 입니다.

다중 규칙은 "파이프" 문자를 사용하여 구분하거나, 배열에 별도의 요소로 구분합니다.

**배열을 사용하여 규칙 지정**

	$validator = Validator::make(
		array('name' => 'Dayle'),
		array('name' => array('required', 'min:5'))
	);

`Validator` 인스턴스가 생성되고 나면, `fails` (또는 `passes`) 메소드를 사용하여 검증을 수행할 수 있습니다.

	if ($validator->fails())
	{
		// 주어진 데이터는 검증을 통과하지 못했습니다.
	}

만약 검증이 실패하면 검증기에서 에러 메시지를 조회할 수 있습니다.

	$messages = $validator->messages();

또한 `failed` 메소드를 사용하여 메시지없이 실패한 검증 규칙 배열을 액세스 할 수도 있습니다.

	$failed = $validator->failed();

**파일 검증**

`Validator` 클래스는 `size`나 `mimes`같은 파일 검증에 필요한 몇가지 규칙들을 제공합니다. 파일을 검증할때는 간단하게 다른 데이터와 함께 검증기로 파일을 전달하면 됩니다.

<a name="working-with-error-messages"></a>
## 에러 메시지와 함께 작업

`Validator` 인스턴스에서 `messages` 메소드를 호출하고 나면, 에러 메시지와 함께 다양한 작업을 할 수 있도록 편리한 메소드를 가지고 있는 `MessageBag` 인스턴스를 받습니다.

**해당 필드의 첫번째 에러 메시지 조회**

	echo $messages->first('email');

**해당 필드의 모든 에러 메시지 조회**

	foreach ($messages->get('email') as $message)
	{
		//
	}

**모든 필드의 모든 에러 메시지 조회**

	foreach ($messages->all() as $message)
	{
		//
	}

**해당 필드의 에러 메시지가 있는지 판단**

	if ($messages->has('email'))
	{
		//
	}

**주어진 포맷과 함께 해당 필드의 에러 메시지 조회**

	echo $messages->first('email', '<p>:message</p>');

> **노트:** 기본적으로 메시지는 트위터 부트스트랩 구문을 사용하여 포맷되어 있습니다.

**주어진 포맷과 함께 모든 메시지 조회**

	foreach ($messages->all('<li>:message</li>') as $message)
	{
		//
	}

<a name="error-messages-and-views"></a>
## 에러 메시지 & 뷰

검증을 수행하고 나면, 뷰로 에러 메시지를 돌려 보낼 쉬운 방법이 필요합니다. 이 방법은 편리하게 Laravel이 처리합니다. 다음의 예제 라우트를 참조하세요.:

	Route::get('register', function()
	{
		return View::make('user.register');
	});

	Route::post('register', function()
	{
		$rules = array(...);

		$validator = Validator::make(Input::all(), $rules);

		if ($validator->fails())
		{
			return Redirect::to('register')->withErrors($validator);
		}
	});

검증이 실패하면, `withErrors` 메소드를 사용하여 `Validator` 인스턴스를 리디렉트로 전달하는 것에 주목하십시오. 이 메소드는 다음 요청에서 사용할수 있도록 에러 메시지를 세션에 플래시 데이터로 저장합니다.

여기서, GET 라우트에서는 뷰에 에러 메시지를 바인딩 할 필요가 없다는 것에 주목하세요. 왜냐하면, Laravel은 항상 세션 데이터에서 에러를 확인하고, 에러가 있을 경우 자동으로 뷰에 에러 메시지를 바인딩 합니다. **그러므로 `$errors` 변수는 모든 요청의 모든 뷰에서 항상 사용 가능하다는 것을 알아두시고**, `$errors` 변수는 항상 정의되어 있다는 것을 알고 안전하게 사용될 수 있도록 해야 합니다. `$errors` 변수는 `MessageBag`의 인스턴스가 됩니다.

그러므로 리디렉션한 후, 뷰에서 자동으로 바인딩 된 `$errors` 변수를 사용할 수 있습니다.:

	<?php echo $errors->first('email'); ?>

<a name="available-validation-rules"></a>
## 사용 가능한 검증 규칙

아래는 사용 가능한 모든 규칙과 규칙의 기능들입니다.:

- [Accepted](#rule-accepted)
- [Active URL](#rule-active-url)
- [After (Date)](#rule-after)
- [Alpha](#rule-alpha)
- [Alpha Dash](#rule-alpha-dash)
- [Alpha Numeric](#rule-alpha-num)
- [Before (Date)](#rule-before)
- [Between](#rule-between)
- [Confirmed](#rule-confirmed)
- [Date](#rule-date)
- [Date Format](#rule-date-format)
- [Different](#rule-different)
- [E-Mail](#rule-email)
- [Exists (Database)](#rule-exists)
- [Image (File)](#rule-image)
- [In](#rule-in)
- [Integer](#rule-integer)
- [IP Address](#rule-ip)
- [Max](#rule-max)
- [MIME Types](#rule-mimes)
- [Min](#rule-min)
- [Not In](#rule-not-in)
- [Numeric](#rule-numeric)
- [Regular Expression](#rule-regex)
- [Required](#rule-required)
- [Required If](#rule-required-if)
- [Required With](#rule-required-with)
- [Required Without](#rule-required-without)
- [Same](#rule-same)
- [Size](#rule-size)
- [Unique (Database)](#rule-unique)
- [URL](#rule-url)

<a name="rule-accepted"></a>
#### accepted

검증에 포함된 필드는 _yes_나 _on_ 또는 _1_ 이어야 합니다. 서비스 약관의 수락을 검증할때 유용합니다.

<a name="rule-active-url"></a>
#### active_url

검증에 포함된 필드는 PHP의 `checkdnsrr` 함수에 따른 유효한 URL이어야 합니다.

<a name="rule-after"></a>
#### after:_date_

검증에 포함된 필드는 주어진 날짜 이후의 값이어야 합니다. 전달된 날짜는 PHP의 `strtotime` 함수에 전달됩니다.

<a name="rule-alpha"></a>
#### alpha

검증에 포함된 필드는 모두 알파벳으로 된 문자여야 합니다.

<a name="rule-alpha-dash"></a>
#### alpha_dash

검증에 포함된 필드는 알파벳이나 숫자뿐만 아니라 대시나 밑줄로 된 문자여야 합니다.

<a name="rule-alpha-num"></a>
#### alpha_num

검증에 포함된 필드는 알파벳과 숫자로만 된 문자여야 합니다.

<a name="rule-before"></a>
#### before:_date_

검증에 포함된 필드는 주어진 날짜 이전의 값이어야 합니다. 전달된 날짜는 PHP의 `strtotime` 함수에 전달됩니다.

<a name="rule-between"></a>
#### between:_min_,_max_

검증에 포함된 필드는 주어진 _min(최소)_, _max(최대)_ 사이의 크기여야 합니다. 문자, 숫자, 파일은 `size` 규칙처럼 평가 됩니다.

<a name="rule-confirmed"></a>
#### confirmed

검증에 포함된 필드는 `foo_confirmation`의 필드와 일치해야 합니다. 예를 들어, 필드가 `password`라면, 일치하는 `password_confirmation` 필드가 input에 존재해야 합니다.

<a name="rule-date"></a>
#### date

검증에 포함된 필드는 PHP의 `strtotime` 함수에 따른 유효한 날짜여야 합니다.

<a name="rule-date-format"></a>
#### date_format:_format_

검증에 포함된 필드는 PHP의 `date_parse_from_format` 함수에 따라 정의된 _format(포맷)_과 일치해야 합니다.

<a name="rule-different"></a>
#### different:_field_

검증에 포함된 필드는 주어진 _field(필드)_와 달라야 합니다.

<a name="rule-email"></a>
#### email

검증에 포함된 필드는 e-mail 주소 형식이어야 합니다.

<a name="rule-exists"></a>
#### exists:_table_,_column_

검증에 포함된 필드는 주어진 데이터베이스 테이블과 컬럼으로 존재해야 합니다.

**Exists규칙의 기본 사용법**

	'state' => 'exists:states'

**사용자 정의 컬럼명 명시**

	'state' => 'exists:states,abbreviation'

또한 쿼리의 "where" 절로 추가되는 조건을 더 명시 할 수도 있습니다.:

	'email' => 'exists:staff,email,account_id,1'

<a name="rule-image"></a>
#### image

검증에 포함된 필드는 이미지(jpeg, png, bmp, gif)여야 합니다.

<a name="rule-in"></a>
#### in:_foo_,_bar_,...

검증에 포함된 필드는 주어진 값의 리스트에 포함되어 있어야 합니다.

<a name="rule-integer"></a>
#### integer

검증에 포함된 필드는 정수 값이어야 합니다.

<a name="rule-ip"></a>
#### ip

검증에 포함된 필드는 IP 주소 형식이어야 합니다.

<a name="rule-max"></a>
#### max:_value_

검증에 포함된 필드는 최대 _value(값)_보다 작아야 합니다. 문자, 숫자, 파일은 `size` 규칙처럼 평가 됩니다.

<a name="rule-mimes"></a>
#### mimes:_foo_,_bar_,...

검증에 포함된 파일은 주어진 확장자 리스트에 중에 하나의 MIME 타입을 포함하고 있어야 합니다.

**MIME 규칙의 기본 사용법**

	'photo' => 'mimes:jpeg,bmp,png'

<a name="rule-min"></a>
#### min:_value_

검증에 포함된 필드는 최소 _value(값)_을 갖고있어야 합니다. 문자, 숫자, 파일은 `size` 규칙처럼 평가 됩니다.

<a name="rule-not-in"></a>
#### not_in:_foo_,_bar_,...

검증에 포함된 필드는 주어진 값들에 포함되어 있지 않아야 합니다.

<a name="rule-numeric"></a>
#### numeric

검증에 포함된 필드는 숫자여야 합니다.

<a name="rule-regex"></a>
#### regex:_pattern_

검증에 포함된 필드는 주어진 정규표현식과 일치해야 합니다.

**노트:** `regex` 패턴을 사용할 경우, 특히나 정규표현식이 파이프 문자를 포함하고 있다면, 파이프 문자를 사용하여 규칙들을 명시하지말고 배열을 이용해야 합니다.

<a name="rule-required"></a>
#### required

검증에 포함된 필드는 값이 존재해야 합니다.

<a name="rule-required-if"></a>
#### required_if:_field_,_value_

검증에 포함된 필드는 _field_와 _value_가 일치 할 경우 반드시 존재해야 합니다.

<a name="rule-required-with"></a>
#### required_with:_foo_,_bar_,...

검증에 포함된 필드는 주어진 다른 필드에 값이 _있을때 만_ 값이 있어야 합니다.

<a name="rule-required-without"></a>
#### required_without:_foo_,_bar_,...

검증에 포함된 필드는 주어진 다른 필드에 값이 _없을때 만_ 값이 있어야 합니다.

<a name="rule-same"></a>
#### same:_field_

검증에 포함된 필드는 주어진 _field(필드)_와 일치해야 합니다.

<a name="rule-size"></a>
#### size:_value_

검증에 포함된 필드는 주어진  _value(값)_과 크기가 일치해야 합니다. 문자열 데이터에서 _value_는 문자의 길이에 해당합니다. 숫자 데이터에서 _value_는 주어진 정수값에 해당합니다. 파일에서 _size_는 파일 크기(킬로바이트)에 해당합니다.

<a name="rule-unique"></a>
#### unique:_table_,_column_,_except_,_idColumn_

검증에 포함된 필드는 주어진 데이터베이스 테이블에서 고유한 값이어야 합니다. 만약 `column` 옵션이 명시되지 않았다면, 필드명이 대신 사용됩니다.

**Unique 규칙의 기본 사용법**

	'email' => 'unique:users'

**사용자 정의 컬럼명 명시**

	'email' => 'unique:users,email_address'

**Unique 규칙이 주어진 ID는 무시하도록 강요**

	'email' => 'unique:users,email_address,10'

**추가적인 Where절 추가**

또한 쿼리의 "where"절에 추가되는 조건을 명시할수도 있습니다.:

	'email' => 'unique:users,email_address,NULL,id,account_id,1

위의 규칙은, 오직 `account_id`가 `1`인 레코드만 유니크 체크에 포함됩니다.

<a name="rule-url"></a>
#### url

검증에 포함된 필드는 URL 형식으로 포맷되어야 합니다.

<a name="custom-error-messages"></a>
## 사용자 정의 에러 메시지

필요한 경우, 기본적으로 제공되는 에러 메시지 대신 사용자 정의 메시지를 사용할 수도 있습니다. 사용자 정의 메시지를 지정하는데는 여러가지 방법이 있습니다.

**검증기로 사용자 정의 메시지 전달**

	$messages = array(
		'required' => ':attribute 필드는 필수입니다.',
	);

	$validator = Validator::make($input, $rules, $messages);

*노트:* `:attribute` 플레이스 홀더는 검증에 있는 실제 필드명으로 교체됩니다. 검증 메시지에서 다른 플레이스 홀더를 사용할 수도 있습니다.

**다른 검증 플레이스 홀더**

	$messages = array(
		'same'    => 'The :attribute and :other must match.',
		'size'    => 'The :attribute must be exactly :size.',
		'between' => 'The :attribute must be between :min - :max.',
		'in'      => 'The :attribute must be one of the following types: :values',
	);

때때로 지정된 필드에 사용자 정의 에러 메시지를 지정할 수도 있습니다.:

**주어진 속성에 사용자 정의 메시지 지정**

	$messages = array(
		'email.required' => '당신의 e-mail 주소가 필요합니다!',
	);

사용자 정의 메시지를 `Validator` 인스턴스에 직접 전달하는 대신, 언어 파일 안에 사용자 정의 메시지를 명시할 수도 있습니다. 이렇게 하려면, `app/lang/xx/validation.php` 언어 파일의 `custom` 배열에 에러 메시지를 추가하면 됩니다.

**언어 파일에 사용자 정의 에러 메시지 명시**

	'custom' => array(
		'email' => array(
			'required' => '당신의 e-mail 주소가 필요합니다!',
		),
	),

<a name="custom-validation-rules"></a>
## 사용자 정의 검증 규칙

Laravel은 도움이 되는 다양한 검증 규칙들을 제공합니다. 하지만, 자신만의 규칙을 명시할 수도 있습니다. `Validator::extend` 메소드를 사용하여 사용자 정의 검증 규칙을 등록할 수 있습니다.:

**사용자 정의 검증 규칙 등록**

	Validator::extend('foo', function($attribute, $value, $parameters)
	{
		return $value == 'foo';
	});

> **노트:** `extend` 메소드에 전달되는 규칙의 이름은 반드시 "스네이크 케이스"여야 합니다..

사용자 정의 검증 클로저는 검증될 `$attribute`의 이름, 속성의 `$value`, 규칙에 전달되는 `$parameters` 배열 이렇게 3가지의 인수를 전달 받습니다.

`extend` 메소드에 클로저 대신 클래스와 메소드를 전달 할수도 있습니다.:

	Validator::extend('foo', 'FooValidator@validate');

또한 사용자 정의 규칙에 대한 에러 메시지도 정의해야 합니다. 인라인 사용자 정의 메시지 배열을 사용하거나 규칙 언어 파일에 항목을 추가하여 에러 메시지를 정의 하면 됩니다.

클로저 콜백을 사용하여 검증기를 확장하는 것 대신, 검증기 클래스 자체를 확장할 수도 있습니다. 이렇게 하려면 `Illuminate\Validation\Validator`를 확장하는 검증기 클래스를 만들어야 합니다. 메소드명 앞에 `validate`를 붙여 검증 메소드를 클래스에 추가할 수 있습니다.:

**검증기 클래스 확장**

	<?php

	class CustomValidator extends Illuminate\Validation\Validator {

		public function validateFoo($attribute, $value, $parameters)
		{
			return $value == 'foo';
		}

	}

다음, 확장된 사용자 정의 검증기 클래스를 등록해야 합니다.:

**사용자 정의 검증기 리졸버 등록**

	Validator::resolver(function($translator, $data, $rules, $messages)
	{
		return new CustomValidator($translator, $data, $rules, $messages);
	});

사용자 정의 검증 규칙을 만들 때, 에러 메시지에 쓸 사용자 정의 플레이스홀더를 정의해야 할때도 있습니다. 위에서 설명한 사용자 정의 검증기 클래스에 `relaceXXX` 메소드를 추가하여 플레이스홀더를 정의할 수 있습니다.

	protected function replaceFoo($message, $attribute, $rule, $parameters)
	{
		return str_replace(':foo', $parameters[0], $message);
	}
