### 클로저

나도 그랬지만 아마도 자바스크립트를 공부할 때 가장 와닿지 않는 개념 중 하나일듯하다. 이게 와닿지 않는 이유는 도대체 이 `클로저`가 왜 쓰이는지를 몰라서이다. 무엇을 공부하든 개념 자체가 아니라 `이걸 어떻게 사용하는가`와 `왜 사용하는가`를 알면 정확한 이해에 도움이 된다.

> 클로저는 **`inner function`이 자신이 선언되었을때의 `Lexical environment`를 기억하여 자신이 호출되었을때 그 환경에 접근할 수 있는 기술**을 의미한다.

우선 **클로저**는 자바스크립트 고유 개념이 아니라, 함수형 프로그래밍에서 사용되는 개념이다. 즉, 자바스크립트에서 함수형 프로그래밍을 위해서 클로저가 사용된다는 의미이다. 클로저를 사용하려는 발상은 함수에서 무언가를 `메모이제이션`한다는 발상에서 시작된다.클로저를 사용해서 내부함수에서 외부함수의 `value`를 `메모이제이션`할 수 있으며, 이러한 함수들은 각각의 속성을 갖는 함수로 변하게 된다. **즉, 클로저를 사용하면 `함수`를 마치 `객체`처럼 사용할 수 있다.**

흔하디 흔한 타이머 예제들부터 시작해보자

```typescript
for (var i = 0; i < 5; i++) {
  setTimeout(function () {
    console.log(i);
  }, i * 1000);
}
```

당연히 결과는 5가 다섯번 출력될 것이다. 콜백으로 불릴때 i는 전역변수 i를 참조하기 때문이다.
클로저를 사용하면, 원하는 대로 동작하게 할 수 있다.

```typescript
for (var i = 0; i < 5; i++) {
  (function (j) {
    setTimeout(function () {
      console.log(j);
    }, j * 1000);
  })(i);
}
// 0, 1, 2, 3, 4
```

포인트는 변수 `i`를 인자로 넘겨서 `inner function`에서 참조할 수 있게 복사하는 것이다.
이게 바로 `lexical environment`에 접근할 수 있다는 의미이다.

사실 이 예제는 와닿지가 않는다. `const`, `let`이 나오기 전에라면 쓰일 수 있겠지만 다음과 같이 `let`을 사용하면 변수가 바로 바인딩되어 올바른 결과를 출력할 수 있다.

```typescript
for (let i = 0; i < 5; i++) {
  setTimeout(function () {
    console.log(i);
  }, i * 1000);
}
```

다음으로 클로저를 사용하면 `private` 변수를 만들 수 있다.

```typescript
var outer = function () {
  var a = 1;
  var inner = function () {
    return ++a;
  };
  return inner;
};
var outer2 = outer();
```

이제 변수 `a`는 직접 접근할 수 없고 오로지 `outer` 함수를 통해서 접근이 가능하다.

이 예제도 딱히 와닿지는 않는다. 굳이 이렇게 하지 않아도 지금은 `Class`가 나와서 `private` 속성을 선언할 수 있기 때문이다.

> **그러면 클로저는 예전 자바스크립트를 위한 유산인가?**

로 내가 잘못생각했었는데, 그렇지 않다. 클로저는 함수형 프로그래밍을 위한 필수 요소이다. 알게 모르게 내가 사용하고 있던 거의 모든 곳에서 클로저가 사용되고 있음을 알 수 있었다.

지금부터 제대로 와닿는 예제를 보자

### 커링

커링은 함수안에 함수를 리턴하는 함수로, n개의 인자를 받는 함수 대신 하나씩 인자를 받는 n개의 함수로 만들 수 있다. 커링을 사용하면 여러가지 기능을 갖는 함수를 만들 수 있고, 처음에 언급했던 것 처럼 `함수를 객체처럼 사용`할 수 있다.

```typescript
function multiply(x) {
  return function (y) {
    return function (z) {
      return x * y * z;
    };
  };
}
const multiply3 = multiply(3);
const multiply3And5 = multiply3(5);
const multiply2And4 = multiply(2)(4);

console.log(multiply3And5(7)); // 105
console.log(multiply2And4(6)); // 48
```

이렇게 한 번의 함수 정의로 각기다른 기능을 하는 함수를 만들어낼수있다. 공식 예제처럼 다음과 같이 사용할 수 있다.

```html
<a href="#" id="size-12">12</a>
<a href="#" id="size-14">14</a>
<a href="#" id="size-16">16</a>
```

```typescript
function makeSizer(size) {
  return function () {
    document.body.style.fontSize = size + "px";
  };
}

var size12 = makeSizer(12);
var size14 = makeSizer(14);
var size16 = makeSizer(16);

document.getElementById("size-12").onclick = size12;
document.getElementById("size-14").onclick = size14;
document.getElementById("size-16").onclick = size16;
```

### 리덕스 미들웨어

당연하게 사용했던 리덕스 미들웨어도 클로저로 구현되어있다. 리덕스 미들웨어를 이용해서 `store`, `next`를 외부에서 접근할 수 없고 오로지 `action`으로만 제어할 수 있게 한다.

```typescript
const middleware = (store) => (next) => (action) => {
  next(action);
  //...
};
```

실제로, `redux-thunk`는 다음과 같이 구현되어있다.

```typescript
const thunk = (store) => (next) => (action) =>
  typeof action === "function"
    ? action(store.dispatch, store.getState)
    : next(action);
```

[리덕스 미들웨어 동작 원리](../redux/middleware.md)

### React Hooks

실제로 클로저를 사용해서 `React Hooks`를 구현할 수 있다. 그렇게 밥먹듯이 썼던 `hook`들이 결국 클로저의 원리로 이루어져있다. `react hooks`도 결국 리액트의 **함수형 프로그래밍**을 위해 개발되었다는것을 명심하자

- 리액트 hooks를 클로저로 만들어보기(https://rinae.dev/posts/getting-closure-on-react-hooks-summary)

### 메모리 관리

클로저를 이용하면 `lexical scope`의 변수에 접근해서 사용할 수 있다. 그럼 결국 그 `자유변수`들은 메모리 어딘가에 남아있다는 건데, 왜 얘네들은 없어지지 않을까? 그건 바로 **어떤 값을 참조하는 변수가 하나라도 있으면 수집 대상에 포함시키지 않는** 가비지 컬렉터의 동작방식 때문이다. 때문에 생각없이 클로저를 사용하면 `메모리 누수`가 생길 수 있으므로 사용하지 않는 클로저에 대해서 메모리 관리가 필요하다.
발상의 출발은 메모이제이션

1.  `return`에 의한 클로저의 메모리 해제

    ```typescript
    const outer = (function () {
      let a = 1;
      const inner = function () {
        return ++a;
      };
      return inner;
    })();

    console.log(outer());
    console.log(outer());
    outer = null; // outer 식별자의 inner 함수 참조를 끊음
    ```

1.  setInterval에 의한 클로저의 메모리 해제

    ```typescript
    (function () {
      let a = 0;
      let intervalId = null;
      const inner = function () {
        if (++a >= 10) {
          clearInterval(intervalId);
          inner = null; // inner 식별자의 함수 참조를 끊음
        }
        console.log(a);
      };
      intervalId = setInterval(inner, 1000);
    })();
    ```

1.  eventListener에 의한 클로저의 메모리 해제

    ```typescript
    (function () {
      let count = 0;
      const button = document.createElement("button");
      button.innerText = "click";

      let clickHandler = function () {
        console.log(++count, "times clicked");
        if (count >= 10) {
          button.removeEventListener("click", clickHandler);
          clickHandler = null; // clickHandler 식별자의 함수 참조를 끊음
        }
      };

      button.addEventListener("click", clickHandler);
      document.body.appendChild(button);
    })();
    ```
