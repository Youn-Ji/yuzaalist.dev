---
emoji: ⚡️
title: Vite 너 뭐 돼?
date: '2022-04-10 23:00:00'
author: 유자
tags: react vite 
categories: featured challenge
---

작년에 회사에서 리뉴얼 프로젝트가 있었다. 기존 서비스에서 UI, 기능을 개선해 유저에게 새롭게 선보이는 것이었다. 프레임워크부터 다 새로 정할 수 있는 상황이었는데 팀장님께서 Vite를 써보자고 하셨다.
당시에 Vite가 뭐지..? 하면서도 깊이 있게 찾아보지 않고 열심히 React 코드만 쳤던 기억이 있다. 

![what is this](../../assets/09_what-is-this-edit.png)

지금이라도 Vite가 어떤 문제를 해결하기 위해 나왔고, 그 문제를 해결하기 위해 어떻게 했는지 알아보려고 한다.

##  해결하고자 한 문제점 

 ES modules를 브라우저에서 사용하지 못하던 시절에 모듈화된 방식으로 자바스크립트를 작성하기 위한 기본(=native) 메카니즘이 없었다. 그래서 어느 브라우저에서나 돌아가는 모듈화된 앱을 만들기 위해 번들링을 하게 되었다.

 webpack, Rollup, Parcel과 같은 다양한 번들링 툴이 나왔고 프론트엔드 개발자들의 DX를 크게 향상시켜주었다.

그러나, 우리가 좀 더 발전된 앱을 만들어 갈수록, 다루는 자바스크립트의 양도 기하급수적으로 증가했다. 규모가 큰 프로젝트에서 수천개의 모듈을 갖고 있는 것은 보통이었다. 

결국 위 툴들에 기반한 자바스크립트는 성능 문제에 부딪히기 시작했다. 개발 서버를 돌리거나, 파일을 수정하고 브라우저에 반영되기까지 꽤 긴 시간이 소요되었다. 

느린 속도는 개발자의 생산성에 큰 영향을 끼친다. Vite는 이것을 해결하고자 했다. 이제는 거의 모든 브라우저에서 ES modules를  지원한다는 것과, compile-to-native 언어 Go로 쓰여진 새로운 번들러 esbuild의 출현을 통해서 말이다.

> 프랑스어로 vite은 quick을 뜻한다.

![vite translate](../../assets/10_vite-translate.png)

## 어떻게 해결했는가

### 느린 서버 시작 시간을 esbuild 와 native ESM을 사용해 해결

개발 서버가 처음 시작될 때, 서브되기 전까지 번들러는 열심히 전체 앱을 빌드한다. 

Vite는 앱의 모듈을 Dependencies와 Source code 두개의 카테고리로 나누어 개발 서버의 시작 시간을 단축시켰다.

Dependencies는 개발하는 동안 자주 변경되지 않는 자바스크립트이다. 수백개의 모듈을 가진 컴포넌트 라이브러리같은 Dependency는 처리 비용이 많이 든다. 또 다양한 라이브러리를 사용하다보면 각 Dependency 마다 사용하는 모듈 포맷(ES modules나 commonJS)이 다를 수도 있다. 

그래서 Vite는 esbuild를 사용해 Dependencies를 미리 번들한다. esbuild는 위에서 작성한 바와 같이 Go로 작성되었고, 자바스크립트를 기반으로 한 번들러보다 10-100배 이상 빠르다. 이로써 처리 비용이 많이 드는 Dependencies를 미리, 좀 더 빨리 해치울 수 있고, 다양한 Dependencies가 쓰는 모듈 포맷을 맞추기 위해 신경쓸 필요가 없다.

Source code는 JSX나 CSS와 같이 변환이 필요한, 자주 수정될 수 있는 자바스크립트이다. 그리고 모든 Source code가 한번에 불러와질 필요는 없다.

Vite는 Source code를 [native ESM](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules)을 사용해 제공한다. 브라우저에게 번들러 작업의 일부를 하게 한 것이다. Vite는 브라우저가 요청할 때만 소스코드를 제공한다. Route를 기반으로 동적 imports가 처리되기 때문에 실제 유저가 보고 있는 화면만 그때 그때 처리된다.

### 느린 업데이트를 native ESM과 HTTP 헤더를 활용해 해결

 업데이트 속도는 앱 크기에 따라 선형적으로 저하된다. 때문에 번들러 기반 빌드 셋업에서 파일이 수정될 때, 전체 번들을 다시 빌드하는 것은 효율적이지 않다.

어떤 번들러에서는, 개발 서버가 메모리에서 번들링을 실행한다. 그래서 파일에 변경점이 있을 때, 모듈 그래프의 한 부분을 비활성화하면 된다. 하지만 이것 역시 전체 번들을 재구성하고 웹페이지를 다시 불러와야 한다. 
번들을 재구성하는데 비용이 많이 들 수 있으며 페이지를 다시 로드하면 앱의 현재 상태가 사라진다. 

이것이 번들러들이 HMR(Hot Module Replacement)를 지원하는 이유이다. (HMR: 다른 페이지에 영향을 주지 않으면서 모듈 자체의 빠른 교체를 가능하게 하는 것)

HMR은 DX를 크게 향상시킨다. 그러나 실제로 Vite 팀은 앱 사이즈가 커질수록 HMR의 업데이트 속도 또한 함께 떨어지는 것을 발견했다. 

그래서 Vite 에서는 HMR이 native ESM 기반으로 실행되게 했다. 파일이 수정됐을때, Vite는 수정된 모듈과 가장 가까운 HMR 바운더리의 연결고리만 비활성화시킨다. 이것으로 앱의 크기와 상관없이 HMR 업데이트를 빠르게 만들어준다. (여기서 정확히 어떤 로직으로 동작하는지 좀 더 찾아봐야할 것 같다.)

또 Vite는 HTTP 헤더를 활용해 전체 페이지 리로드 속도를 향상시켰다. 소스코드 모듈 요청은 `304:Not Modified`를 통해 조건부 처리되며, 디펜던시 모듈 요청은 `Cache-Control: max-age=31536000, immutable` 을 통해 캐시되게 해 한번 캐시되면 서버에 다시 요청하지 않는다. 

## Vite 뭐 되는구나 개발자한테
Vite는 DX 향상을 위한 즉 개발자를 위한 툴이라는 것을 느꼈다. 
실제로 사용해 봤을 때, 로컬에서 매우 빠르게 실행되는 것을 체감했고 수정 사항도 빠릿빠릿하게 반영되었다. 

Production에서는 여전히 번들링 툴을 사용하고 있는데, 그 이유는 ES modules 사용 시에 중첩된 imports로 인한 네트워크 요청이 불필요하게 생길 수 있기 때문이다. Production에서 최적의 성능을 얻기 위해서는 아직까진 tree-shaking, lazy-loading, common chunk splitting을 하는 것이 효과적이라고 한다.

그렇다면 Production에서 번들링할 때 왜 esbuild를 쓰지 않는 것일까? 
그 이유는 esbuild 자체적으로 아직 완벽하지 않기 때문이다. esbuild는 여전히 개발 중인 피쳐가 있는 상황이다. (code-splitting, CSS handling 등)

그리하여 현재 Vite팀은 [Rollup](https://rollupjs.org/guide/en/) 이라는 툴을 Production 번들러로 사용하고 있다. 

## ref
- https://vitejs.dev/guide/why.html




```toc
```




