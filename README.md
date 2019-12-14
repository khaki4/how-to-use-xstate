# XState

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