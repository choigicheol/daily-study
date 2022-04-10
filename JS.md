1. JS
   - [event-loop](#event-loop)

---

### event-loop

##### 들어가기전에

#### Process & thread

**Process**

- 운영체제로부터 자원을 할당받는 작업의 단위
- 디스크로부터 메모리에 적재되어 운영체제로부터 주소 공간, 파일, 메모리 등을 할당받으며 이것들을 총칭하여 프로세스라고 한다.
- 함수의 매개변수, 복귀 주소, 로컬 변수와 같은 임시 자료를 저장하는 프로세스 "스택"과 전역 변수들을 저장하는 데이터 섹션, 프로세스 실행 중에 동적으로 할당되는 메모리인 "힙"을 포함한다.

**thread**

- 프로세스가 할당받은 자원을 이용하는 실행의 단위
- 한 프로세스 내에서 동작되는 여러 실행 흐름으로 프로세스 내의 Heap, Data, Code 영역을 공유
- 하나의 프로세스를 다수의 실행 단위인 스레드로 구분하여, 자원을 공유하고 자원의 생성과 관리의 중복성을 최소화하여 수행 능력을 향상시키는 것을 멀티스레딩이라고 한다.
- 이 경우 각각의 스레드는 독립적인 작업을 수행해야 하기 때문에 각자의 스택과 PC 레지스터 값을 가지고 있다.

---

자바스크립트의 가장 큰 특징 중하나는 ‘단일 스레드'언어라는 점이다. 스레드가 하나라는 것은, 동시에 하나의 작업만을 처리할 수 있다는 것을 의미한다. 그러나 우리가 브라우저에서 사용하는것을 보면 멀티 스레드 처럼 보여진다.

실제 자바스크립트가 구동되는 환경(브라우저, Node.js등)에서는 주로 여러 개의 스레드가 사용되는데, 이러한 구동 환경이 단일 호출 스택을 사용하는 자바 스크립트 엔진과 연동하기 위해 사용되는 장치가 바로 '이벤트 루프' 이다.

핵심만 말하자면, JavaScript event loop는 call stack이 비어있는 경우, task queue에서 대기하던 callback을 call stack으로 옮겨서 callback을 실행시켜주는 역할을 한다.

---

![JSEventLoopImg](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FEzg2O%2FbtrySN8VT1f%2FYpK0kBgNB2rXLCuMqNDDZK%2Fimg.png)
`브라우저 환경에서 자바스크립트가 돌아가는 모습을 그림으로 표현하면 위와 같다.`

### 엔진 (Engine)

우리가 만드는 JS파일을 컴퓨터는 읽을 수가 없다. 따라서 컴퓨터가 이해하는 언어로 바꿔주는 작업이 필요한데, 자바스크립트 엔진은 자바스크립트 코드를 실행하는 프로그램 혹은 인터프리터를 의미한다.

자바스크립트 엔진의 종류는 다양한데, 각 엔진이 서로 다른 웹 브라우저들, Node.js에서 동작하도록 만들어져있기 때문이다. 구글이 만든 V8엔진이 가장 유명하며 크롬과 Node.js에서 사용된다.

엔진의 두 요소로는 메모리 힙과 콜스택이 있다. 메모리 힙에서는 메모리 할당이 이뤄지는데, 변수 등이 저장된다. 콜스택에는 코드 실행에 따라 호출 스택이 쌓인다. 마지막에 들어간 요소가 가장 먼저 나오는 LIFO(Last In First Out)의 구조다. 콜스택을 통해 현 시점에서 실행되고 있는 자바스크립트가 어느 부분인지 알 수 있다.

### Web API

그림의 오른쪽에 있는 Wep API는 JS Engine의 밖에 그려져 있다. 즉, 자바스크립트 엔진이 아니다. Web API 는 브라우저에서 제공하는 API 로, DOM, Ajax, Timeout 등이 있다.

Call Stack에서 실행된 비동기 함수는 Web API를 호출하고, Web API는 콜백함수를 Callback Queue에 넣는다.

### Callback Queue

비동기적으로 실행된 콜백함수가 보관 되는 영역이다. 콜백이 큐(Queue) 형태로 쌓인다. 선입선출(FIFO) 구조다. 콜백 큐 안에는 콜백 함수들이 대기하고 있다. 콜스택 안에 쌓인 작업들이 마무리되고 콜스택이 비워지고 나면, 콜백 큐 안에 대기하고 있던 콜백 함수들이 콜스택에 추가된다. 콜백 큐의 내부는 다시 Task Queue(Event Queue), Microtask Queue(Job Queue), Animatino Frames로 구성되어 있다. 이들 간의 우선순위는 다음과 같다.

`MicroTask Queue > Animation Frames > Task Queue`

### 이벤트 루프 (Event Loop)

콜스택이 비워지는 것을 어떻게 바로 알 수 있을까? 그것을 가능하게 해주는 것이 바로 이벤트 루프다. 이벤트 루프는 현재 실행 중인 작업이 없을 때(주로 콜스택이 비워졌을 때) 콜백 큐에 대기하고 있던 첫 번째 태스크를 꺼내와 콜스택으로 보내주는 역할을 한다.

```js
console.log(‘1’);

setTimeout(function callBack1(){
  console.log(‘2’)
}, 5000);

setTimeout(function callBack2(){
  console.log(‘3’)
}, 0);

console.log(‘4’)
```

해당 코드를 실행하면 콘솔이 어떻게 찍힐까?

```js
1;
4;
3;
2;
```

`정답은 1 - 4 - 3 - 2의 순서로 찍혀서 나온다.`

![JSEventLoopImg](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FEzg2O%2FbtrySN8VT1f%2FYpK0kBgNB2rXLCuMqNDDZK%2Fimg.png)

위 그림을 같이 보면서 설명하면

- 첫 번째 줄에서 console.log(‘1’)이 콜스택에 들어간 뒤, 콘솔에 ‘1’를 찍고 콜스택이 비워진다.
- 첫 번째 setTimeout 함수가 콜스택으로 들어가 실행된다. Web API가 호출되고 콜스택은 비워진다.(callBack1 함수가 Web API에서 5초간 대기한 후 콜백 큐로 이동한다)
- 두 번째 setTimtout 함수가 콜스택에 들어가고 다시 Web API를 호출한 뒤 콜스택은 비워진다. (callBack2 함수가 Web API에서 0초간 대기한 후 콜백 큐로 이동한다)
- console.log(‘4’)가 콜스택에 올라간 뒤 콘솔에 ‘4’를 찍고 콜스택을 비운다.
- 콜백 큐는 선입선출이므로 먼저 들어간 콜백이 먼저 콜스택으로 이동하게 된다. 대기시간을 고려하면 callBack2의 대기시간이 0ms, cacllBack1이 5000ms이므로 callBack2가 콜스택으로 먼저 이동할 수 있다.
- 콜백 큐에는 callBack2, callBack1 순서로 콜백이 쌓여있다. ‘4’가 콘솔에 찍힌 후 콜스택이 비워지고 나면 이벤트 루프는 다음 콜백 큐에 있던 콜백을 콜스택으로 이동시킨다.

### 프로미스와 이벤트 루프

```js
console.log(‘1’);

setTimeout(function callBack1(){
  console.log(‘2’)
}, 0);

Promise.resolve().then(function(){
  console.log('3')
})

console.log(‘4’)
```

위와같이 프로미스까지 같이 있는 경우에는 어떻게 될까?

```js
1;
4;
3;
2;
```

`정답은 1 - 4 - 3 - 2 이다`

setTimeout 함수의 대기시간이 0ms임에도 불구하고 3이 2보다 먼저 콘솔이 찍혀서 나오는 이유는 Task Queue의 우선순위 때문이다. 이전에도 말한바와 같이 Task Queue에도 우선순위가 있는데 (Microtask Queue > Animation Frames > Task Queue) 일반적인 작업은 대부분 Task Queue에 쌓인다. setTimeout 역시 Task Queue에 쌓인다. 하지만 프로미스의 경우 Task Queue가 아닌 Microtask Queue에 쌓인다.

Microtask Queue가 다 비워진 후에야 Task Queue의 콜백이 실행되므로, 프로미스가 실행된 이후 setTimeout 콜백이 실행된다. 그렇기 때문에 위와 같은 순서로 콘솔이 찍히게 되는 것이다.
