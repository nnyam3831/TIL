### OR 연산

가장 첫 번째 마주친 값

가장 level이 높은 카테고리 Id값을 할당하고 싶다.

```javascript
const categoryId = categoryId3 | categoryId2 | categoryId1;
```

### And 연산

값들이 다 유효하다면 마지막으로 확인한 truthy 값

```javascript
const result1 = "Apple" && "Melon"; // Melon
const result2 = undefined && "Red"; // undefined
```

### 객체를 생성하는 방법

1. 생성자

   ```javascript
   function Plus(a1, a2) {
     this.a1 = a1;
     this.a2 = a2;
     this.result = () => {
       return this.a1 + this.a2;
     };
   }
   const p = new Plus(1, 2);
   ```

   생성자 형식은 prototype을 정의 가능

2. 객체 리터럴

   ```javascript
   const Person = {
     name: "seongwon",
     sayHello: function () {
       console.log(`Hello, my name is ${name}`);
     },
   };
   ```

   객체 리터럴은 prototype이 없다

### Arguments

Javascript에서는 arguments라는 이름으로 인자들을 참조할 수 있다.

```javascript
function test() {
  console.log(arguments);
}
test(1, 2, 3); // [Arguments] { '0': 1, '1': 2, '2': 3 }
test("a", "b", 3); // [Arguments] { '0': 'a', '1': 'b', '2': 3 }
```

### 1급 객체

변수에 담을 수 있다.

```javascript
var bar = function () {
  return "javscript";
};
console.log(bar()); // javascript
```

파라미터로 전달할 수 있다.

```javascript
var test = function (func) {
  func(); // 파라미터로 받은 함수 호출
};

// test() 함수에 다른 함수를 파라미터로 넣어 호출
test(function () {
  console.log("javascript");
});
```

리턴 값으로 사용할 수 있다.

```javascript
function test() {
  return function () {
    console.log("javscript");
  };
}

var bar = test();
bar();
```

자바스크립트에서 함수가 1급객체이기 때문에,

1. `콜백 패턴 을 사용할 수 있다.`
1. `고차함수(High-order function) 를 만들 수 있다.`
1. `Javascript의 클로저(closure) 를 사용해 커링(currying)과 메모이제이션(memoization) 이 가능하다.`
