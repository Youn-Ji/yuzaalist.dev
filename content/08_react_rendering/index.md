---
emoji: 🎢
title: 리액트 콘솔 대잔치
date: '2023-10-11 20:00:00'
author: 유자
tags: React setState 비동기
categories: challenge
---

## 왜 그러는거야?
직장에서 친해진 개발자 분들과 이런 저런 이야기를 나누는 디스코드 방이 있습니다. 항상 요상한 문제를 내서 시험에 들게 하는 분이 계신데 오늘은 이런 문제를 가지고 오셨습니다. 

```js
function App () {
 const [state, setState] = useState(0)
 console.log(0)

 useEffect(()=>{
  console.log(1)
 },[state])

 Promise
 .resolve()
 .then(() => console.log(2))

 setTimeout(()=>console.log(3), 0)

 useLayoutEffect(() => {
  console.log(4)
  setState(s=>s+1)
 }, [])

 return <div/>
}
```

싸늘하다.. 비수가 날아와

App이라는 '함수'가 실행되면서 0이 가장 먼저 찍힐 것이고, layout이 페인트 되기 전에 useLayoutEffect가 실행되니까 4가 찍힐거라 생각했습니다. 그 다음에 useEffect가 돌면서 1이 찍힐 것 같습니다.
Promise then의 콜백은 브라우저 이벤트 루프의 마이크로 태스크 큐로, setTimeout의 콜백은 태스크 큐로 들어갑니다.
마이크로 태스크 큐의 우선순위가 태스크 큐보다 높기 때문에 2, 3의 순서로 찍힐 것이라 예상했습니다. 

결론은 0 4 1 2 3.

앗 근데 useLayoutEffect안에 있는 setState를 까먹었습니다. setState까지 고려해야할까요? 설마 그런 문제가 어딨냐며 말도 안된다면서 위의 답을 보냈습니다. 
하지만 setState까지 고려해야하는 문제가 맞았습니다..

콘솔에 찍히는 로그는 0 4 1 0 1 2 2 3 3 였습니다. (여기 [StackBlitz](https://stackblitz.com/edit/stackblitz-starters-v3pkby?description=React%20%20%20TypeScript%20starter%20project&file=src%2FApp.tsx,src%2Findex.tsx&title=React%20Starter)에서 확인 가능합니다)

## 리액트 비동기 vs 자바스크립트 비동기

평소 리액트에서 이야기하는 setState의 비동기 동작과 자바스크립트의 비동기 동작을 크게 구분하지 않고 쓰고 있었습니다. 
setState가 비동기로 동작한다고 말하는 이유는 여러번 호출됐을 때, 내부의 스케줄러를 통해 한번에 state를 업데이트하는 방식 때문입니다. 

이 [스케줄러](https://github.com/facebook/react/blob/main/packages/scheduler/src/forks/Scheduler.js)는 브라우저 엔진에서 제공되는 것이 아닌 자바스크립트로 작성된 코드이며, 리액트 고유의 렌더링 프로세스입니다. 

그럼 스케줄러 내부에서는 브라우저 이벤트 루프를 타는 비동기 로직을 사용하지 않는 것일까요? 

아래는 리액트 스케줄러 코드의 일부분을 발췌한 것입니다. 사용할 수 있는 웹 API를 체크해서 스케줄러가 실행되는 환경을 나눕니다. DOM and Worker environments에서 MessageChannel을 사용해 메시지를 주고받습니다. 이는 Web Worker 기반으로 완전히 다른 스레드를 사용합니다. 
이런저런 스택이나 큐가 쌓여있는 기존 스레드가 아닌 새로운 스레드를 타는 것이니 훨씬 속도가 빠를 것 같습니다. 

```js
// Capture local references to native APIs, in case a polyfill overrides them.
const localSetTimeout = typeof setTimeout === 'function' ? setTimeout : null;
const localClearTimeout =
  typeof clearTimeout === 'function' ? clearTimeout : null;
const localSetImmediate =
  typeof setImmediate !== 'undefined' ? setImmediate : null; // IE and Node.js + jsdom

.
.
.

let schedulePerformWorkUntilDeadline;
if (typeof localSetImmediate === 'function') {
  // Node.js and old IE.
  // There's a few reasons for why we prefer setImmediate.
  //
  // Unlike MessageChannel, it doesn't prevent a Node.js process from exiting.
  // (Even though this is a DOM fork of the Scheduler, you could get here
  // with a mix of Node.js 15+, which has a MessageChannel, and jsdom.)
  // https://github.com/facebook/react/issues/20756
  //
  // But also, it runs earlier which is the semantic we want.
  // If other browsers ever implement it, it's better to use it.
  // Although both of these would be inferior to native scheduling.
  schedulePerformWorkUntilDeadline = () => {
    localSetImmediate(performWorkUntilDeadline);
  };
} else if (typeof MessageChannel !== 'undefined') {
  // DOM and Worker environments.
  // We prefer MessageChannel because of the 4ms setTimeout clamping.
  const channel = new MessageChannel();
  const port = channel.port2;
  channel.port1.onmessage = performWorkUntilDeadline;
  schedulePerformWorkUntilDeadline = () => {
    port.postMessage(null);
  };
} else {
  // We should only fallback here in non-browser environments.
  schedulePerformWorkUntilDeadline = () => {
    // $FlowFixMe[not-a-function] nullable value
    localSetTimeout(performWorkUntilDeadline, 0);
  };
}
```
<div style="display: flex; justify-content: center; font-size: 14px; color: gray"><span>출처 https://github.com/facebook/react/blob/main/packages/scheduler/src/forks/Scheduler.js#L586-L617</span></div>
<br/>


리액트가 브라우저에서 렌더링되면 Web Worker에서 주고받는 메시지 기반으로 스케줄링 될테니 기존 스레드의 이벤트 루프보다 훨씬 빠른 연산이 가능할 것으로 보입니다. 

이를 통해 리액트에서 이야기하는 비동기가 브라우저 레벨의 비동기가 아닌 것은 확실히 알 수 있었습니다. 

그래서 로그도 `0 4 1 0 1` / `2 2 3 3` 이렇게 리액트 코드의 로그, 콜백 코드의 로그 2개의 파트로 나뉘어지는 것이 아닐까요.

리액트 내에서 복잡하게 돌아가는 렌더링 작업들은 결국 자바스크립트 레벨로 봤을 때 하나의 동기 로직이고, Promise, setTimeout과 같은 찐(?) 비동기 로직만이 브라우저 엔진의 이벤트 루프를 탈 수 있다는 것을 다시 한번 되새길 수 있는 날이었습니다. 

그럼 이만.

```toc
```




