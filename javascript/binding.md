### This

자바스크립트에서는 `this`의 사용을 굉장히 조심해야한다. `this`를 사용하면서 가장 중요한 사실은 `this`는 실행될 때 런타임에서 동적으로 `binding`된다는 것이며 `어떻게 호출되는지가 중요하다는 것`이다. 즉, `this`는 **현재 실행되는 코드의 실행 컨텍스트**를 의미한다.
`this`는 브라우저에서 기본적으로 window, Node에서 global 객체이다. 만약 strict 모드를 적용한다면 undefined으로 정의된다.
`this`는 함수 내부에서는 전역객체를 가리키며, 객체의 메소드에서는 해당 객체를 가리킨다.

```typescript
const obj = {
  name: "zz",
  something: {
    name: "kk",
    hello: function () {
      console.log(this.name); // kk
    },
  },
};
```

### Binding

앞서 말했던 동적으로 `binding`된다는 개념을 확인해보자

#### 암시적 바인딩

함수 호출시 객체의 프로퍼티로 접근할 때 실행하는 바인딩

```typescript
function hello() {
  console.log(this.name);
}
var obj = {
  name: "chris",
  hello: hello,
};

obj.hello(); // chris
```

객체 프로퍼티에 할당했더라도 다음과 같이 할당하면 다르게 동작한다.

```typescript
function hello() {
  console.log(this.name);
}
var obj = {
  name: "chris",
  hello: hello,
};

var helloFn = obj.hello;
name = "global text!";
helloFn(); // global text
```

이렇게 변수에 할당하게 될 경우, 객체의 속성으로 참조하지 않기 때문에 `global`의 `name`인 `global text`로 접근하게 된다.

때문에 콜백을 사용할 때 다음과 같은 실수를 하게 될 수 있다.

```typescript
function hello() {
  console.log(this.name);
}

var obj = {
  name: "chris",
  hello: hello,
};

setTimeout(obj.hello, 1000); // undefined or global context
name = "global context!";
```

#### 명시적 바인딩

이렇게 동적으로 바인딩되는 특성때문에 명시적으로 바인딩할수있는 `call`, `apply`, `bind`와 같은 `api`를 제공해준다. `call`과 `apply`는 args만 다를뿐 컨텍스트 객체를 명시한다는 점에서 같은 기능을하는 함수이다.

`call` 함수로 컨텍스트를 명시할 수 있다.

```typescript
function hello() {
  console.log(this.name);
}

var obj = {
  name: "chris",
};

name = "global context!";
hello.call(obj); // "chris"
```

#### 하드 바인딩

마찬가지로 `bind`함수를 통해 컨텍스트를 바인딩 할 수 있는데, 하드 바인딩은 명시적 바인딩보다 더 높은 우선순위를 갖는다.

```typescript
var myMethod = function () {
  console.log(this);
};

var obj1 = {
  a: 2,
};

var obj2 = {
  a: 3,
};

myMethod = myMethod.bind(obj1); // 2
myMethod.call(obj2); // 2
```

`bind`함수를 사용해서 위의 setTimeOut 예제를 올바르게 동작하게 할 수 있다.

```typescript
function hello() {
  console.log(this.name);
}

var obj = {
  name: "chris",
  hello: hello,
};

setTimeout(obj.hello.bind(obj), 1000); // chris
name = "global context!";
```

### New 바인딩

`New 바인딩`은 그저 새로운 객체를 반환한다. 그래서 다른 명시적 바인딩으로 함수를 다른 객체에 바인딩하더라도 `New`로 생성한 객체에 바인딩된다.

```typescript
function hello(name) {
  this.name = name;
}

var obj1 = {};

var helloFn = hello.bind(obj1);
helloFn("chris");
console.log(obj1.name); // chris

var obj2 = new helloFn("alice");
console.log(obj1.name); // chris
console.log(obj2.name); // alice
```

정리하자면,

1. new로 함수를 호출했는가? 그럼 실행결과 반환되는 값이 this다.

   ```typescript
   var obj = new hello(); // this === obj
   ```

2. call, apply, bind로 함수를 호출했는가? 그럼 인자로 넘겨준 객체가 this다.

   ```typescript
   var obj = {};
   hello.call(obj); // this === obj
   hello.apply(obj); // this === obj
   hello.bind(obj)(); // this === obj
   ```

3. 객체 프로퍼티로 접근하여 함수를 실행했는가? 그럼 이 객체가 this다.

   ```typescript
   obj.hello(); // this === obj
   ```

4. 이외의 경우는 this는 전역 객체다.

결과적으로 우선순위는 다음과 같다.

**`New 바인딩 > 명시적 바인딩 > 암시적 바인딩`**

### Arrow function

다 정리한 줄 알았는데, 복병이 등장한다. 바로 `화살표 함수`이다. 위에서 언급했듯이 컨텍스트 바인딩은 `동적으로 바인딩`된다. 하지만 `화살표 함수`에서는 함수가 정의되는 **코드상 상위 블록의 컨텍스트를 this로 바인딩**하며 이를 **Lexical this**라 한다.

`화살표 함수`의 특징들은 다음과 같다.

- `화살표 함수`는 `prototype` 속성이 없으며, `new`로 생성될 수 없다.
- 자신만의 `this`가 없으며 무조건 상위 스코프에 따라 결정된다.
- `arguments`를 사용할 수 없다. 따라서 다음과 같이 사용해야 한다.

  ```typescript
  function func1() {
    console.log(arguments[0]);
  }

  const func2 = (...args) => {
    console.log(args[0]);ㄴ
  ```

- `method`로 사용하지 않는다. `method`로 사용하게 되면 해당 객체를 가리키지않고 전역객체를 가리키기 때문이다.

  ```typescript
  const obj1 = {
    name: "Lee",
    sayName: () => {
      console.log(`hi + ${this.name}`);
    },
  };
  obj1.sayName();
  ```

- 만약 `addEventListener`에서 해당 `element`를 접근하고 싶다면, `화살표 함수`를 사용해서는 안된다.
  ```typescript
  var button = document.getElementById("myButton");
  button.addEventListener("click", function () {
    console.log(this === button); // => true
    this.innerHTML = "Clicked button";
  });
  ```

#### 리액트 `Class 컴포넌트`에서 화살표 함수를 사용했던 이유

지금은 많이 사용하고 있지 않지만 리액트에서 `Class 컴포넌트`안에 `bind`를 굳이 사용해서 함수를 컴포넌트와 연결하는 이유를 이제 알 수 있다. 예를 들어 다음과 같이 바인딩을 해주었는데,

```typescript
this.handleClick.bind(this);
```

이는 이 `handleClick`함수가 콜백으로 동작하기 때문이다. 리액트 컴포넌트의 `onClick` 이벤트에 콜백으로 등록을 해놓으면, 클릭했을때 `handleClick`함수가 실행되는 컨텍스트는 `window`이므로 `window`에서는 `handleClick` 함수를 찾을 수 없기 때문에 하드바인딩을 통해서 해당 컴포넌트와 연결시켜주는 것이다.
하지만 앞서 이야기했듯 `화살표 함수`는 자신이 선언되었을때의 상위 스코프를 가리키므로, 다음과 같이 선언하게 되면 바인딩 필요없이 컴포넌트의 `handleClick`에 접근할 수 있었던 것이다.

```typescript
const handleClick = () => {...}
```
