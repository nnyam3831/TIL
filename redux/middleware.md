### 리덕스 미들웨어

리덕스는 액션 -> 미들웨어 -> 리듀서 -> 스토어 순으로 동작한다. 리덕스 미들웨어는 클로저를 사용해서 다음과 같이 구현한다.

```typescript
const middleware = (store) => (next) => (action) => {
  next(action);
  //...
};
```

`redux-thunk`를 보면 다음과 같이 구현되어있다.

```typescript
const thunk = (store) => (next) => (action) =>
  typeof action === "function"
    ? action(store.dispatch, store.getState)
    : next(action);
```

`thunk`를 다음과 같이 사용하는데

```typescript
const getSomething = () => (dispatch, getState) => {
  // ...
};
```

단순하게, 액션이 함수면 각각 `dispatc`h와 `getState`를 부여한다는 아주 간단한 원리이다.

### applyMiddleware

리덕스에서 middleware를 사용하려면 다음과 같이 설정한다.

```typescript
const store = createStore(reducer, applyMiddlewares(middleware1, middleware2));
```

액션이 발생하면 각각 미들웨어를 거치고 reducer에 도착하게 된다. `applyMiddleware`는 아래와 비슷하게 구현되어있다.

```typescript
const applyMiddleware = (...middlewares)=> createStore => (...args)=>{
  const store = createStore(...args);
  const funcWithStore = middlewares.map(middleware => middleware(store));

  const chainedFunc = funcsWithStore.reduce((acc,cur)=>next=>acc(cur(next)));

  return {
    ...store,
    dispatch:chaindeFunc(store.dispatch);
  };
}
```

`funcWithStore`: 각각의 `middleware`들에게 `store`를 부여한 클로저
`chainedFunc`: `reduce`로 chaining을 건다. 미들웨어가 `m1`, `m2`, `m3`라면 `m1(m2(m3(next)))`처럼 될것이다. 마지막 미들웨어에서 리듀서에게 넘기며 미들웨어 체이닝이 마무리된다.
