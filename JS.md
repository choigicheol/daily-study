1. JS
   - [event-loop](#event-loop)
   - [변수](#변수)
      + [선언](#선언)
      + [초기화](#초기화)
      + [할당](#할당)
      + [Temporal Dead Zone](#temporal-dead-zone-호이스팅)
    - [스코프 Scope](#스코프-scope)
    - [클로저 Closure](#클로저-closure)
---

## event-loop

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

---

## 변수

- 변수는 하나의 값을 저장하기 위해 확보한 메모리 공간 자체 또는 그 메모리 공간을 식별하기 위해 붙인 이름을 말한다.

![variable](https://miro.medium.com/max/700/1*IiejRUFbks-TaOzJJvdoVw.jpeg)

```js
const myNumber = 23
// 변수명(식별자): myNumber
// 해당 값의 위치(메모리 주소): 0012CCGWH80
// 변수 값(저장된 값): 23
```

자바스크립트는 매니지드 언어(managed language)이기 때문에 개발자가 직접 메모리를 제어하지 못한다. 따라서 개발자가 직접 메모리 주소를 통해 값을 저장하고 참조할 필요가 없고 변수를 통해 안전하게 값에 접근이 가능하다.

> Managed Language 매니지드 언어
자바스크립트 같은 매니지드 언어는 메모리의 할당 및 해제를 위한 메모리 관리 기능을 언어 차원에서 담당하고 개발자의 직접적인 메모리 제어를 허용하지 않는다. 따라서 개발자가 직접 메모리를 할당하고 해제할 수 없다. 재할당에 의해 더 이상 사용하지 않는 메모리의 해제는 가비지 콜렉터가 수행한다. 매니지드 언어는 개발자의 역량에 의존하는 부분이 상대적으로 작아져 어느 정도 일정한 생산성을 확보할 수 있다는 장점이 있지만 성능 면에서는 어느 정도 손실은 감수할 수밖에 없다.

> UnManaged Language 언매니지드 언어
C언어 같은 언매니지드 언어는 개발자가 명시적으로 메모리를 할당하고 해제하기 위한 malloc()과 free() 같은 저수준 메모리 제어 기능을 제공한다. 이렇듯 언매니지드 언어는 개발자가 직접 메모리 제어를 주도할 수 있으므로 개발자의 역량에 따라 최적의 성능을 확보할 수 있지만 능숙하지 않다면 오히려 치명적인 오류를 발생할 가능성이 있다.


`변수명(식별자)인 myNumber는 변수의 값이 아닌 메모리 주소를 기억`하고 있다. 변수명을 사용하면, 자바스크립트 엔진이 변수명과 매핑된 메모리 주소를 통해 저장된 값(23)을 반환한다.
---
### 선언
자바스크립트에서의 `변수 선언(Declaration)은 실행 컨텍스트의 변수 객체에 변수를 등록하는 단계`를 의미한다. `이 변수 객체는 스코프가 참조하는 대상`이다. 한 마디로, 스코프에 변수를 등록하는 단계이며 `이 단계에서 호이스팅이 일어난다.`

### 초기화
`초기화(Initialization)는 실행 컨텍스트에 존재하는 변수 객체에 선언 단계의 변수를 위한 메모리를 만드는 단계`이다. 이 단계에서 `할당된 메모리에 undefined로 초기화` 된다.

### 할당
`할당(Assignment)은 undefined로 초기화 된 메모리에 다른 값을 넣는 것`이다.

### Temporal Dead Zone (+호이스팅)
호이스팅은 **var, let, const, function, class** 키워드 등을 사용해서 선언하는 모든 식별자(변수, 함수, 클래스 등)가 코드의 선두로 끌어 올려진 것처럼 동작하는 자바스크립트 고유의 특징이다.

우선 let,const와 var의 호이스팅 방식의 차이를 봐야한다.  

**var의 경우** `선언 단계와 함께 undefined로 초기화`되므로 **초기화 코드를 만나기 전부터 참조가 가능**하다.

반면에, **let, const**로 선언된 변수는 `선언 단계와 초기화 단계가 분리되어 진행`된다. 즉, **스코프에 변수를 등록하지만, 초기화 단계는 변수 선언문 코드에 도달했을때 이루어지기 때문에** 초기화 이전에 변수에 접근하려고 하면 **참조 에러(Reference Error)가 발생**한다.

따라서, `스코프의 시작 지점부터 초기화 시작 지점까지는 변수를 참조할 수 없는`데, 이를 **Temporal Dead Zone**이라고 한다.  

**let**과 **const**는 **호이스팅이 되기는 하지만** 초기화가 이루어지지 않은 상태(Uninitialized)에서 호이스팅이 되기 때문에 **초기화 단계를 만나기 전에는 참조할 수가 없으**며 일시적 사각지대 (TDZ)가 생기는 것이다.

---
# 스코프 Scope
자바스크립트에서 스코프는 **변수가 유효할 수 있는 범위**이며 일반적으로 중괄호로 감싸진 영역을 말한다.  
핵심만 말하자면, 스코프는 변수의 수명을 결정하고 확인할 수 있는 범위이다.  

스코프는 크게 Local Scope와 Global Scope로 나눌 수 있다.  
Global Scope는 최상단의 스코프로써 이 곳에서 선언된 변수(전역 변수)는 어떤 영역에서든 접근이 가능하다.  
Local Scope는 Global Scope에 포함되어 있는 영역으로 이곳에서 선언된 변수(지역 변수)는 전역(Global)에서 선언된 변수보다 더 높은 우선순위를 가진다.  

```js
let age = 10;
function printAge(){
   let age = 15;
   console.log(age); // 15
}
```

앞서 언급한 Global / Local 두 스코프에서의 변수들 간에는 반드시 지켜야 하는 규칙이 있다. 바로 각 영역에서 선언된 변수들끼리의 접근 가능 여부이다.  
Local Scope에서 선언된 변수는 Global Scope에선 참조가 불가능하다. 하지만 Global Scope에서 선언된 변수(전역 변수)는 Local Scope에서 참조가 가능하다.

```js
let global = 'Global';
function checkAccess(){
   let local = 'Local';
   console.log(global);  
}
checkAccess(); // 'Global'
console.log(local); // ReferenceError
```
---
# 클로저 Closure

**클로저는 내부함수의 변수가 외부함수의 변수에 접근할 수 있는 매커니즘이다.**  

**함수와 함수가 선언된 어휘적 환경(lexical environment)의 조합을 통해 만들어진 매커니즘.  
이 환경은 클로저가 생성된 시점의 유효 스코프 내에 있는 모든 지역 변수로 구성된다. 그래서 클로저가 포함된 내부 함수에서 외부 함수의 스코프에 접근할 수 있다.**  

일반적으로 함수가 실행될 때 생성된 컨텍스트는 함수가 종료될 때 가비지컬렉션의 수집대상이 되어 사라진다. 하지만 클로져 패턴이 사용된 경우에는 내부함수의 변수가 언제 외부함수의 변수를 참조할지 알 수 없기 때문에 외부함수가 종료되어도 가비지 컬렉션의 수집대상이 되지않고 메모리상에 남아있게 된다. 이런 이유로 클로저 패턴을 남발하게 되면 메모리 누수가 발생하고 이로 인해 퍼포먼스 저하가 일어날 수 있다.

```js
function outerFn(){
	let outerVar = 'outer';
	console.log(outerVar);

	//클로저 함수
	function innerFn(){
	   let innerVar = 'inner';
	   console.log(innerVar);
	}
	//클로저 함수 안에서는
	//지역변수(innerVar)
	//외부함수의 변수(outerVar)
	//전역변수(globalVar)
	//접근이 모두 가능합니다.
	return innerFn;
}
let globalVar = 'global';
let innerFn = outerFn();
innerFn();
```
