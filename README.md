##  前端组件库建设的示例 

技术栈 `vite3、vitepress、vitest、vue3、tsx`

一个使用 pnpm 建立的 `monorepo` 工程

一个 `card` 组件示例 及 测试示例

使用 vitepress 生成组件文档, `githup Actions 结合 githup pages` 自动部署 [猛击预览文档](https://link.juejin.cn?target=https%3A%2F%2Fvaebe.github.io%2Fccui%2F)

微脚手架用于: `生成组件开发模版`、`组件文档模版`、`生成组件主题文件`、`打包发布npm`

组件库组件主题切换实现

## 文章的整理思路

先创建一个 `monorepo` 项目包含lint工具链保证项目的代码的质量及代码风格。

往里添加一个存放组件库相关的目录`ccui`,存放组件的目录 `ui`,文档目录 `docs`。

然后增加脚手架目录 `cli` 完善项目解决重复劳动。

打包发布项目到 `npm`

使用 `githup Actions 结合 githup pages` 自动部署文档

因为文章内容比较多，作者也不是一次写完的所以难免有所疏漏，望海涵！

# 开发环境

| 系统  | node     | pnpm | typescript | vite     | vitest    |
| ----- | -------- | ---- | ---------- | -------- | --------- |
| macos | v16.13.2 | 7.x  | "^4.6.4"   | "^3.0.0" | "^0.18.1" |

# 创建 monorepo 工程

新建 `cc-ui` 文件夹，进入 `cc-ui` 文件夹内

执行 `git init` 创建一个 .git 文件，然后新建一个仓库并关联。

执行 `pnpm init` 创建 `package.json` 文件

## 创建 .npmrc 文件

写入如下内容 `enable-pre-post-scripts=true` 执行npm 的前置、后置钩子脚本

## 创建 pnpm-workspace.yaml

写入如下内容指定 `packages` 为工作目录

```vbnet
packages:
  - 'packages/**'
复制代码
```

[pnpm-workspace.yaml 工作空间](https://link.juejin.cn?target=https%3A%2F%2Fpnpm.io%2Fzh%2Fpnpm-workspace_yaml) 定义工作空间的根目录(也就是monorepo所有子项目存放的目录)。

同级创建 `packages` 文件夹，`packages` 就是项目存放的目录

## 安装`lint`工具

**偷偷告诉你其实可以直接拷贝示例仓库的代码方便又快捷**

安装lint工具的内容其实有点多分了一篇文章使这篇文章可以更加专注介绍组件库的建设

lint 工具链的安装可以查看[夏天需要吃西瓜,项目也需要Lint工具链！](https://juejin.cn/post/7123516611528654884) 不过需要做一些修改

## 修改 `.eslintignore`

```css
node_modules/*
packages/**/node_modules/*
packages/**/dist/*
packages/**/build/*
packages/**/lib/*
packages/**/src/*.d.ts

packages/**/__tests__/*
**.json
**.svg
复制代码
```

## 修改 `.gitignore`

```bash
.DS_Store
node_modules
dist
dist-ssr

# local env files
.env.local
.env.*.local

# Log files
npm-debug.log*
yarn-debug.log*
yarn-error.log*
pnpm-debug.log*

# Editor directories and files
.idea
.vscode
*.suo
*.ntvs*
*.njsproj
*.sln
*.sw?
*.zip

# 忽略packages下相关文件
packages/ccui/ui/vue-ccui.ts
packages/ccui/ui/theme/theme.scss
packages/ccui/ui/theme/darkTheme.css
packages/ccui/docs/.vitepress/config/sidebar.ts
packages/ccui/docs/.vitepress/config/enSidebar.ts
packages/ccui/docs/.vitepress/config.ts.js
packages/ccui/build

**CHANGELOG.md
# test
packages/ccui/coverage
packages/ccui/ui/**/__snapshots__
.pnpm-debug.log
复制代码
```

## 修改 `.ls-lint.yml`

```yml
ls:
  packages/*:
    .dir: kebab-case | regex:__[a-z0-9]+__
    .scss: kebab-case # 对所有 scss 文件使用 kebab-case 形式
    .vue: kebab-case | pascalcase  # vue 组件推荐 大写字母开头 额外配置 pascalcase
    .js: kebab-case
    .ts: kebab-case
    .tsx: kebab-case
    .route.ts: kebab-case
    .type.ts: kebab-case
    .test.ts: kebab-case
    .config.ts: kebab-case

ignore:
  # ccui
  - packages/ccui/node_modules
  - packages/ccui/docs
  - packages/ccui/ui/theme
  - packages/cli/node_modules
  - packages/build
复制代码
```

## `package.json` 的相关命令解释

```json
// 用于本地启动项目进行开发
"dev": "pnpm --filter vue3-ccui dev",

// 项目文档打包 
// 因为忽略了一些vitepress的配置文件先执行生成生成文件命令
"docs:build": "pnpm --filter vue3-ccui predev -- -e prod && pnpm --filter vue3-ccui docs:build",

// 执行 eslint 与 stylelint
"lint": "pnpm run lint:script && pnpm run lint:style",

// 执行 eslint 代码检查
"lint:script": "eslint --ext "packages/**/*.{vue,js,jsx,ts,tsx}" --fix --quiet ./",

// 执行 stylelint 进行 css,scss 样式检查
"lint:style": "stylelint --fix "packages/**/*.{css,scss}"",

// 安装 husky
"postinstall": "husky install",
复制代码
```

## 最后你的目录应该是这样

![截屏2022-07-20 上午11.16.59.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/804eb4a4e74849a593e0aef6f51f29e4~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

- 这样的话基础工程就创建完毕了，下边就开始组件库项目的创建

# 创建组件库目录

进入 `packages` 目录下执行 `pnpm create vite` 创建名为 `ccui` 的项目[vite 官网](https://link.juejin.cn?target=https%3A%2F%2Fcn.vitejs.dev%2Fguide%2F%23scaffolding-your-first-vite-project)

创建完成后如下图所示 ![截屏2022-07-20 上午11.39.27.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9b9d4e4a5e9e4b8c93f1240547c50dd4~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

进入 `packages/ccui` 目录，删除 `.vscode、public、src、index.html、.gitignore` 文件 ![截屏2022-07-20 上午11.44.59.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/67a001a054394bb09cb55e8fd101683a~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

记得删除 `package.json` 中 `"type": "module"`

## 修改 `/packages/ccui/package.json` 如下

```perl
{
  "name": "vue3-ccui",
  "version": "1.0.6",
  "license": "MIT",
  "description": "vue3-ccui components based on Vite and Vue3",
  "keywords": [
    "frontend",
    "typescript",
    "ui-component",
    "components",
    "vue-components",
    "vue",
    "vue3",
    "vite",
    "jsx",
    "vue3-ccui"
  ],
  "homepage": "https://vaebe.github.io/ccui/",
  "repository": {
    "type": "git",
    "url": "https://github.com/vaebe/ccui.git"
  },
  "main": "vue-ccui.umd.js",
  "module": "vue-ccui.es.js",
  "style": "style.css",
  "scripts": {
    "dev": "vitepress dev docs",
    "docs:build": "vitepress build docs",
    "docs:serve": "vitepress serve docs",
    "register:components": "vitepress-rc",
    "test": "vitest",
    "coverage": "vitest run --coverage"
  },
  "dependencies": {
    "vitepress-theme-demoblock": "^1.4.2",
    "vue": "^3.2.37"
  },
  "devDependencies": {
    "vite-svg-loader": "^3.4.0",
    "@types/node": "^18.0.5",
    "@vitejs/plugin-vue-jsx": "^2.0.0",
    "@vue/test-utils": "^2.0.2",
    "jsdom": "^20.0.0",
    "@vitejs/plugin-vue": "^3.0.0",
    "c8": "^7.11.3",
    "typescript": "^4.6.4",
    "vite": "^3.0.0",
    "vitest": "^0.18.1",
    "sass": "^1.53.0",
    "vue-tsc": "^0.38.4",
    "vitepress": "1.0.0-alpha.4"
  }
}
复制代码
```

执行 `pnpm i` 安装依赖

这里移除了一个命令 `"predev": "node ../cli/index.js create -t ccui --ignore-parse-error && node ../cli/index.js generate:theme",`作用是 启动项目时自动生成一些文件 ![截屏2022-07-20 下午5.18.12.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6497e00975834cba97ee5350168a7efe~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

## 更改 `vite.config.ts` 配置 `jsx`

```javascript
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

// jsx 依赖
import vueJsx from '@vitejs/plugin-vue-jsx';


export default defineConfig({
  plugins: [vue(),vueJsx()]
})
复制代码
```

## 在 `packages/ccui` 目录下创建如下所示文件及目录

在 `packages/ccui` 创建 `ui` 目录，用于存放组件代码及文档代码，后边还会有一个微脚手架，所以用目录隔离开。

```js
├── ui // 组件库目录 
│   ├── card // card 组件目录
│   │   ├── index.ts // 组件出口
│   │   ├── src
│   │   │   ├── card-types.ts // 组件类型相关 如props
│   │   │   ├── card.scss // 组件样式
│   │   │   └── card.tsx // 组件
│   │   └── test
│   │       └── card.test.ts // 组件测试文件
│   └── shared // 公共目录
│       ├── hooks
│       │   └── use-namespace.ts // class bem
复制代码
```

![截屏2022-07-20 下午2.01.25.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bee1e2dbfb524967a5bf357ae39821e3~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

# 开发 Card 组件

来写一个基本 Card 组件，组件代码不是很难直接贴代码了

Card 组件样式分为三种 `'always' | 'hover' | 'never'` 主要用到了 `slot` 来接收自定义 header 及 组件的默认内容。

## card-types.ts

定义card组件所使用到的类型。

```ts
import type { PropType, ExtractPropTypes } from 'vue';

type ShadowType = PropType<'always' | 'hover' | 'never'>;

export const cardProps = {
  shadow: {
    type: String as ShadowType,
    default: 'always'
  },
  header: {
    type: String,
    default: ''
  },
  bodyStyle: {
    type: Object,
    default: () => {
      return { padding: '20px' };
    }
  }
} as const;

// 获取 props 类型
export type CardProps = ExtractPropTypes<typeof cardProps>;
复制代码
```

## `shared/hooks/use-namespace.ts`

用于统一管理class的命名空间后期可很方便的更改。具体使用看下边 `card.tsx`

```ts
export type UseNamespace = {
  b: () => string;
  e: (el: string) => string;
  m: (mo: string) => string;
  em: (el: string, mo: string) => string;
};

function createBem(
  namespace: string,
  element?: string,
  modifier?: string
): string {
  let cls = namespace;
  if (element) {
    cls += `__${element}`;
  }
  if (modifier) {
    cls += `--${modifier}`;
  }
  return cls;
}

/**
 * useNamespace
 *
 * @param block current block name
 * @param needDot Do you need a dot prefix (defalut: false)
 * @returns UseNamespace
 */
export function useNamespace(block: string, needDot = false): UseNamespace {
  // .ccui 项目 class 的命名空间
  const namespace = needDot ? `.ccui-${block}` : `ccui-${block}`;
  const b = () => createBem(namespace);
  const e = (element: string) => (element ? createBem(namespace, element) : '');
  const m = (modifier: string) =>
    modifier ? createBem(namespace, '', modifier) : '';
  const em = (element: string, modifier: string) =>
    element && modifier ? createBem(namespace, element, modifier) : '';
  return {
    b,
    e,
    m,
    em
  };
}
复制代码
```

## card.tsx

Card 组件的实现

```tsx
import { defineComponent, computed } from 'vue';
import { cardProps, CardProps } from './card-types';
import './card.scss';
import { useNamespace } from '../../shared/hooks/use-namespace';

export default defineComponent({
  name: 'CCard',
  props: cardProps,
  setup(props: CardProps, { slots }) {
    const ns = useNamespace('card');

    // ccui-card ccui-card__nse ccui-card--nsm ccui-card__em--open
    console.log(ns.b(), ns.e('nse'), ns.m('nsm'), ns.em('em', 'open'));

    const boxClass = `${ns.b()} ${ns.m(props.shadow)}-shadow`;

    const isHeader = computed(() => {
      return props.header || slots.header;
    });

    return () => (
      <div class={boxClass}>
        <div class={ns.m('header')} v-show={isHeader}>
          {(slots.header && slots.header()) || props.header}
        </div>
        <div class={ns.m('body')} style={props.bodyStyle}>
          {slots.default && slots.default()}
        </div>
      </div>
    );
  }
});
复制代码
```

## card.scss

组件的样式，这里的 `$cls-prefix` 是命名空间也是为了可以方便的更改。

```scss
// 后边组件主题环节会通过此文件替换 下边的两个变量
// @import '../../style-var/index.scss';  
$cls-prefix: 'ccui';
$ccui-global-bg: "#ffffff";

.#{$cls-prefix}-card {
  overflow: hidden;
  color: #303133;
  background: $ccui-global-bg;
  border: 1px solid #ebeef5;
  border-radius: 4px;
  transition: 0.3s;

  &--header {
    box-sizing: border-box;
    padding: 18px 20px;
    border-bottom: 1px solid #ebeef5;
  }
}

.#{$cls-prefix}-card--always-shadow {
  box-shadow: 0 2px 12px 0 rgb(0 0 0 / 10%);
}

.#{$cls-prefix}-card--hover-shadow {
  &:hover,
  &:focus {
    box-shadow: 0 2px 12px 0 rgb(0 0 0 / 10%);
  }
}

.#{$cls-prefix}-card--never-shadow {
  box-shadow: none;
}
复制代码
```

## index.ts

用于组件的注册、导出

```ts
import type { App } from 'vue';
import Card from './src/card';

// 作为插件引入
Card.install = function (app: App): void {
  app.component(Card.name, Card);
};

// 按需
export { Card };

// 内部统一注册
export default {
  title: 'Card 卡片',
  category: '数据展示',
  status: '100%',
  install(app: App): void {
    app.component(Card.name, Card);
  }
};
复制代码
```

# Card 组件测试

上边的 `package.json` 文件已经配置了 [vitest](https://link.juejin.cn?target=https%3A%2F%2Fcn.vitest.dev%2Fguide%2F) 及相关依赖，也已经创建了测试目录及文件 `test/card.test.ts`。

## 修改 `packages/ccui/vite.config.ts` 配置文件

这里的 `defineConfig` 没有声明 `test` 字段所以会飘红。

也可以使用 `import { defineConfig } from 'vitest/config';` 声明导出貌似有些问题同样是会有类型错误提示。🤪

```ts
import { defineConfig } from 'vite';

// jsx 依赖
import vueJsx from '@vitejs/plugin-vue-jsx';

export default defineConfig({
  plugins: [vueJsx()],
  test: {
    globals: true,
    environment: 'jsdom',
    transformMode: {
      web: [/.[tj]sx$/]
    }
  }
});
复制代码
```

## card.test.ts

Card 组件的测试文件

```ts
// 引入测试相关依赖
import { shallowMount } from '@vue/test-utils';
import { expect, test, it } from 'vitest';

// 引入 card
import { Card } from '../index';
import { useNamespace } from '../../shared/hooks/use-namespace';

const ns = useNamespace('card');

// 创建测试
test('mount component', () => {
  // 创建一个包装后的测试组件
  const wrapper = shallowMount(Card, {
    props: {
      shadow: 'hover'
    }
  });

  //  测试组件是否生成成功 
  it('Card demo has created successfully', async () => {
    expect(wrapper).toBeTruthy();
  });

  // 测试组件是否有 .ccui-card 属性的元素
  it('Card should have content', () => {
    const container = wrapper.find(ns.b());
    expect(container.exists()).toBeTruthy();
  });

  it('Card should have header', () => {
    const container = wrapper.find(ns.m('header'));
    expect(container.exists()).toBeTruthy();
  });
});
复制代码
```

## 执行测试

`pnpm test` 与 `pnpm coverage` 上边的 `package.json` 已经做了配置

在 `packages/ccui` 目录下执行 `pnpm test` ![截屏2022-07-20 下午5.35.52.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5641b83a3f6545ed95ff2843ea11de0d~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

在 `packages/ccui` 目录下执行 `pnpm coverage` 获取测试覆盖率 ![截屏2022-07-20 下午5.38.10.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ef27285cd8784b2586cc6996bd2c4742~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

## 到这里一个不是很完美的 card 组件就完成了

- 接下来就是组件的文档了🤔

# 组件文档

- 文档采用 `vitepress` 一家人和和美美！

## 创建文档目录

- 在 `/packages/ccui/` 创建组件名为 `docs` 目录
- 将上方的 `vite.config.ts` 拷贝一份放到 `/packages/ccui/docs`中

```js
├── components // 组件文档
│   └── card // card组件的文档
│       └── index.md
├── index.md // vitepress 默认页面
├── introduce.md // 介绍 用来介绍项目
├── public // 公共目录
│   └── logo.svg // logo
└── vite.config.ts // vitepress 配置
复制代码
```

## 创建 `index.md` 文件

`vitepress` 默认展示的首页

```yaml
---
layout: home
hero:
  name: vue3-ccui
  text: 一个使用 vue3、tsx 的组件库示例
  tagline: 看过星辰大海，才明白自己渺小如沙
  image:
    src: /logo.svg
    alt: cc ui
  actions:
  - theme: brand
    text: 开始
    link: /introduce
features:
  - icon: 🛠️
    title: 丰富的功能
    details: 内置微脚手架，专注于组件的开发。

  - icon: ⚡️
    title: 快速
    details: vite3不只是快。
 
  - icon: 💡
    title: 技术栈
    details: vite3、vitepress、vitest、vue3、tsx。
---
复制代码
```

## 创建 public 目录

目前是将 `logo.svg` 文件放了进去暂无其他用途

## 创建 .vitepress 目录

目录结构如下，一些配置类的东西组合起来麻烦了点！

```js
├── config  
│   ├── enSidebar.ts // 后边通过 微脚手架生成
│   ├── index.ts
│   ├── markdown.ts
│   ├── nav.ts // header 导航展示内容的配置
│   └── sidebar.ts // 后边通过 微脚手架生成
├── config.ts
└── theme // 关于主题样式的一些配置
    ├── index.ts
    ├── register-components.js // 注册插件
    └── styles // vitepress 样式
        ├── demo-block.scss // 自定义 vitepress-theme-demoblock的样式
        ├── index.scss // 出口文件
        └── vars.css // vitepress 样式

复制代码
```

### `config/enSidebar.ts`

```ts
// 英文侧边栏 暂时不用 建个空目录
export default {}
复制代码
```

### `config/markdown.ts`

注册 `vitepress-theme-demoblock` 这个插件，类似与 `element-ui`的组件代码展示。

```ts
const markdown = {
  config: (md) => {
    const { demoBlockPlugin } = require('vitepress-theme-demoblock')
    md.use(demoBlockPlugin, {
      cssPreprocessor: 'scss'
    })
  }
}
export default markdown
复制代码
```

![截屏2022-07-24 下午1.11.28.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/74963c6b81424d1389386019b0884aee~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

### `config/nav.ts`

nav 头部展示内容的配置

```ts
export default [
  { text: 'code仓库', link: 'https://github.com/vaebe/ccui.git' }
]
复制代码
```

![截屏2022-07-24 下午1.15.13.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b50023423ebe42ccb51d0f0dc98032fe~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

### `config/sidebar.ts`

侧边栏的配置

**原来子项是 `children` 现在是 `items` 需要注意**

```ts
export default {
  "/": [
    {
      text: "快速开始",
      link: "/",
      items: [
        {
          text: "简介",
          link: "/introduce",
        },
      ],
    },
    {
      text: "数据展示",
      items: [
        {
          text: "Card 卡片",
          link: "/components/card/",
          status: "100%",
        },
      ],
    },
  ],
};
复制代码
```

![截屏2022-07-24 下午1.38.02.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/67600ab594f445fdaa4b5376a7cb4acb~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

### `config/index.ts`

将 `config` 的配置集中导出

```ts
import nav from './nav';
import markdown from './markdown';
import sidebar from './sidebar';

export default {
  lang: 'en-US',
  title: 'vue-cc-ui',
  description: ' vue-cc-ui 组件库',
  lastUpdated: true,
  head: [['link', { rel: 'icon', type: 'image/svg+xml', href: '/logo.svg' }]],
  markdown,
  themeConfig: {
    sidebar,
    nav,
    logo: '/logo.svg'
  }
};
复制代码
```

### `theme/register-components.js`

用于注册自定义组件

```ts
import Demo from 'vitepress-theme-demoblock/components/Demo.vue'
import DemoBlock from 'vitepress-theme-demoblock/components/DemoBlock.vue'
export function registerComponents(app) {
  app.component('Demo', Demo)
  app.component('DemoBlock', DemoBlock)
}
复制代码
```

### `theme/styles/demo-block.scss`

用于修改覆盖 `vitepress-theme-demoblock` 组件的样式使其可以跟随 `vitepress` 的主题进行变化。

```css
.demo-block {
  border: solid 1px var(--vp-c-divider-light) !important;

  &.hover {
    box-shadow: none !important;
  }

  .source {
    overflow: unset !important;

    .demo-spacing {
      & > * {
        margin: 0 8px 8px 0;

        &:last-child {
          margin-right: 0;
        }
      }

      &:last-child {
        & > * {
          margin-bottom: 0;
        }
      }
    }
  }
}

.demo-block-control.is-fixed {
  border-right: solid 1px var(--vp-c-divider-light) !important;
}

.demo-block-control {
  background-color: var(--vp-c-brand-dimm) !important;
  border-top: solid 1px var(--vp-c-divider-light) !important;
  color: var(--vp-c-text-2) !important;

  &:hover {
    color: var(--vp-c-text-1) !important;
  }

  .control-button {
    color: var(--vp-c-text-1) !important;
  }
}

.meta {
  border-top: solid 1px var(--vp-c-divider-light) !important;
  background-color: var(--vp-c-brand-dimm) !important;

  .description {
    border: solid 1px var(--vp-c-divider-light) !important;
    color: var(--vp-c-text-1) !important;
    background-color: var(--vp-c-bg) !important;
  }
}

[class^='version-tag'] {
  display: inline-block;
  padding: 0 4px;
  line-height: 20px;
  color: #ffffff;
  border-radius: 4px;
}

.version-tag-1 {
  background-color: #3dcca6;
}

.version-tag-2 {
  background-color: #f66f6a;
}


// doc code 文档块不显示 css js
.vp-doc [class~='language-css']:before {
  display: none;
}

.vp-doc [class~='language-javascript']:before {
  display: none;
}
复制代码
```

#### `theme/styles/vars.css`

这个文件是从 `vite githup` 文档拷过来的🤪现成的，因为我觉得首页标题那个渐变很好看。

```css
/**
 * Colors
 * -------------------------------------------------------------------------- */

:root {
    --vp-c-brand: #646cff;
    --vp-c-brand-light: #747bff;
    --vp-c-brand-lighter: #9499ff;
    --vp-c-brand-lightest: #bcc0ff;
    --vp-c-brand-dark: #535bf2;
    --vp-c-brand-darker: #454ce1;
    --vp-c-brand-dimm: rgba(100, 108, 255, 0.08);
}

/**
 * Component: Button
 * -------------------------------------------------------------------------- */

:root {
    --vp-button-brand-border: var(--vp-c-brand-light);
    --vp-button-brand-text: var(--vp-c-text-dark-1);
    --vp-button-brand-bg: var(--vp-c-brand);
    --vp-button-brand-hover-border: var(--vp-c-brand-light);
    --vp-button-brand-hover-text: var(--vp-c-text-dark-1);
    --vp-button-brand-hover-bg: var(--vp-c-brand-light);
    --vp-button-brand-active-border: var(--vp-c-brand-light);
    --vp-button-brand-active-text: var(--vp-c-text-dark-1);
    --vp-button-brand-active-bg: var(--vp-button-brand-bg);
}

/**
 * Component: Home
 * -------------------------------------------------------------------------- */

:root {
    --vp-home-hero-name-color: transparent;
    --vp-home-hero-name-background: -webkit-linear-gradient(
            120deg,
            #bd34fe 30%,
            #41d1ff
    );

    --vp-home-hero-image-background-image: linear-gradient(
            -45deg,
            #f56a92 50%,
            #61c4ec 50%
    );
    --vp-home-hero-image-filter: blur(40px);
}

@media (min-width: 640px) {
    :root {
        --vp-home-hero-image-filter: blur(56px);
    }
}

@media (min-width: 960px) {
    :root {
        --vp-home-hero-image-filter: blur(72px);
    }
}

/**
 * Component: Custom Block
 * -------------------------------------------------------------------------- */

:root {
    --vp-custom-block-tip-border: var(--vp-c-brand);
    --vp-custom-block-tip-text: var(--vp-c-brand-darker);
    --vp-custom-block-tip-bg: var(--vp-c-brand-dimm);
}

.dark {
    --vp-custom-block-tip-border: var(--vp-c-brand);
    --vp-custom-block-tip-text: var(--vp-c-brand-lightest);
    --vp-custom-block-tip-bg: var(--vp-c-brand-dimm);
}

/**
 * Component: Algolia
 * -------------------------------------------------------------------------- */

.DocSearch {
    --docsearch-primary-color: var(--vp-c-brand) !important;
}

/**
 * VitePress: Custom fix
 * -------------------------------------------------------------------------- */

/*
  Use lighter colors for links in dark mode for a11y.
  Also specify some classes twice to have higher specificity
  over scoped class data attribute.
*/
.dark .vp-doc a,
.dark .vp-doc a > code,
.dark .VPNavBarMenuLink.VPNavBarMenuLink:hover,
.dark .VPNavBarMenuLink.VPNavBarMenuLink.active,
.dark .link.link:hover,
.dark .link.link.active,
.dark .edit-link-button.edit-link-button,
.dark .pager-link .title {
    color: var(--vp-c-brand-lighter);
}

.dark .vp-doc a:hover,
.dark .vp-doc a > code:hover {
    color: var(--vp-c-brand-lightest);
    opacity: 1;
}

/* Transition by color instead of opacity */
.dark .vp-doc .custom-block a {
    transition: color 0.25s;
}
复制代码
```

### `theme/styles/index.scss`

这个其实就是把css 放在一起导出

```scss
@use "vars.css";
@use "demo-block";
复制代码
```

### `theme/index.ts`

最终配置会合并到这个配置文件，**组件也会在这里进行注册**，注册过的组件才可以在 `.md` 文件中使用

后边组件注册会通过脚手架生成一个文件来注册

```javascript
import Theme from "vitepress/theme";
import "./styles/index.scss";
import "vitepress-theme-demoblock/theme/styles/index.css";
import { registerComponents } from "./register-components.js";
// 引入组件 注册
import CardInstall from "../../../ui/card/index";

export default {
  ...Theme,
  enhanceApp({ app }) {
    //  注册组件
    app.use(CardInstall);
    registerComponents(app);
  },
};
复制代码
```

## components 组件文档目录

因为目前只有一个组件所以只有一个card的目录，与组件一一对应。

```js
├── components
│   └── card
│       └── index.md
复制代码
```

### card/index.md

Card 组件文档 .md

~~~xml
# Card 卡片

+ 将信息聚合在卡片容器中展示。

## 何时使用

+ 基础卡片容器，其中可包含文字，列表，图片，段落，用于概览展示时。

## 基本用法

:::demo Card 示例

```vue

<template>
  <div>
    <c-card style="margin-bottom: 20px" header="这是标题">
      我们终将远行，和过去稚嫩的自己告别。这是一个流行告别的时代，陪你颠沛流离的人越来越少，直至没有。
      我们也要习惯昔日好友的渐行渐远，因为我们终将长大，长大到可以独自一人抵挡风雨。
    </c-card>

    <c-card shadow="hover" style="margin-bottom: 20px" header="这是标题 hover">
      我们终将远行，和过去稚嫩的自己告别。这是一个流行告别的时代，陪你颠沛流离的人越来越少，直至没有。
      我们也要习惯昔日好友的渐行渐远，因为我们终将长大，长大到可以独自一人抵挡风雨。
    </c-card>

    <c-card shadow="never" style="margin-bottom: 20px" header="这是标题 never">
      我们终将远行，和过去稚嫩的自己告别。这是一个流行告别的时代，陪你颠沛流离的人越来越少，直至没有。
      我们也要习惯昔日好友的渐行渐远，因为我们终将长大，长大到可以独自一人抵挡风雨。
    </c-card>
  </div>

</template>

<script>
export default {
  name: 'cardBox'
};
</script>

<style lang="scss" scoped>
</style>
```

:::

## c-card

c-card 参数

| 参数 | 类型 | 默认 | 说明 |
| ---- | ---- | ---- | ---- |
| header | string |  —    | 卡片的标题 可以通过设置 header 来修改标题，也可以通过 slot#header 传入 DOM 节点 |
| body-style |object| '{ padding: '20px' }'| body 的样式  |
| shadow | string | always | 设置阴影显示时机 always / hover / never  |
复制代码
~~~

## 到这里组件文档就结束了

后续组件模版、文档模版都会由微脚手架来创建，只需要执行命令即可！

目录应该是这样的 ![截屏2022-07-20 下午4.10.44.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cfa45b54139a4d289659c1efec8c4945~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

然后运行 `package.json` 中 `dev` 命令 ![截屏2022-07-20 下午4.12.34.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ec6e6f8677884f3f8d7312d4caedb71e~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

查看成果！ ![截屏2022-07-20 下午4.14.21.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/67c9e14384f441f7ad53ad2a786d4b0c~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

![截屏2022-07-20 下午4.15.26.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d996ebbb36714ef5884096d07a7f5f39~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

# 微脚手架

脚手架相关代码来源于 [vue devui](https://link.juejin.cn?target=https%3A%2F%2Fvue-devui.github.io%2Fquick-start%2F)，做了一些修改，考虑到存在 `devui` 相关文案、变量可能有一定不便，去除了 `devui` 相关文案、变量！

相关代码已经上传仓库这里不在一一进行展示，仅对一些代码进行说明,目录是 `packages/cli`.

## 为什么需要微脚手架

解决组件初始化相关文件的重复工作，创建组件的目录及文件格式时相对固定的，可以规范组件的目录。

重复修改公共文件带来的代码冲突，比如`packages/ccui/docs/.vitepress/config/sidebar.ts`,讲这些公共文件使用脚手架进行生成。

打包发布的重复工作。

## 脚手架具备哪些功能

初始化组件、文档文件

自动生成公共文件，如:

```js
packages/ccui/docs/.vitepress/config/sidebar.ts
packages/ccui/ui/vue-ccui.ts
复制代码
```

生成主题变量文件

组件库打包、发布`npm`包

其他的可由`packages/cli/index.js` 入口文件自行探索。

## 目录结构

```js
├── commands // 命令
│   ├── build-nuxt-auto-import.js // 看命名应该是nuxt自动引入
│   ├── build.js // 打包配置
│   ├── code-check.js // 代码检查
│   ├── create.js // 创建组件、生成全部组件注册文件等
│   ├── generate-dts.js // 生成.d.ts文件
│   ├── generate-theme.js // 生成主题配置
│   └── release.js // 发布 主要是拷贝一些文件到打包的目录
├── inquirers // 执行命令询问的过程
│   ├── component.js
│   └── create.js
├── shared
│   ├── constant.js // 常量
│   ├── logger.js // log输入 带颜色的😏
│   └── utils.js // 一些工具函数
├── templates // 组件模版
│   ├── component.js // 用于生成 组件模版、文档模版、测试模版等
│   ├── vitepress-sidebar.js // 用于生成文档侧边栏配置
│   └── vue-ui.js // 用于生成组件导出、注册文件
├── index.js // 入口文件
├── package.json
├── CHANGELOG.md // 自动生成 changelog 的文件
├── README.md // 说明文件
复制代码
```

## 使用

`package.json` 中定义了如下命令

```js
// 打包文档
"build": "pnpm generate:theme && node --max-old-space-size=4096 node_modules/vitepress/bin/vitepress.js build docs && cp public/* docs/.vitepress/dist/assets && cp docs/assets/* docs/.vitepress/dist/assets",

// 打包组件 为 库
"build:lib": "pnpm --filter vue3-ccui predev -- -e prod && pnpm build:components && pnpm changelog && pnpm release",

// 打包组件
"build:components": "node ./index.js build",

// 生成主题相关文件
"generate:theme": "node ./index.js generate:theme",

// 生成.d.ts
"generate:dts": "node ./index.js generate:dts",

// 发布
"release": "node ./index.js release",

// 执行 cli 的创建命令可以选择创建组件或者项目组件的vue-ccui.ts
"cli": "node ./index.js create",

// 执行 cli 创建命令创建组件
"cli:create": "node ./index.js create -t component",

// 打包前执行生成最新的 组件注册文件即 vue-ccui.ts
"prebuild": "node ./index.js create -t ccui --ignore-parse-error",

// 读取 ccui/package.json 的版本来生成日志
"changelog": "conventional-changelog -k '../ccui/package.json' -p angular -i CHANGELOG.md -s"
复制代码
```

## 修改 docs 文档配置

下边的文件生成就覆盖了无需修改。

```js
 packages/ccui/ui/vue-ccui.ts
 packages/ccui/docs/.vitepress/config/enSidebar.ts
 packages/ccui/docs/.vitepress/config/sidebar.ts
复制代码
```

### 修改 `packages/ccui/docs/.vitepress/theme/index.ts` 为

```javascript
import Theme from 'vitepress/theme';
import './styles/index.scss';
import 'vitepress-theme-demoblock/theme/styles/index.css';
import { registerComponents } from './register-components.js';

// 主要变化是这行代码
import vue_ui from '../../../ui/vue-ccui';

export default {
  ...Theme,
  enhanceApp({ app }) {
    app.use(vue_ui);
    registerComponents(app);
  }
};
复制代码
```

## 修改 `ccui` 配置

### 修改 `ccui/package.json`

到这里已经有了脚手架加上启动自动生成文件的命令

```js
"predev": "node ../cli/index.js create -t ccui --ignore-parse-error && node ../cli/index.js generate:theme",
复制代码
```

## 主题

主题写在这里奇奇怪怪的，但是确实是需要用脚手架生成主题文件，这里涉及三个文件。

### 全局css样式 `packages/ccui/ui/style-var`

这里引入了一份默认的主题变量及一个命名空间。

```css
@import '../theme/theme';

$cls-prefix: ccui;
复制代码
```

有了这个以后，也就可以把`packages/ccui/ui/card/src/card.scss`修改一下了，移除声明的 **scss变量** 改为由`packages/ccui/ui/style-var/index`导入。 ![截屏2022-07-20 下午6.08.43.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a4d23e56649e4800b416eefe877b37d2~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

### 主题目录 `packages/ccui/ui/theme`

`themes` 定义了 `light、dark` ts 文件用户生成主题变量。

会根据这两个主题 `ts` 文件使用脚手架生成 `theme.scss、darkTheme.css` 主题变量文件。

```css
`theme.scss` 默认 `scss变量`
$ccui-global-bg: var(--ccui-global-bg, #f3f6f8);
$ccui-global-bg-normal: var(--ccui-global-bg-normal, #ffffff);
$ccui-base-bg: var(--ccui-base-bg, #ffffff);
$ccui-base-bg-dark: var(--ccui-base-bg-dark, #333854);
$ccui-brand: var(--ccui-brand, #5e7ce0);
$ccui-brand-foil: var(--ccui-brand-foil, #859bff);
复制代码
// `darkTheme.css` 深色 `css变量`
.dark{
    --ccui-global-bg: #202124;
    --ccui-global-bg-normal: #202124;
    --ccui-base-bg: #2E2F31;
    --ccui-base-bg-dark: #2e2f31;
    --ccui-brand: #5e7ce0;
}
复制代码
```

### 生成主题变量文件 `packages/cli/commands/generate-theme.js`

```ts
require('esbuild-register');
const path = require('path');
const fs = require('fs-extra');
const logger = require('../shared/logger');
const { CSS_CLASS_PREFIX } = require('../shared/constant');
const lightTheme = require('../../ccui/ui/theme/themes/light.ts').default;
const darkTheme = require('../../ccui/ui/theme/themes/dark.ts').default;

const lightFileStr = Object.entries(lightTheme)
  .map(
    ([key, value]) =>
      `$${CSS_CLASS_PREFIX}-${key}: var(--${CSS_CLASS_PREFIX}-${key}, ${value})`
  )
  .join(';\n');

let darkCssVariablesStr = Object.entries(darkTheme)
  .map(([key, value]) => `--${CSS_CLASS_PREFIX}-${key}: ${value}`)
  .join(';\n');

darkCssVariablesStr = `.dark{
${darkCssVariablesStr}
}`;

exports.generateTheme = async () => {
  const lightThemeFilePath = path.resolve(
    __dirname,
    '../../ccui/ui/theme/theme.scss'
  );
  const darkThemeFilePath = path.resolve(
    __dirname,
    '../../ccui/ui/theme/darkTheme.css'
  );

  try {
    await fs.outputFile(lightThemeFilePath, lightFileStr, 'utf-8');
    logger.success(`生成theme主题文件成功, ${lightThemeFilePath}!`);

    await fs.outputFile(darkThemeFilePath, darkCssVariablesStr, 'utf-8');
    logger.success(`生成 darkTheme css 主题变量成功, ${darkThemeFilePath}!`);
  } catch (err) {
    logger.success('生成主题文件失败！');
  }
};
复制代码
```

`theme.scss` 生成的是`scss变量` 其目的是为了加载 `css 变量` css变量不存在时使用默认的配置。

`darkTheme.css` 生成的是 `css变量` 且在 `.dark class` 内，可以通过为根元素设置 `class类名` 实现主题换肤（elememnt-plus 深色主题切换就是这种方式）。

这样做后边打包发布完成后修改 `packages/ccui/docs/.vitepress/config/index.ts` 的配置可以实现与`vitepress` 一起换肤，因为 `vitepress` 也是修改 `body className` 为 `dark` 只需要引入深色主题文件即可，主要代码如下：

```js
// 发布完 npm 后 https://unpkg.com/ 就会存在发布的文件
head: [
// ... 其他配置
  [
    'link',
    {
      rel: 'stylesheet',
      href: 'https://unpkg.com/vue3-ccui/theme/darkTheme.css'
    }
  ]
],
复制代码
```

## 生成更新日志

[猛击查看 conventional-changelog-cli](https://link.juejin.cn?target=https%3A%2F%2Fwww.npmjs.com%2Fpackage%2Fconventional-changelog-cli) 用来输出git的日志

### 生成规则

符合 `commit` 规范且读取的 `package.json` 中的 `version` 要大于最后一个 `git tag`

比如这里配置读取的是 `ccui/package.json`(默认是就近选择) `version 是 1.0.1` 那么最后一个 `git tag` 就不能超过 `1.0.1`

```less
// [1.0.10] 读取的 `package.json` 中的 `version`
// 1.0.6...1.0.10  输出 “git tag” 1.0.6 到当前版本的1.1.10的日志
// 2022-07-24 生成日期
## [1.0.10](https://github.com/vaebe/ccui/compare/1.0.6...1.0.10) (2022-07-24)
复制代码
```

### 安装依赖

```
pnpm install conventional-changelog-cli -D
复制代码
```

### `package.json  scripts` 中增加

根据最后一个 `tag` 标签匹配“功能”、“修复”、“性能改进”或“重大更改”模式的提交生成更改日志。

```json
"changelog": "conventional-changelog -p angular -i CHANGELOG.md -s"
复制代码
```

生成全部更新日志

```css
conventional-changelog -p angular -i CHANGELOG.md -s -r 0
复制代码
```

![截屏2022-07-24 下午6.39.28.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0c582648bc85464bb90c4530ed20d41e~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

# 


作者：唐诗
链接：https://juejin.cn/post/7124487017588588574
来源：稀土掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。