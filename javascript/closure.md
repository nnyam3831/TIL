### 클로저

나도 그랬지만 아마도 자바스크립트를 공부할 때 가장 와닿지 않는 개념 중 하나이다. 무엇을 공부하든 개념 자체가 아니라 `이걸 어떻게 사용하는가`와 `왜 사용하는가`를 알면 정확히 이해할 수 있다.

우선 **클로저**는 자바스크립트 고유 개념이 아니라, 함수형 프로그래밍에서 사용되는 개념이다. 즉, 자바스크립트에서 함수형 프로그래밍을 위해서 클로저가 사용된다는 말이다. 요는 `함수`를 마치 `객체`처럼 사용하기 위한 개념이다.

발상의 출발은 메모이제이션

- 리덕스의 미들웨어
- 커링
- 프라이빗 변수
- 다양한 커스터마이징
- 렉시컬 스코프
- 가비지 컬렉터의 동작방식(어떤 값을 참조하는 변수가 하나라도 있으면 수집 대상에 포함시키지 않음)

꼭 리턴하는 경우에만 외부로 전달하는것은 아니다. 예를 들면 외부객체인 Window에서 하도록(이벤트 루프로 넘기는 경우)해도 참조

```typescript
(function () {
  let a = 0;
  let intervalId = null;
  const inner = function () {
    if (++a >= 10) {
      clearInterval(intervalId);
    }
    console.log(a);
  };
  intervalId = setInterval(inner, 1000);
})();
```

```typescript
// (1) return에 의한 클로저의 메모리 해제

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
// (2) setInterval에 의한 클로저의 메모리 해제

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
// (3) eventListener에 의한 클로저의 메모리 해제

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

- 리액트 hooks를 클로저로 만들어보기(https://rinae.dev/posts/getting-closure-on-react-hooks-summary)
