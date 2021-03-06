# offscreencanvas-study

간단한 offscreencanvas 실습 저장소입니다.

### 실습내용

- 기존의 이미지 load와 같이 시간이 소요되는 요소를 메인쓰레드가 아닌 worker 쓰레드에서 처리함으로써, 성능향상을 도모했습니다.
- offscreencanvas api를 사용해 캔버스 렌더링을 worker쓰레드에서 처리하도록 했습니다. (기존 main 쓰레드에 있는 캔버스에 반영됩니다.)
- 백그라운드 캔버스에서 먼저 필요한 내용을 그리고 presentation 캔버스에 덮어쓰기 하는 방식을 좀 더 효율적인 성능 개선을 진행했습니다.
  - 백그라운드 캔버스에 대한 렌더링을 worker thread에서 진행합니다.
  - 그려진 캔버스를 transferToImageBitmap 메서드를 이용해 반환합니다.
  - 이를 통해 백그라운드의 이미지를 모두 복사하는 것이 아닌 버퍼에 대한 포인터만을 참조함으로써, 추가적인 성능향상이 기대됩니다.

### 실습환경

- 실습은 CRA 환경에서 진행되었습니다.
- web worker의 경우 웹팩 버전(5이상 여부) 및 CRA에 따라 추가적인 설정이 필요합니다.

- **웹팩 5이상**의 경우 can use without `[worker-loader](https://github.com/webpack-contrib/worker-loader)`

```tsx
// src/index.js
const worker = new Worker(new URL("./deep-thought.js", import.meta.url));
worker.postMessage({
  question:
    "The Answer to the Ultimate Question of Life, The Universe, and Everything.",
});
worker.onmessage = ({ data: { answer } }) => {
  console.log(answer);
};

// src/deep-thought.js
self.onmessage = ({ data: { question } }) => {
  self.postMessage({
    answer: 42,
  });
};
Node.js;
```

[Web Workers | webpack](https://webpack.js.org/guides/web-workers/)

- **웹팩 4버전**의 경우 worker-loader 설정 필요

```tsx
// webpack.config.js
module.exports = {
  module: {
    rules: [
      {
        test: /\.worker\.js$/,
        use: { loader: "worker-loader" },
      },
    ],
  },
};

// App.js
import Worker from "./file.worker.js";

const worker = new Worker();

worker.postMessage({ a: 1 });
worker.onmessage = function (event) {};

worker.addEventListener("message", function (event) {});
```

[worker-loader | webpack](https://v4.webpack.js.org/loaders/worker-loader/)

**CRA**로 프로젝트를 생성한 경우 config-overrides.js 파일을 생성 후 설정

```tsx
// config-overrides.js
const WorkerPlugin = require("worker-plugin");

module.exports = function override(config, env) {
  config.plugins.push(new WorkerPlugin());
  return config;
};
```

```tsx
...
useEffect(() => {
	const canvas = canvasRef.current;

  const offscreen = canvas.transferControlToOffscreen();
  const worker = new Worker('../../workers/offscreencanvas.js', {
    type: 'module',
  });
  const init = () => {
    worker.postMessage({ canvas: offscreen }, [offscreen]);
  };
  init();
}, [])
...
```

```tsx
// offscreencanvas.js
const worker = self;

worker.onmessage = (e) => {
  const { canvas } = e.data;
  console.log(e.data, canvas);
};
```
- 타입스크립트의 경우 별도의 패키지 설치가 필요합니다.
   - `yarn add @types/offscreencanvas`
   - `npm i @types/offscreencanvas`
