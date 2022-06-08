2. REACT
   - [Hooks](#hooks)
   - [useCallback](#useCallback)
   <!-- - [변수](#변수)
      + [선언](#선언)
      + [초기화](#초기화)
      + [할당](#할당)
      + [Temporal Dead Zone](#temporal-dead-zone-호이스팅)
   - [스코프 Scope](#스코프-scope)
     - [함수레벨 스코프](#함수레벨-스코프function-level-scope)
     - [블록레벨 스코프](#블록레벨-스코프block-level-scope)
   - [var,let,const 차이](#varletconst-차이)
   - [클로저 Closure](#클로저-closure)
   - [비동기, callback, promise, async/await](#비동기-callback-promise-asyncawait) -->

---

# hooks

#### useMemo

- 최적화를 위한 hook
  함수형 컴포넌트는 jsx를 리턴하는 함수이다. 함수형 컴포넌트가 랜더링 된다는것은 함수가 호출된다는 것이다.
  함수가 호출 될 때 마다 함수 내부의 표현식(변수, 함수 등)이 초기화 되기때문에 무거운 작업의 경우 결과가 변하지 않음에도 불과하고 계속해서 호출이 된다.

**useMemo의 형식**

> const memo = useMemo(() => {}, []) // 첫번째 인자로는 콜백함수, 두번째인자로는 의존성배열을 받는다.
> 의존성배열로 전달해주는 변수에 변화가 있을때만 콜백함수를 실행해 새로운값을 반환하고,
> 다른 값에 의해 컴포넌트가 리랜더링 될때는 기존에 메모이제이션한 값을 가져와 사용한다.

```js
import react, { useMemo, useState } from "react";

const hardCalculate = (num) => {
  for (let i = 0; i < 999999999; i++) {}
  console.log("hardCalculate");
  return num + 10000;
};

const easyCalculate = (num) => {
  return num + 1;
};

function App() {
  const [hardNum, setHardNum] = useState(1);
  const [easyNum, setEasyNum] = useState(1);

  const hardSum = hardCalculate(hardNum);
  const easySum = easyCalculate(easyNum);
  return (
    <>
      <h3>느린 계산기</h3>
      <input
        type="number"
        value={hardNum}
        onChange={(e) => setHardNum(parseInt(e.target.value))}
      />
      <span> + 10000 = {hardSum} </span>

      <h3>빠른 계산기</h3>
      <input
        type="number"
        value={easyNum}
        onChange={(e) => setEasyNum(parseInt(e.target.value))}
      />
      <span> + 1 = {easySum} </span>
    </>
  );
}

export default App;
```

> 위의 코드는 빠른계산기의 입력값을 변경해도 개발자도구에 "hardCalculate"가 찍히며 똑같이 딜레이가 발생한다.
> 느린계산기의 값이 변경될때 App 컴포넌트가 리랜더링 되면서 hardSum도 초기화 되며 HardCalculate()가 실행되기 때문이다.

```js
import react, { useMemo, useState } from "react";

const hardCalculate = (num) => {
  for (let i = 0; i < 999999999; i++) {}
  console.log("hardCalculate");
  return num + 10000;
};

const easyCalculate = (num) => {
  return num + 1;
};

function App() {
  const [hardNum, setHardNum] = useState(1);
  const [easyNum, setEasyNum] = useState(1);

  const hardSum = useMemo(() => {
    return hardCalculate(hardNum);
  }, [hardNum]);

  const easySum = easyCalculate(easyNum);

  return (
    <>
      <h3>느린 계산기</h3>
      <input
        type="number"
        value={hardNum}
        onChange={(e) => setHardNum(parseInt(e.target.value))}
      />
      <span> + 10000 = {hardSum} </span>

      <h3>빠른 계산기</h3>
      <input
        type="number"
        value={easyNum}
        onChange={(e) => setEasyNum(parseInt(e.target.value))}
      />
      <span> + 1 = {easySum} </span>
    </>
  );
}

export default App;
```

> useMemo를 이용해 무거운 작업에 대해 의존성배열의 값이 변할때만 실행되게끔 수정하였다.
> 이제는 빠른계산기의 input 값이 변경되어 리랜더링이 일어났을때 hardSum에 저장되었던 값을 다시 불러오기때문에 빠른계산기에서는 딜레이가 발생하지않는다.

그러나 함수 내부의 변수가 초기화 되면서 1초 이상 걸리는 경우가 많지 않다. useMemo를 더 유용하게 사용하는 경우는 다음과 같다.

```js
// case 1
import react, { useMemo, useState, useEffect } from "react";

function App() {
  const [name, setName] = useState("");
  const [isAdult, setIsAdult] = useState(false);

  const message = isAdult ? "성인" : "미성년자",


  useEffect(() => {
    console.log("useEffect");
  }, [message]);

  return (
    <>
      <h3>이름 입력</h3>
      <input
        type="text"
        value={name}
        onChange={(e) => setName(e.target.value)}
      />

      <h3>성인 인가요</h3>
      <button onClick={() => setIsAdult(true)}>yes</button>
      <button onClick={() => setIsAdult(false)}>no</button>
      <span>{message}</span>
    </>
  );
}

export default App;
```

```js
// case 2
import react, { useMemo, useState, useEffect } from "react";

function App() {
  const [name, setName] = useState("");
  const [isAdult, setIsAdult] = useState(false);

  const message = {
    data: isAdult ? "성인" : "미성년자",
  };

  useEffect(() => {
    console.log("useEffect");
  }, [message]);

  return (
    <>
      <h3>이름 입력</h3>
      <input
        type="text"
        value={name}
        onChange={(e) => setName(e.target.value)}
      />

      <h3>성인 인가요</h3>
      <button onClick={() => setIsAdult(true)}>yes</button>
      <button onClick={() => setIsAdult(false)}>no</button>
      <span>{message.data}</span>
    </>
  );
}

export default App;
```

> 두개의 코드에서 차이는 useEffect가 실행되는 조건인 의존성배열안의 변수의 변화이다.
> case1의 코드는 input에서 name을 변경하여 App이 리랜더링이 되더라도 message의 값이 원시타입이기 때문에 값이 변경되지않아 useEffect가 실행되지않는다.
> 반대로 case2 의 코드는 input에 입력이 발생하면 App이 리랜더링 되면서 message를 초기화하는데
> message가 object이기 때문에 초기화 시 주소의 변경이 일어나 해당 변수가 변한것으로 간주한다.
> 때문에 input에 입력이 변경되고 state가 업데이트 되면서 App컴포넌트가 리랜더링 될 때 useEffect가 실행되어 버린다. (만약 useEffect에서 API 호출과 같은 무거운 작업이 진행된다면?)

```js
// case 2
import react, { useMemo, useState, useEffect } from "react";

function App() {
  const [name, setName] = useState("");
  const [isAdult, setIsAdult] = useState(false);

  const message = useMemo(() => {
    return { data: isAdult ? "성인" : "미성년자" };
  }, [isAdult]);

  useEffect(() => {
    console.log("useEffect");
  }, [message]);

  return (
    <>
      <h3>이름 입력</h3>
      <input
        type="text"
        value={name}
        onChange={(e) => setName(e.target.value)}
      />

      <h3>성인 인가요</h3>
      <button onClick={() => setIsAdult(true)}>yes</button>
      <button onClick={() => setIsAdult(false)}>no</button>
      <span>{message.data}</span>
    </>
  );
}

export default App;
```

> App 컴포넌트가 초기화 될때 message가 초기화 되는것을 막으면 된다. isAdult의 변화가 있을때만 변경이 되도록 해준다.
