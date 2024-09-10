# strict mode

Week: 4주차, X
Chapter: 20

## 20.1 strict mode란?

```jsx
function foo() {
	x = 10;
}
foo();

console.log(x); // ?
```

foo 함수의 스코프에서 x 변수의 선언 검색

→ foo 함수의 스코프에는  x 변수의 선언이 없으므로 검색에 실패

→ x 변수를 검색하기 위해 foo 함수 컨텍스트의 상위 스코프에서 x 변수의 선언 검색

→ 전역 스코프에도 x 변수의 선언이 존재하지 않음

이때 암묵적으로 전역 객체에 x 프로퍼티를 동적으로 생성함

따라서 전역 객체의 x 프로퍼티는 전역 변수처럼 사용이 가능함

→ 암묵적 전역(implicit global)

그러나 개발자의 의도와 상관없이 발생하기 때문에 오류를 발생시키는 원인이 될 수 있음

따라서 반드시 var, let, const 키워드를 사용하여 변수를 선언한 뒤 변수를 사용해야 함

근데 실수가 발생할 수 있으므로 잠재적인 오류를 발생시키기 어려운 개발 환경을 만들고, 그 환경에서 개발하는 것이 좀 더 근본적인 해결책임

→ 이를 지원하기 위해 ES5부터 strict mode가 추가됨

ESLint 같은 도구를 사용해서 strict mode와 유사한 효과를 얻을 수 있음.

## 20.2 strict mode의 적용

전역의 선두 또는 함수 몸체의 선두에 ‘use strict’;를 추가하면 됨

코드의 선두에 작성하지 않으면 strict mode가 제대로 동작하지 않음

```jsx
'use strict';

function foo() {
	x = 10; // ReferenceError: x is not defined
}
foo();
```

함수 몸체의 선두에 추가하면 해당 함수와 중첩 함수에 strict mode가 적용됨

## 20.3 전역에 strict mode를 적용하는 것은 피하자

```jsx
<!DOCTYPE html>
<html>
<body>
	<script>
		'use strict';
	</script>
	<script>
		x = 1;
		console.log(x); // 1
	</script>
	<script>
		'use strict';
		y = 1;
		console.log(y); // ReferenceError: y is not defined
	</script>
</body>
</html>
```

전역에 적용한 strict mode는 스크립트 단위로 적용됨

그러나 strict mode가 적용된 스크립트와 적용되지 않은 스크립트를 혼용하는 것은 오류를 발생시킬 수 있음

특히 외부 서드파티 라이브러리를 사용하는 경우, 라이브러리가 non-strict mode인 경우도 있기 때문에 전역에 strict mode를 적용하는 것은 바람직하지 않음

## 20.4 함수 단위로 strict mode를 적용하는 것도 피하자

어떤 함수는 strict mode를 적용하고, 어떤 함수는 strict mode를 적용하지 않는 것은 바람직하지 않음

또, 모든 함수에 strict mode를 적용하는 건 번거로움

```jsx
(function () {
	// non-strict mode
	var let = 10; // 에러 발생 x
	function foo() {
		'use strict';
		let = 20; // SyntaxError: Unexpected strict mode reserved word
	}
	foo();
}());
```

strict mode가 적용된 함수가 참조할 함수 외부의 컨텍스트에 strict mode를 적용하지 않으면 이 또한 문제가 될 수 있음

따라서 strict mode는 즉시 실행함수로 감ㅅ싼 스크립트 단위로 적용하는 것이 바람직함

## 20.5 strict mode가 발생시키는 에러

### 암묵적 전역

선언하지 않은 변수를 참조하면 ReferenceError 발생

```jsx
(function () {
	'use strict';
	
	x = 1;
	console.log(x); // ReferenceError: x is not defined
}());
```

### 변수, 함수, 매개변수의 삭제

delete 연산자로 변수, 함수, 매개변수를 삭제하면 SyntaxError 발생

```jsx
(function () {
	'use strict';
	
	var x = 1;
	delete x; // SyntaxError: Delete of an unqualified identifier in strict mode.
	
	function foo(a) {
		delete a; // SyntaxError: Delete of an unqualified identifier in strict mode.
	}
	delete foo; // SyntaxError: Delete of an unqualified identifier in strict mode.
}());
```

### 매개변수 이름의 중복

중복된 매개변수 이름을 사용하면 SyntaxError 발생

```jsx
(function () {
	'use strict';
	
	//SyntaxError: Duplicate parameter name not allowed in this context
	function foo(x, x) {
		return x + x;
	}
	console.log(foo(1, 2));
}());
```

### with문 사용

with문을 사용하면 SyntaxError 발생

```jsx
(function () {
	'use strict';
	
	//SyntaxError: Strict mode code may not include a with statement
	with({ x: 1 }) {
		console.log(x);
	}
}());
```

## 20.6 strict mode 적용에 의한 변화

### 일반 함수의 this

strict mode에서 함수를 일반 함수로서 호출하면 this에 undefined가 바인딩 됨

생성자 함수가 아닌 일반 함수 내부에서는 this를 사용할 필요가 없기 때문.

에러는 발생하지 않음

```jsx
(function () {
	'use strict';
	
	function foo() {
		console.log(this); // undefined
	}
	foo();
	
	function Foo() {
		console.log(this); // Foo
	}
	new Foo();
}());
```

### arguments 객체

strict mode에서는 매개변수에 전달된 인수를 재할당하여 변경해도 arguments 객체에 반영되지 않음

```jsx
(function (a) {
	'use strict';
	
	// 매개변수에 전달된 인수를 재할당하여 변경
	a = 2;
	
	// 변경된 인수가 arguments 객체에 반영되지 않음
	console.log(arguments); // { 0: 1, length: 1 }
}(1));
```