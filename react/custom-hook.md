## 커스텀 훅에 관한 고찰

2020 FE Conf를 보다가 당근마켓의 원지혁님의 발표에서 커스텀 훅에 대한 새로운 관점을 얻게 되어 정리한다.

#### 선언적으로 작성한다는 것

먼저, 선언적으로 작성한다는 것은 How가 아니라 What을 명시한다는 것을 의미한다. 계속해서 FP가 관심을 받고 있는것도 명령형 프로그래밍이 아닌 선언형 프로그래밍의 장점을 위해서임을 잊어서는 안된다.

에를 들어 함수형 프로그래밍에서는 다음과 같이 선언적으로 iterable을 조작할 수 있다.

```typescript
cosnt result = [1,2,3,4,5]
                    .filter(elem => elem % 2)
                    .map(elem => elem * 3)
// [3, 9, 15]
```

이렇듯 내가 이 배열을 갖고 **_무엇_** 을 할까가 강조되는게 바로 **_선언적_** 작성하는 것이다.

그렇다면 react custom hook 또한 리액트를 좀 더 선언적으로, FP로 작성하기 위해 고안된 방법이라면 이를 좀 더 선언적으로 코드를 작성할 필요가 있다. 나 또한 습관적으로 커스텀 훅을 그저 로직을 분리하기 위해 사용했었는데, 로직을 분리하는데 있어 더 선언적으로 더 나은 코드를 작성하는 방법은 아래와 같은 방법이 될 수 있을 것 같다.

### 일정 스크롤 이상일때 특정 상태를 true로 바꾸기

```typescript
import React, { useEffect, useState } from "react"

const ExampleComponent: React.FC = () => {
	const [scrolled, setScrolled] = useState(false)

	useEffect(() => {
		const onScroll = () => {
			if (window.scrollY > 500) {
				setScroll(true)
			} else {
				setScroll(false)
			}
		}
		window.addEventListener("scroll", onScroll)

		return () => {
			window.removeEventListener("scroll", onScroll)
		}
	}, [setScrolled])

	return <div>...</div>
}
```

이러한 리액트 컴포넌트가 있다고 하자, 일반적으로 커스텀 훅으로 분리를 한다면 다음과 같이 분리할 것이다.

```typescript
const ExampleComponent: React.FC = () => {
	const scrolled = useScrolled()

	return <div>...</div>
}

function useScrolled() {
	const [scrolled, setScrolled] = useState(false)

	useEffect(() => {
		const onScroll = () => {
			if (window.scrollY > 500) {
				setScroll(true)
			} else {
				setScroll(false)
			}
		}
		window.addEventListener("scroll", onScroll)

		return () => {
			window.removeEventListener("scroll", onScroll)
		}
	}, [setScrolled])

	return scrolled
}
```

위와 같이 커스텀훅을 사용함으로써 컴포넌트와 로직을 별개로 분리할 수 있고 대부분 이러한 방식으로 리팩토링을 진행한다.(나도 그렇다)
하지만 위를 **_선언적_** 으로 작성하려면 어떻게 해야할까?
선언적으로 작성하는 것에 시작은 관심사의 분리, 즉 **_How_**와 **_What_**을 분리해야 한다.

위의 코드에서 useEffect를 사용하는것, 이벤트리스너를 등록하는게 **_어떻게_** 동작하는지를 기술하는 것이라고 볼 수 있고, scrolled 상태와 scroll 이벤트 리스너에서 실행될 함수가 바로 **_무엇_** 에 대하여 기술하는 것이라고 볼 수 있다.

이렇게 **_How_**와 **_What_**을 분리하면 아래와 같이 리팩토링 될 수 있다.

```typescript
// How
function useWindowScrollEffect(
	listener: (scrollY: number) => void,
	deps?: any[]
) {
	useEffect(() => {
		const onScroll = () => {
			listener(window.scrollY)
		}
		window.addEventListener("scroll", onScroll)

		return () => {
			window.removeEventListener("scroll", onScroll)
		}
	}, [deps])
}

// What
const ExampleComponent: React.FC = () => {
	const [scrolled, setScrolled] = useState(false)

	useWindowScrollEffect(
		(scrollY) => {
			if (scrollY > 500) {
				setScroll(true)
			} else {
				setScroll(false)
			}
		},
		[setScrolled]
	)

	return <div>...</div>
}
```
