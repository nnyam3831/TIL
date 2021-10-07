# Promise

Promise에서 선언하였을 때 실행되는 부분을 `executor`라고 한다. 이 `executor`는 선언과 동시에 실행되기 때문에(resolve가 나중에 실행되는 것)해당 부분에서 네트워크 요청을 하면 안된다.

Promise는 setTimeout 같은 것들과 비슷하게 이벤트루프에서 처리되는 비동기 객체이다. `microTaskQueue`에 담기는데 우선순위가 해당 큐는 우선순위가 가장 높게 처리된다. [참고](https://kkangdda.tistory.com/77)

- 콜백함수를 태스크 큐에 넣는 함수들

  - `setTimeout`, `setInterval`, `setImmediate`,`requestAnimationFrame`, `I/O`, `UI 렌더링`

- 콜백함수를 마이크로태스크 큐에 넣는 함수들
  - `process.nextTick`, `Promise`, `Object.observe`,`MutationObserver`

### 실행순서

맨날 async/await만 쓰다보니까 Promise랑 같이 생각하면 헷갈릴 때가 있어서 정리해봄

#### 예제 1

```ts
console.log('start')
setTimeout(()=>{
    console.log('timeout)
})
Promise.resolve('promise').then(console.log)
console.log('end)
```

이런 코드가 있다고 하자. 실행순서는?

- 한 줄씩 콜스택에 쌓이니깐 'start' 처음은 자명
- `setTimeout`의 콜백은 타이머 시간 돌고 `macrotask queue`으로 들어감
- 콜스택에 `promise.reesolve('promise')`위에 `console.log`순서로 쌓임
- `console.log`는 `microtask queue`에 쌓이고 `Promise!'는 resolve됨 (`promise.resolve`의 콜백, 콜스택 빔)
- 콜스택 비었으므로 마지막 'end'
- `microtask queue`에서 `promise` 콜백꺼내서 실행 'promise'
- `macrotask queue`에서 `setTimeout`콜백 꺼내서 실행 'timeout'

결과적으로, 순서 `start, end, promise, timeout`

#### 예제 2

`async/await` 쓸 때

```ts
Promise.resolve("hello")
async function hello() {
	return "hello"
}
// 두개 같은거
```

```ts
const one = () => Promise.resolve("one")

async function myFunc() {
	console.log("in function")
	const res = await one()
	console.log(res)
}

console.log("before function")
myFunc()
console.log("after function")
```

실행 순서는?

- before function
- myFunc()안에 in function 콘솔 출력
- `one` promise 실행하면서 `await`이후는 프로미스 콜백이 됨
- after function 출력
- `microtask queue`에서 res 콘솔 출력

결과적으로, 순서 `before function, in function, after function, one`

### catch의 동작방법

### Promise.all, Promise.racem, Promise.allSettled

**Promise.all**은 모든 프로미스가 끝났을 때 한 번에 return한다(순서 보장, but 병렬적으로 처리되기 때문에 끝나는 시간은 제각각)

```typescript
function delay(ms) {
	return new Promise((resolve) => setTimeout(resolve, ms))
}

async function getApple() {
	await delay(3000)
	return "apple"
}

async function getBanana() {
	await delay(2000)
	return "banana"
}

// 병렬처리
async function pickAllFruits() {
	const applePromise = getApple()
	const bananaPromise = getBanana()
	const apple = await applePromise
	const banana = await bananaPromise
	return `${apple} + ${banana}`
}

pickAllFruits().then(console.log) // apple + banana
```

위의 `pickAllFruits`는 `Promise.all`로 병렬처리 할 수 있다.

```typescript
async function pickAllFruitsWithPromiseAll() {
	const [apple, banana] = await Promise.all([getApple(), getBanana()])
	return `${apple} + ${banana}`
}
pickAllFruitsWithPromiseAll().then(console.log) // apple + banana
```

**Promise.race**는 Array에서 가장 먼저 끝나는 놈을 return

```typescript
async function pickFirstFruit() {
	return Promise.race([getApple(), getBanana()])
}

pickFirstFruit().then(console.log) // banana
```

**Promise.allSettled**는 에러가 발생하더라도 다음 모든 프로미스를 병력적으로 실행 후 결과를 리턴한다. `Promise.all`은 에러가 발생하면 이후 프로미스를 무시

### 순차적으로 Promise를 실행하는 법
