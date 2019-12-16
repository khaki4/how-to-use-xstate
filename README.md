# XState

> finite state machines have five parts:

- A finite number of states
- A finite number of events
- An initial state
- A transition function that determines the next state given the current state and event
- A (possibly empty) set of final states

---

```javascript
const [peopleMachine, send] = useMachine(fetchMachine, {
    actions: {
      fetchData: () => {
        fetchPeople()
          .then(r => r.results)
          .then(
            results => {
              console.log(results);
              send({ type: 'RESOLVE', results });
            },
            message => {
              console.log(message);
              send({ type: 'REJECT', message });
            }
          );
      }
    }
  });
```

`fetchData` `action`의 구체적인 내용은 실행 부에서 삽입하는 형태

- 장점: fetchMachine의 추상화가 단계가 높아진다.
- 단점: 아래 코드와 같이 fetchData의 중복이 발생 한다.

```javascript
const [aState, sendA] = useMachine(fetchMachine, {
    actions: {
      fetchData: () => {
        fetchApiA()
          .then(r => r.results)
          .then(
            results => {
              send({ type: 'RESOLVE', results });
            },
            message => {
              send({ type: 'REJECT', message });
            }
          );
      }
    }
  });

  const [bState, sendB] = useMachine(fetchMachine, {
    actions: {
      fetchData: () => {
        fetchApiB()
          .then(r => r.results)
          .then(
            results => {
              send({ type: 'RESOLVE', results });
            },
            message => {
              send({ type: 'REJECT', message });
            }
          );
      }
    }
  });
```

---
위 패턴의 단점이 발생하는 중복을 제거하려면 
```javascript
.then(
    results => {
      send({ type: 'RESOLVE', results });
    },
    message => {
      send({ type: 'REJECT', message });
    }
  );
```
부분을 `fetchMachine` 안으로 보내면 된다.

그렇게 코드를 작성하면

```javascript
const [aState, sendA] = useMachine(fetchMachine, {
    services: {
      fetchData: (ctx, event) => fetchApiA().then(r => r.results)
    }
  });

const [aState, sendB] = useMachine(fetchMachine, {
    services: {
      fetchData: (ctx, event) => fetchApiB().then(r => r.results)
    }
  });
```

`actions` -> `services` 로 바뀐다.

[gist link](https://xstate.js.org/viz/?gist=f13a7dd1f1cd8a1ff2346d328f2a28aa)
<img src="./Screen Shot 2019-12-14 at 12.01.16 PM.png" />

[gist link](https://xstate.js.org/viz/?gist=4a2430de75b191b91308859cb340f48a)
<img src="./Screen Shot 2019-12-14 at 12.56.08 PM.png" />

[gist link](https://xstate.js.org/viz/?gist=7b429d7f098a5d43c705c280509fe8c1)
<img src="./Screen Shot 2019-12-15 at 12.21.36 AM.png" />

### 위 차트에 대한 추가 설명 [reference](https://ko.wikipedia.org/wiki/%EC%9C%A0%ED%95%9C_%EC%83%81%ED%83%9C_%EA%B8%B0%EA%B3%84)

### 수리 기계와 인식기

수리 기계(acceptors)와 인식기(recognizers)는 입력값이 기계에서 받아 들여졌는지 이진값인 '예 또는 아니오'로 결과를 출력한다. 유한 오토마타의 모든 상태는 받아들여지는지 또는 받아들여지지 않는지에 대한 값을 가지고 있다. 모든 입력이 처리되었을때, 만약 현재 상태가 받아들여질 수 있는 상태라면 그 입력값은 기계에 의해 받아들여진 것이다. 반대로 현재 상태가 받아들여질 수 없는 상태라면 그 입력값은 거부된 것이다.

#### 무어 모델(Moore model)

이 종류의 유한 상태 오토마타는 오직 진입 동작만을 사용한다. 즉 출력값은 오직 현재 상태에 따라서만 결정된다. 무어 기계의 장점은 행위를 단순화시킬 수 있다는 것이다. 엘리베이터 문을 예시로 들자면 유한 상태 기계는 “문을 열어라”와 “문을 닫으라” 라는 상태를 변경해 주는 두 명령어를 인식할 수 있다. “열리는 중”의 상태에 있는 진입 동작은 모터를 돌려 문을 열기 시작하는 것이고 “닫히는 중”의 상태에 있는 진입 동작은 모터를 반대로 돌려 문을 닫는 것이다. “열림”과 “닫힘”의 상태는 완전히 열리거나 닫힌 상태에서 모터를 정지시킨다.

#### 밀리 모델(Mealy model)
이 종류의 유한 상태 오토마타는 오직 입력값만을 사용한다. 즉 출력 값은 입력 값과 현재 상태 모두에 의존한다. 밀리 기계는 일반적으로 상태의 수를 줄이는데 사용한다. 앞의 무어 기계와 같은 엘리베이터 문을 예시로 들자면 무어 모델과 달리 중간의 “열리는 중”과 “닫히는 중”의 상태가 없다. 또한 진입 동작이 아니라, 다음과 같이 현재 상태와 입력 값에 모두 영향을 받는 두 종류의 입력 행위가 있다. “만약 문을 열어라 라는 입력 값이 들어온다면 문을 열기 위해 모터를 작동시킨다”와 “만약 문을 닫아라 라는 입력 값이 들어온다면 문을 닫기 위해 모터를 반대 방향으로 작동시킨다”라는 두 입력 행위가 존재한다.

---
### Parallel State

직교성(Orthogonality)을 가지는 상태 관리

[gist link](https://xstate.js.org/viz/?gist=17b90095e49189b4fe4fbaddaf8feede)
<img src="./Screen Shot 2019-12-15 at 8.43.54 PM.png" />

---
### History

[gist link](https://xstate.js.org/viz/?gist=0be1e082b0a20e58d208a113c810bbac)
<img src="./Screen Shot 2019-12-16 at 9.47.37 PM.png" />
