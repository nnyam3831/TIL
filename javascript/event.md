### 이벤트 버블링

이벤트가 발생하면 자바스크립트에서는 자식 -> 부모로 이벤트가 전파된다.(`default: true`)

### 이벤트 캡쳐

반대로 부모 -> 자식으로 이벤트가 전달되는걸 말한다. 이건 `default: false`인데, `addEventListener`에서 설정할 수 있다.

```typescript
target.addEventListener("click", function () {}, true);
```

### 이벤트 위임

어떤 리스트가 있고, 그 리스트를 클릭했을때 처리하는 방법은 매우 일반적으로 볼 수 있는 UI이다.
다음과 같은 html이 있다고 하자

```html
<ul id="todo-app">
  <li class="item">Walk the dog</li>
  <li class="item">Pay bills</li>
  <li class="item">Make dinner</li>
  <li class="item">Code for one hour</li>
</ul>
```

일반적으로 다음과 같이 클릭이벤트를 구현한다.

```javascript
document.addEventListener("DOMContentLoaded", function () {
  const app = document.getElementById("todo-app");
  const items = app.getElementsByClassName("item");

  for (let item of items) {
    item.addEventListener("click", function () {
      // onClick function
    });
  }
});
```

이렇게 구현하게 되면 리스트의 갯수대로 이벤트를 등록하게 된다. 이벤트를 등록하는 것 자체도 리소스를 소모하는 것이기 때문에 최대한 효율적으로하는 방법을 생각해야 하낟.

이벤트 버블링을 이용하여 부모 요소에서 이벤트를 등록하면, 하나의 이벤트 등록으로 하위 요소들의 이벤트를 제어할 수 있다.

```javascript
document.addEventListener("DOMContentLoaded", function () {
  const app = document.getElementById("todo-app");

  app.addEventListener("click", function (e) {
    if (e.target && e.target.nodeName === "LI") {
      const item = e.target;
      // onClick function
    }
  });
});
```
