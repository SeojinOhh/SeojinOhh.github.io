---
title: 서버에 Docker Container 생성, 프로젝트 실행하기
date: 2024-09-20 12:00:00 +09:00
categories: [과제]
tags: [Docker, Container]
---

## **0. 서버 접속하기**
---
vscode를 통해 서버에 접속한다. 패스워드 입력.

## **1. Docker 이미지 다운로드**
---
원하는 [Docker](https://hub.docker.com/) Image를 `pull`하여 다운받는다.
`docker pull <Image>`

NVIDIA의 PyTorch 이미지를 사용해 GPU 지원이 가능하도록 설정하였다.

```bash
$ docker pull nvcr.io/nvidia/pytorch:24.07-py3
```

## **2. Docker 컨테이너 생성**
---
프로젝트 디렉토리와 Docker 컨테이너를 연결하기 위해 `-v` 플래그를 사용하여 volume을 설정한다.

아래의 커맨드에서 `/home/cybermarine/workspace/flux` 부분에는 원하는 디렉토리의 경로를 넣고, `flux_container` 부분에는 원하는 컨테이너의 이름을 입력.

이렇게 실행시키면, 원하는 폴더를 `/workspace`로 삼는 Docker Container가 생성.

```bash
$ docker run --gpus all -it -v </home/cybermarine/workspace/flux>:/workspace/flux --name <flux_container> --ipc=host --privileged nvcr.io/nvidia/pytorch:24.07-py3
```

## **3. FLUX 프로젝트 실행**
---
먼저 컨테이너 내부로 접속한다.

```bash
$ docker exec -it flux_container /bin/bash
```

진행 중 컨테이너가 실행되지 않고 중지된 상태라는 문제가 발생하면 아래의 방법으로 해결 후 다시 실행.

```bash
$ docker ps -a                  //중지된 컨테이너 확인
$ docker start flux_container   //중지된 상태라면, 컨테이너 다시 시작
```

컨테이너 내부에서 FLUX 프로젝트 디렉토리로 이동한다.

```bash
$ cd /workspace/flux
```

FLUX 프로젝트의 의존성을 설치하기 위해, `npm install` 명령어를 사용한다. 프로젝트에서 필요한 패키지들을 설치. 의존성 충돌이 있을 수 있으므로 `--force` 플래그를 사용해 강제로 설치

```bash
$ npm install --force
```

의존성 설치가 완료되면, 프로젝트를 빌드한다. 빌드는 FLUX 모듈을 생성하는 과정.

```bash
$ npm run build
```

FLUX 프로젝트가 빌드되면, 생성된 `Flux.js` 파일을 다른 React 프로젝트에서 사용할 수 있다. 빌드된 파일은 `dist` 폴더에 저장된다.

파일 권한 문제로 인한 에러가 발생할 경우 flux 디렉토리와 그 하위 파일들에 대해 현재 사용자가 읽고 쓸 수 있도록 권한을 변경해야한다.
아래의 명령어로 디렉토리와 파일의 소유권을 변경 가능.

```bash
$ sudo chown -R $USER:$USER /home/cybermarine/workspace/flux
```

React 애플리케이션을 생성한다.

```bash
$ npx create-react-app my-flux-app
```

생성된 프로젝트 디렉토리로 이동.

```bash
$ cd my-flux-app
```

개발 서버를 시작한다. 브라우저에서 `http://localhost:3000`에 접속하여 프로젝트를 확인 가능.

로컬에서 React 애플리케이션을 개발하고, Flux 패턴을 적용하려면 Flux 라이브러리를 설치하고 애플리케이션에 통합하는 작업 필요. 

Flux 프로젝트를 애플리케이션에서 사용하려면, flux 패키지를 설치해야 한다.

```bash
$ npm install flux
```

React 프로젝트에서 Flux를 사용하려면 Dispatcher를 불러오고 사용하는 코드를 작성해야 한다. Dispatcher는 Flux 패턴에서 중심적인 역할. 이 객체는 액션(action)을 받아서 스토어(store)로 전달하는 역할.

먼저 Dispatcher를 설정하는 파일을 만들어야 한다. 프로젝트의 `src` 폴더 안에 `dispatcher.js` 파일을 생성.

```bash
$ cd src
$ touch dispatcher.js
```

`dispatcher.js` 파일에 Dispatcher 코드를 작성한다.

프로젝트 폴더에서 `code .` 명령어를 입력하면 VS Code가 해당 폴더를 열어준디.

```javascript
// src/dispatcher.js
import { Dispatcher } from 'flux';

// 새로운 Dispatcher 객체 생성
const AppDispatcher = new Dispatcher();

export default AppDispatcher;
```

다음으로 액션(Action)을 만든다. 액션은 사용자의 인터페이스 상에서 발생하는 이벤트나 작업을 설명하는 객체.

액션을 정의할 파일을 만들어야 한다. `src` 폴더 내에 `actions` 폴더를 만들고 `myActions.js` 파일을 생성

```bash
$ mkdir src/actions
$ touch src/actions/myActions.js
```

```javascript
import AppDispatcher from '../dispatcher'; // Dispatcher를 불러옴

// 액션 객체를 정의합니다.
export const myActions = {
  incrementCounter() {
    // 액션을 Dispatcher에 전달
    AppDispatcher.dispatch({
      actionType: 'INCREMENT_COUNTER',
    });
  },
  resetCounter() {
    // 액션을 Dispatcher에 전달
    AppDispatcher.dispatch({
      actionType: 'RESET_COUNTER',
    });
  },
};
```

이제 스토어(Store)를 만든다. 스토어는 애플리케이션의 상태를 저장하는 역할.

스토어를 설정할 파일을 생성하고, 스토어 코드를 작성한다. 

```bash
$ mkdir src/stores
$ touch src/stores/myStore.js
```

```javascript
import { ReduceStore } from 'flux/utils'; // flux 패키지에서 ReduceStore를 불러옵니다.
import Dispatcher from '../dispatcher'; // 우리가 만든 디스패처를 가져옵니다.

// 스토어 클래스를 정의합니다.
class MyStore extends ReduceStore {
  constructor() {
    super(Dispatcher); // 디스패처와 연결합니다.
  }

  // 초기 상태를 설정합니다.
  getInitialState() {
    return {
      count: 0 // 기본적으로 count는 0입니다.
    };
  }

  // 액션에 따라 상태를 변경하는 함수입니다.
  reduce(state, action) {
    switch (action.type) {
      case 'INCREMENT': // INCREMENT 액션이 발생하면
        return {
          ...state, // 기존 상태를 복사한 후
          count: state.count + 1 // count를 1 증가시킵니다.
        };
      case 'DECREMENT': // DECREMENT 액션이 발생하면
        return {
          ...state, // 기존 상태를 복사한 후
          count: state.count - 1 // count를 1 감소시킵니다.
        };
      default:
        return state; // 기본적으로 상태는 변경되지 않습니다.
    }
  }
}

// 스토어 객체를 생성하고 내보냅니다.
const myStoreInstance = new MyStore(); // 인스턴스를 변수에 할당
export default myStoreInstance; // 변수를 내보냄
```

이제 React 컴포넌트에서 스토어를 사용하기 위해 `src/App.js` 파일을 열고 아래와 같이 작성.

```javascript
import React, { useState, useEffect } from 'react';
import MyStore from './stores/myStore'; // 우리가 만든 스토어를 가져옵니다.
import Dispatcher from './dispatcher'; // 디스패처를 가져옵니다.

function App() {
  // 스토어에서 가져올 상태를 관리하기 위해 useState를 사용합니다.
  const [count, setCount] = useState(MyStore.getState().count);

  // 스토어에서 상태가 변경되면 호출되는 함수입니다.
  const onChange = () => {
    setCount(MyStore.getState().count); // 스토어의 상태가 변경되면 count를 업데이트합니다.
  };

  // 컴포넌트가 마운트될 때 스토어의 변경사항을 구독(subscribe)하고, 언마운트될 때 구독을 해제합니다.
  useEffect(() => {
    MyStore.addListener(onChange); // 스토어에 변화가 생기면 onChange 함수가 호출됩니다.
    return () => {
      MyStore.removeListener(onChange); // 컴포넌트가 언마운트되면 구독을 해제합니다.
    };
  }, []);

  // INCREMENT 액션을 디스패처로 보냅니다.
  const increment = () => {
    Dispatcher.dispatch({
      type: 'INCREMENT' // INCREMENT 액션을 디스패처에 전달합니다.
    });
  };

  // DECREMENT 액션을 디스패처로 보냅니다.
  const decrement = () => {
    Dispatcher.dispatch({
      type: 'DECREMENT' // DECREMENT 액션을 디스패처에 전달합니다.
    });
  };

  return (
    <div className="App">
      <h1>Count: {count}</h1> {/* 스토어에서 가져온 count 값을 표시합니다. */}
      <button onClick={increment}>Increment</button> {/* 버튼을 누르면 INCREMENT 액션을 디스패처에 전달합니다. */}
      <button onClick={decrement}>Decrement</button> {/* 버튼을 누르면 DECREMENT 액션을 디스패처에 전달합니다. */}
    </div>
  );
}

export default App;
```

Flux 패턴이 적용된 애플리케이션을 실행하고 동작을 확인한다. 프로젝트의 루트 디렉토리로 이동한 다음, 애플리케이션을 실행.

```bash
$ npm start
```

*Compiled successfully!*라는 문구가 뜨면 프로젝트가 성공적으로 컴파일된 것.
브라우저에서 `http://localhost:3000`을 방문하여 Flux를 사용한 React 애플리케이션을 확인할 수 있다.

중간중간 사소한 에러가 많이 발생했지만 에러 내용과 해결 과정 기제는 생략했음.