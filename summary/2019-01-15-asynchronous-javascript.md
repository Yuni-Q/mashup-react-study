# 자바스크립트 비동기 프로그래밍

## 동기식 처리 모델과 비동기식 처리 모델

동기식 처리 모델에서는 순차적으로 태스크를 수행한다. 즉, 어떤 작업이 완료될 때까지 코드의 실행을 멈추고 기다리는 식이다. 이와는 달리, 비동기식 처리 모델에서는 현재 진행 중인 작업이 완료되기를 기다리지 않고 다음 작업을 계속해서 실행해나간다.

두 방식의 차이에 대한 이해를 돕기 위해, 서버에서 데이터를 받아와 화면에 표시하는 작업을 수행하는 상황을 가정해보자.

먼저, 클라이언트는 필요한 데이터를 서버에 요청한다. 이렇게 요청을 보낸 이후에 서버로부터 데이터를 응답을 받기까지는 비교적 오랜 시간을 필요로 한다. 만약 이를 동기식으로 처리하게 되면, 서버로부터 응답을 받기 전까지 모든 태스크의 수행은 블로킹(Blocking)된 상태로 남아있게 된다. 이는 사용자 경험을 완전히 망가뜨릴 수 있다. 따라서 동기식 처리 모델은 데이터 통신과 같은 오랜 시간을 필요로 하는 작업에 적합하지 않다.

위 상황에 비동기식 처리 모델을 적용하면 훨씬 더 효율적인 방식으로 작업을 수행할 수 있다. 동기식 모델과 달리, 클라이언트는 요청을 보낸 후 서버로부터 응답을 받을 때까지 코드의 실행을 중지하지 않는다. 다음 태스크를 계속해서 실행한 후, 요청에 대한 응답이 완료되었을 때 해당 태스크에 대한 처리를 재개하는 식이다. 이러한 AJAX 요청 작업과 타이머 함수, DOM 이벤트 등은 모두 비동기식 처리 모델로 동작하도록 설계되어있다.

## 자바스크립트 실행 환경

동기식/비동기식 처리 모델이 구체적으로 어떻게 동작하는지를 이해하기 위해, 먼저 자바스크립트 코드의 실행 환경에 대해 알아볼 필요가 있다. 브라우저 환경의 경우, 크게 다음의 영역들로 구성된다.

* 자바스크립트 엔진
* 태스크 큐
* 이벤트 루프
* Web APIs

![자바스크립트 실행 환경](./assets/2019-01-15-asynchronous-javascript-1.png)

### 자바스크립트 엔진

자바스크립트 코드를 해석하고 실행하는 인터프리터를 자바스크립트 엔진이라고 한다. 가장 널리 알려져있고 많이 사용되는 엔진 중 하나는 크롬 V8 엔진이다. V8 엔진은 크롬 브라우저, Node.js 등에서 사용되고 있으며, 크게 다음 두 부분으로 구성된다.

* 메모리 힙(Memory Heap): 동적으로 생성된 객체 인스턴스가 이루어지는 영역이다.
* 호출 스택(Call Stack): 요청된 작업의 스택 프레임을 순차적으로 담아 관리하는 영역이다. 싱글 스레드 기반의 자바스크립트는 단 하나의 호출 스택을 사용하기 때문에 해당 태스크가 종료되기 전까지는 다른 어떤 태스크도 수행될 수 없다.

### 태스크 큐(또는 이벤트 큐)

비동기 처리 함수의 콜백 함수, 비동기식 이벤트 핸들러, 타이머 함수(`setTimeout`, `setInterval`)의 콜백 함수가 보관되는 영역으로 이벤트 루프에 의해 특정 시점(호출 스택이 비어졌을 때)에 순차적으로 호출 스택으로 이동되어 실행된다.

### 이벤트 루프

호출 스택 내에서 현재 실행중인 태스크가 있는지 그리고 태스크 큐에 태스크가 있는지를 반복하여 확인한다. 만약 호출 스택이 비어있다면 태스크 큐 내의 태스크가 호출 스택으로 이동되고 실행된다.

### Web APIs

브라우저가 제공하는 `setTimeout` 등의 타이머 함수, `XMLHttpRequest`와 같은 AJAX 관련 객체 등을 일컫어 Web APIs라고 한다.

### 동기식 처리 모델 예시

다음은 동기식 처리 모델의 예시 코드이다.

```javascript
// 인자로 주어진 시간만큼 코드의 실행을 미루는 함수
function sleep(ms) {
  const start = Date.now();
  while ((Date.now() - start) < ms);
  console.log('Sleeping..')
}

function sayHello(name) {
  sleep(3000)
  console.log(`Hello, ${name}!`)
}

sayHello('Kim')
```

시간에 따른 호출 스택의 모습은 다음과 같다.

![동기식 처리 모델에서의 호출 스택](./assets/2019-01-15-asynchronous-javascript-2.png)

1. 코드가 실행되는 순간, 호출 스택의 최상단에 전역 실행 컨텍스트(Global execution context)가 쌓인다.
2. `sayHello`가 그 위에 쌓인다.

### 비동기식 처리 모델 예시

브라우저에 위임한다.

위임된 일이 끝나면, 그 결과와 콜백을 작업 큐(task queue)에 추가합니다.
브라우저는 호출 스택이 비워질 때마다 작업 큐에서 가장 오래된 작업을 꺼내와서 해당 작업에 대한 콜백을 실행시킵니다. 브라우저는 이 과정을 끊임없이 반복하는데, 이를 이벤트 루프(event loop)라고 부릅니다.

## 비동기 프로그래밍 기법

비동기 작업 간에 순서를 보장

여러가지 기법이 발전했다.

### 콜백 패턴(Callback)

### 1.1 콜백 패턴의 문제점

#### 1.1.1 콜백 지옥(Callback Hell)

#### 1.1.2 에러 처리의 한계

## 2. 프로미스(Promise)

## 2.1 비동기 함수(Async Function)

## References

* [프로미스 - Poiemaweb](https://poiemaweb.com/es6-promise)
* [이벤트 - Poiemaweb](https://poiemaweb.com/js-event)
* https://engineering.huiseoul.com/%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8%EB%8A%94-%EC%96%B4%EB%96%BB%EA%B2%8C-%EC%9E%91%EB%8F%99%ED%95%98%EB%8A%94%EA%B0%80-%EC%9D%B4%EB%B2%A4%ED%8A%B8-%EB%A3%A8%ED%94%84%EC%99%80-%EB%B9%84%EB%8F%99%EA%B8%B0-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D%EC%9D%98-%EB%B6%80%EC%83%81-async-await%EC%9D%84-%EC%9D%B4%EC%9A%A9%ED%95%9C-%EC%BD%94%EB%94%A9-%ED%8C%81-%EB%8B%A4%EC%84%AF-%EA%B0%80%EC%A7%80-df65ffb4e7e
* https://helloworldjavascript.net/pages/285-async.html