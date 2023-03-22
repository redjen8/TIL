---
title: "ECMAScript Module과 Import"
date: 2023-03-22T20:58:51+09:00
draft: false
author: redjen
---

자바스크립트로 웹 페이지를 만들던, node.js 어플리케이션을 만들던 사용하게 되는 자바스크립트의 기능으로는 `import`가 있다.

모든 코드는 한 군데에 작성하는 것보다 기능적인 측면으로 묶어서 관리하기 위해 모듈화를 하게 되는데,
FE 화면 개발을 하다보면 필연적으로 겪게 되는 개선 과제로도 ~ 공통 모듈화가 있다.

공통 모듈화를 진행하게 되면 코드의 재사용성은 물론, 가독성도 올라가므로 꽤 효과적인 방법으로 어지러운 코드를 리팩토링하는 방법인 것이다.

그럼, 이 모듈은 어떻게 관리되고 `import`를 사용해서 어떻게 불러와질까? 

https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import

## import

import 선언은 다른 모듈에 의해 export 된 read-only 라이브 바인딩을 불러오기 위해서 사용된다.

이렇게 불러와진 바인딩들은 '라이브 바인딩'이라고 불리는데
- 바인딩을 export 한 모듈에 의해 업데이트 되지만
- import 하는 모듈에 의해서 재할당 되지는 못하기 때문이다.

소스 파일에서 `import` 선언을 사용하기 위해서는 해당 파일이 먼저 런타임에 모듈로 해석되는지를 확인해야 한다.
- HTML에서는 `<script>` 태그 안에 `type="module"`을 추가해서 모듈화 시킬 수있다.
- 모듈들은 엄격 모드에서는 자동으로 해석된다.

`type="module"`을 사용하지 않는 보다 동적인 `import()`도 있다.

### `import` 선언의 특징

`import` 선언은 다음 특징을 가진다.
- 모듈들 안에서만 이루어질 수 있다.
- 최상위 레벨에서만 가능하다. (블럭 안이나, 함수 안에서는 불가능하다)
- `import` 선언이 모듈이 아닌 컨텍스트를 조우하게 되었을 때에는 `SyntaxError`가 발생한다.
	- `type="module"` 선언이 이루어지지 않은 `<script>`
	- "script"와 "function body"를 파싱 목표로 하는 eval`, `new Function` 등이 에시이다.
- 모듈이 아닌 컨텍스트에서 모듈을 `import`하려면, 동적 import를 사용해야 한다.

`import` 선언들은 구문적으로 엄격하게 설계되어, 모듈이 평가되기 전에 정적으로 분석 / 링크될 수 있다.
- 문자열 리터럴 specifier 이어야 한다.
- 최상위 레벨에서만 실행되어야 한다.
- 모든 바인딩은 식별자여야 한다.

> 이는 모듈을 본질적으로 비동기화 시켜서 최상위 수준에서의 await 기능을 지원하기 위함이다.

### Named import

`my-module` 모듈에서 명시적으로 export 문을 사용해서 내보내어진 `myExport`라는 값이 주어지면, `myExport`는 현재 스코프에 포함된다.

```js
import { myExport } from "/modules/my-module.js";
```

같은 모듈에서 동시에 여러 이름들을 불러올 수 있으며, import 당시에 rename도 가능하다. (`as` 문을 사용하여)

### Default import

Default import는 기본 import 문법으로 불러와지는 경우에 해당한다. 
```js
import myDefault from "/modules/my-module.js";
```

default import는 기본적으로 이름을 명시적으로 지정하지 않기 떄문에, 원하는 어떤 이름이든 붙일 수 있다.

또한 default import는 namespace import나 named import와 같이 사용할 수 있다. 이러한 케이스에서는 default import가 우선적으로 선언되어야 한다.

### Namespace import

다음 코드는 `/modules/my-module.js`에 있는 모든 모듈의 export를 포함하는 `myModule`을 현재 스코프에 추가한다.

```js
import * as myModule from "/modules/my-module.js";
```

여기서 `myModule`은 모든 export를 속성으로 포함하는 네임스페이스 객체를 나타낸다.
상기 코드에서 가져온 모듈에 `doAllTheAmazingThings()` export가 포함된 경우, 다음과 같이 호출할 수 있다.

```js
myModule.doAllTheAmazingThings();
```

### 모듈의 부가적인 요소만 불러오기

전체 모듈의 어떠한 것도 import 하지 않고,  오로지 부가적인 요소만 불러올 수도 있다.

이 경우, 모듈의 global 한 코드만 실행하고, 어떠한 값도 가져오지 않기 때문에 polyfill들에서 많이 사용된다.

```js
import "/modules/my-module.js";
```

이런 기법이 polyfill에서 많이 사용되는 이유는, global 변수를 mutate 하기 때문이다.

## webpack과 module

https://webpack.kr/guides/ecma-script-modules/

webpack에서는 파일이 ECMAScript Module (ESM)인지, 또는 다른 모듈 시스템인지를 자동으로 검사한다.

Node.js의 경우에는 `package.json`파일의 속성을 사용해서 파일의 모듈 유형을 명시적으로 설정할 수 있게 한다.

- `"type": "module"` 선언이 되어 있다면 `package.json` 아래의 모든 파일이 ESM이 된다.
- `"type": "commonjs"` 선언이 되어 있다면 CommonJS 모듈이 된다.

또한 파일의 확장자를 사용해서도 모듈의 유형을 설정할 수 있는데,
- `.mjs`는 ESM이 되도록 강제한다.
- `.cjs`는 CommonJS가 되도록 강제한다.

DataURI들에서 `text/javascript`나 `application/javascript` MIME 타입으로 설정했다면 불러오는 모듈 유형은 자동으로 ESM으로 강제된다.

## 라이브러리 개발 시에 한번 더 생각해야 하는 점

https://toss.tech/article/commonjs-esm-exports-field

공통된 라이브러리를 개발하는 입장에서는 모듈을 어떻게 import 하던 불러온 모듈을 정상적으로 사용할 수 있어야 한다. 때문에 개발 할 때 ESM과 CommonJS 모듈을 둘 다 염두에 두고 개발해야 한다.

큰 차이점으로는 CJS Module loader가 동기적으로 작동하는 반면, ESM Module loader는 비동기적으로 작동하는 차이가 있다.

> 때문에 ESM에서 CJS를 import 할 수 있지만, CJS에서 ESM을 import할 수는 없다. CJS는 최상위 레벨에서의 await을 지원하지 않기 때문이다.


또 하나의 차이점으로는.. CJS는 tree shaking이 어려운 반면 ESM은 tree shaking이 용이하다는 면이 있다고 한다.
- CJS는 빌드 타임에 정적으로 모듈을 분석하기 어렵다. 런타임에서만 모듈의 관계를 해석한다.
- ESM은 정적인 구조로 모듈끼리 의존하도록 강제한다. 때문에 빌드 타임에 정적으로 모듈 간 의존 관계를 파악할 수 있기 때문에 tree shaking이 용이하다.



