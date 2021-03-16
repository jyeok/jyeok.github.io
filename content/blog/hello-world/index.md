---
title: gatsby-starter-blog 분석
date: "2021-03-06"
description: "Hello World"
---

기존의 blog starter로부터 `gatsby`에서 오피셜하게 지원하는 [`gatsby-starter-blog`](https://github.com/gatsbyjs/gatsby-starter-blog)로 이주하게 되면서 기존의 스켈레톤 코드를 살펴보았다.

# directory root

## `Prettier`

`.prettierrc`와 `prettierignore`은 [`prettier'](https://prettier.io)의 설정과 관련된 파일들이 들어 있다.

린터로 `eslint`를 사용하므로 [`eslint-config-prettier`](https://github.com/prettier/eslint-config-prettier#installation)를 추가적으로 설치해 주었다.

## `gatsby-browser`

```js
// custom typefaces
import "typeface-montserrat";
import "typeface-merriweather";
// normalize CSS across browsers
import "./src/normalize.css";
// custom CSS styles
import "./src/style.css";

// Highlighting for code blocks
import "prismjs/themes/prism.css";
```

[Gatsby Browser API](https://www.gatsbyjs.com/docs/reference/config-files/gatsby-browser/)를 통해 client side와 interact한다.

## `gatsby-config`

site metadata와 gatsby plugin을 설정하는 파일이다.

### `siteMetadata`

사이트의 메타데이터.

### [plugin](https://www.gatsbyjs.com/docs/reference/config-files/gatsby-config/)

#### [`gatsby-plugin-image`](https://www.gatsbyjs.com/plugins/gatsby-plugin-image/)

정적 이미지의 경우 `StaticImage` 컴포넌트를 사용한다. 파일 경로 혹은 리모트 url을 모두 넘겨줄 수 있는데, 이 파일들은 **build time**에 다운로드되고 로드되므로 파일 이름을 prop처럼 넘겨줄 수 없다. 즉 filename은 local variable 혹은 static string이어야 한다. 기술적인 이유로 parent component의 prop을 `StaticImage`의 `prop`으로 넘겨줄 수 없다. 이를 원한다면 Static이 아니라 Dynamic Image을 만들어야 한다.

#### [`gatsby-source-filesystem`](https://www.gatsbyjs.com/plugins/gatsby-source-filesystem/)

파일로부터 `File` node를 만들어낸다.

#### [`gatsby-transformer-remark`](https://www.gatsbyjs.com/plugins/gatsby-transformer-remark/)

Markdown 파일들을 parse하여 node로 변환한다. [이 링크](https://using-remark.gatsbyjs.org/?__hstc=247646936.e39ee17690d9bb2b23fabf7f247f1f89.1614924405128.1615025114172.1615045675644.7&__hssc=247646936.1.1615045675644&__hsfp=2710090937)에서 다양한 플러그인과 예시를 확인할 수 있다.

#### [`gatsby-transformer-sharp`](https://github.com/lovell/sharp)

[`libvips`](https://github.com/libvips/libvips) 라이브러리를 사용하여 빠른 이미지 프로세싱을 지원하는 플러그인이다. [Lanczos Resampling](https://en.wikipedia.org/wiki/Lanczos_resampling) 기법 사용. `gatsby-transformer-sharp`에서는 이미지를 `ImageSharp` node로 변환한다. 사용을 위해서는 `gatsby-source-filesystem` 등의 source plugin이 필요하다.

#### [`gatsby-plugin-feed`](https://www.gatsbyjs.com/plugins/gatsby-plugin-feed/)

RSS feed를 만들어 주는 플러그인으로 내부에서 [`rss`](https://www.npmjs.com/package/rss#itemoptions) 패키지를 사용한다.

`output`, `query`, `title`을 각각 반드시 포함해야 하며 사용할 때 용도에 맞게 `serialize` 함수를 구현해 주어야 한다. `match`는 optional configuration으로 `pathname`이 RegExp를 만족하는지의 여부를 테스트하여 피드를 필터링할 때 사용된다.

#### [`gatsby-plugin-manifest`](https://www.gatsbyjs.com/plugins/gatsby-plugin-manifest/)

모바일 브라우저에서 'add to home screen` 기능을 사용할 때 app manifest를 제공한다.

#### [`gatsby-plugin-react-helmet`](https://www.gatsbyjs.com/plugins/gatsby-plugin-react-helmet/)

React Helmet은 document head를 컨트롤하기 위해 사용된다. 이는 웹 브라우저들에서도 사용되지만 SEO (Search Engine Optimization)에서도 중요한데, 검색 엔진들은 search result를 정렬할 때 document head 안에 있는 site metadata들을 적극적으로 이용하기 때문이다.

SPA에서는 화면을 이동할 때 마다 서버로부터 전체 페이지 데이터를 받아오는 것이 아니라 필요한 데이터만을 요청한다. 즉 브라우저 단에서 라우팅이 바뀌어도 문서의 title은 처음에 받은 값이 유지된다. 즉 화면이 이동될 때 동적으로 타이틀을 바꾸기 위해서는 `document.title`의 값을 변경해 주어야 한다.

자세한 정보는 [링크](https://jeonghwan-kim.github.io/dev/2020/08/15/react-helmet.html#검색엔진-최적화가-필요할-때)에서..

#### [`gatsby-plugin-gatsby-cloud`](https://www.gatsbyjs.com/plugins/gatsby-plugin-gatsby-cloud/)

`public` folder에 Header과 Redirection을 정의하는 `_headers.json`, `_redirects.json` 파일을 생성한다. 기본값으로는 보안을 위한 헤더만을 정의한다.

## `gatsby-node.js`

Gatsby Node API는 브라우저에 영향을 미치는 Gatsby의 setting이고 이를 customization하거나 extend하기 위해서는 이 파일을 수정한다. 이 파일에 담긴 코드는 build time에 한 번 실행된다. 여기서 페이지들을 동적으로 생성하거나 GraphQL에 노드를 추가하거나 build lifecycle에서 일어나는 이벤트들에 접근할 수 있다. 수정 할 수 있는 API의 리스트들은 [이 링크](https://www.gatsbyjs.com/docs/reference/config-files/gatsby-node/)에서 확인할 수 있다.

Gatsby Node API에서는 첫 파라미터로 [Node API Helper](https://www.gatsbyjs.com/docs/reference/config-files/node-api-helpers)를 담고 있는 객체를 받는다. 어떤 헬퍼들을 사용할 수 있는지는 위 링크에서 확인할 수 있다.

어떤 plugin들은 **async operation**을 수행하는데 이 때 반드시

- `Promise` API를 사용하거나
- `async`/`await` 문법을 사용하거나
- callback을 세 번째 argument로 넘겨야 한다.

이를 통해 Gatsby가 다른 API와의 순서를 고려하여 처리되어야 하는 특정 플러그인들을 제대로 처리할 수 있다.

```js
// Async/await
exports.createPages = async () => {
  // do async work
  const result = await fetchExternalData();
};
// Promise API
exports.createPages = () => {
  return new Promise((resolve, reject) => {
    // do async work
  });
};
// Callback API
exports.createPages = (_, pluginOptions, cb) => {
  // do async work
  cb();
};
```

## [`gatsby-ssr.js`](https://www.gatsbyjs.com/docs/reference/config-files/gatsby-ssr/)

기본 starter에는 정의되어있지 않지만 이 파일은 gatsby가 [서버 사이드 렌더링](https://velopert.com/3425)시 사용할 API들을 정의한다.
