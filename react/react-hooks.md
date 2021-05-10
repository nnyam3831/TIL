# 클로저를 활용해서 `react-hook`을 바닥부터 만들어보자

### useState

```typescript
function useState(initialValue) {
  let _value = initialValue;
  const state = () => _value;
  const setState = (newValue) => {
    _value = newValue;
  };

  return [state, setState];
}

const [count, setCount] = useState(0);
console.log(count());
setCount(5);
console.log(count());
```

이렇게하면 일단은 동작한다. 지금은 getter로 state를 가져올 수 밖에 없는데, 실제로 state는 변수이므로 수정이 필요하다. 이는 클로저를 또 다른 클로저 내부로 이동해서 변수를 최상위 함수인 React안으로 잡아놓음으로써 해결할 수 있다. 가상의 리액트 환경을 만들고 시나리오를 테스트해보자.

```typescript
const React = (function () {
  let _val; // 여기다 val을 잡아놓음으로써 state의 스코프 체이닝이 일어나 계속해서 참조할 수 있음
  function useState(initVal) {
    const state = _val || initVal;
    const setState = (newVal) => {
      _val = newVal;
    };
    return [state, setState];
  }
  function render(Component) {
    const C = Component();
    C.render();
    return C;
  }
  return { useState, render };
})();

function Component() {
  const [count, setCount] = React.useState(1);
  return {
    render: () => console.log(count),
    click: () => setCount(count + 1),
  };
}
let App;
App = React.render(Component); // 1
App.click();
App = React.render(Component); // 2
App.click();
App = React.render(Component); // 3
App.click();
App = React.render(Component); // 4
```

여기서 문제는 `useState`를 한 번 더 사용하면 발생하는데, 지금 state는 React 함수 안에서 `_val`에 의존하고있으므로 각 훅의 state를 배열에 담아야한다.

```typescript
const React = (function () {
  let hooks = [];
  let idx = 0;
  function useState(initValue) {
    const state = hooks[idx] || initValue;
    const _idx = idx;
    const setState = (newVal) => {
      hooks[_idx] = newVal;
    };
    idx++;
    return [state, setState];
  }

  function render(Component) {
    idx = 0; // 렌더링시 초기화, 순서대로 인덱스 부여받음
    const C = Component();
    C.render();
    return C;
  }

  return { useState, render };
})();

function Component() {
  const [count, setCount] = React.useState(1);
  const [text, setText] = React.useState("apple");
  return {
    render: () => console.log({ count, text }),
    click: () => setCount(count + 1),
    type: (word) => setText(word),
  };
}
let App;
App = React.render(Component); // {count: 1, text: 'apple'}
App.click();
App = React.render(Component); // {count: 2, text: 2}
App.type("banana");
App = React.render(Component); // {count: 'banana', text: 'banana'}
```

결국 변수를 남겨놓고(`클로저`) 불러와서 사용하는 방식을 훅으로 정의했을 뿐이다,
여기서 `hook`의 규칙 중 하나인 **최상위에서만 훅을 호출하기**와 **조건부로 훅을 호출하지 않기**를 이해할 수 있다. 조건부로 훅을 호출하게 된다면 인덱스의 순서를 보장할 수 없기 때문이다.

### useEffect

```typescript
const React = (function () {
  let hooks = [];
  let idx = 0;
  function useState(initValue) {
    const state = hooks[idx] || initValue;
    const _idx = idx;
    const setState = (newVal) => {
      hooks[_idx] = newVal;
    };
    idx++;
    return [state, setState];
  }

  function useEffect(callback, depsArray) {
    const oldDeps = hooks[idx];
    let hasChanged = true;
    if (oldDeps) {
      hasChanged = depsArray.some((dep, i) => !Object.is(dep, oldDeps[i]));
    }

    if (hasChanged) {
      callback();
    }

    hooks[idx] = depsArray;
    idx++;
  }

  function render(Component) {
    idx = 0;
    const C = Component();
    C.render();
    return C;
  }

  return { useState, useEffect, render };
})();

function Component() {
  const [count, setCount] = React.useState(1);
  const [text, setText] = React.useState("apple");
  React.useEffect(() => {
    console.log("effect!", count, text);
  }, [count, text]);

  return {
    render: () => console.log({ count, text }),
    click: () => setCount(count + 1),
    type: (word) => setText(word),
    noop: () => setCount(count),
  };
}
let App;
App = React.render(Component);
// 이펙트 0 foo
// render {count: 0, text: 'foo'}
App.click();
App = React.render(Component);
// 이펙트 1 foo
// render {count: 1, text: 'foo'}
App.type("bar");
App = React.render(Component);
// 이펙트 1 bar
// render {count: 1, text: 'bar'}
App.noop();
App = React.render(Component);
// // 이펙트가 실행되지 않음
// render {count: 1, text: 'bar'}
App.click();
App = React.render(Component);
// 이펙트 2 bar
// render {count: 2, text: 'bar'}
```
