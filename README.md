<!-- prettier-ignore-start -->
<!-- SOMETHING AUTO-GENERATED BY TOOLS - START -->
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->
<!-- code_chunk_output -->

- [项目运行指南](#项目运行指南)
- [开发本地环境变量](#开发本地环境变量)
- [开发相关插件/工具](#开发相关插件工具)
- [开发规范](#开发规范)
  - [vue](#vue)
    - [【保持单向数据流】](#保持单向数据流)
    - [【慎用全局注册】](#慎用全局注册)
    - [【组件名称】](#组件名称)
    - [【组件中的 CSS】](#组件中的-css)
    - [【统一标签顺序】](#统一标签顺序)
    - [【其它注意事项】](#其它注意事项)
    - [【 !!!其它则严格遵守 vue 官方风格指南】](#a-target_blank-hrefhttpscnvuejsorgv2style-guide其它则严格遵守-vue-官方风格指南a)
  - [vue-router](#vue-router)
  - [vuex](#vuex)
  - [模块复用](#模块复用)
  - [其它杂项](#其它杂项)
  - [代码注释](#代码注释)
  - [工程目录结构](#工程目录结构)
  - [前端部署](#前端部署)
  - [给 UI 的建议](#给-ui-的建议)
  - [填坑 Q 群：901842001](#填坑-q-群901842001)
  
<!-- /code_chunk_output -->
<!-- SOMETHING AUTO-GENERATED BY TOOLS - END -->
<!-- prettier-ignore-end -->

# 项目运行指南

- 安装/更新依赖包：`npm install`
- 运行：
  - 启动为 dev 环境：`npm run serve` 或 `npm start`
  - 打包为 stage 环境：`npm run build:stage`
  - 打包为 prod 环境：`npm run build`
  - 检查并修复源码：`npm run lint`
  - 运行单元测试：`npm run test:unit`

# 开发本地环境变量

- 新建 .env.development.local 来重写部分环境变量，如：
  - 模拟数据：`VUE_APP_MOCK = true`
  - 接口前缀：`VUE_APP_BASEURL_API = /api`
  - 接口地址：`DEV_PROXY_TARGET_API = http://10.25.73.159:8081`
  - ...

# 开发相关插件/工具

- VSCode 相关插件
  - 必要插件
    - `ESLint`
    - `Vetur`
    - `Prettier - Code formatter`
    - `path Autocomplete`
  - 推荐插件
    - `vscode-element-helper` (element-ui 专用)
    - `Debugger for Chrome`
    - `GitLens -- Git supercharged`
- Chrome 相关插件
  - 必要插件
    - `vue-devtools`
  - 推荐插件
    - `json-formatter`
- 其它工具
  - 推荐：`Postman`

# 开发规范

## vue

### 【保持单向数据流】

- 单个组件的数据流

  ```
  props、data/$store、computed (由前面派生)
    ↓
  template/render
    ↓
  用户交互事件、初始化的异步回调
    ↓
  data/$store
  ```

- 组件间的数据流
  - 父向子传递用 props
  - 子向父传递用 vue 内置的自定义事件
  - 父子双向传递用 <a target="_blank" href="https://cn.vuejs.org/v2/guide/components-custom-events.html">v-model</a> 或 <a target="_blank" href="https://cn.vuejs.org/v2/guide/components-custom-events.html">.sync</a>
  - 跨越传递用 vuex（慎用 EventBus）
  - 紧密耦合的祖孙间传递也可以考虑用父组件作为中间运输层
  - 紧密耦合的兄弟间传递也可以考虑用父组件作为中转运输层

### 【慎用全局注册】

- 组件、混入 ... 应使用局部注册

  局部注册可保持清晰的依赖关系，并且 IDE 智能感知更为友好

### 【组件名称】

- 名称大小写

  ```html
  <script>
    import MyComponent from '@/components/MyComponent.vue' // 文件名使用 PascalCase 命名法
    export default {
      name: 'ComponentName', // 必须有 name
      components: { MyComponent },
    }
  </script>

  <template>
    <div>
      <!-- 在 template 中一律使用 kebab-case 方式调用 -->
      <my-component />
      <el-input />
    </div>
  </template>
  ```

- 使用前缀
  - 非业务通用组件使用 Base 前缀
  - <a href="#hash_Ex">扩展/包装第三方开源组件或内部公共库组件</a>使用 Ex 前缀
  - 单例组件使用 The 前缀

### 【组件中的 CSS】

- 使用 <a target="_blank" href="https://vue-loader.vuejs.org/zh/guide/css-modules.html">CSS Modules</a>，基于如下考虑：

  - 不让外部进行样式重写，避免耦合 (与组件通信的唯一方式是 props)
  - 放心使用简短且语义强的 class 名，无需担心命名空间，使可读性更好

- 使用方式

  - 语法

    ```less
    /* 默认为局部 */
    .xxx {
    }

    /* 从局部转到全局 */
    :global(.yyy) {
    }
    // or
    :global {
    }

    /* 从全局转到局部 */
    :local(.zzz) {
    }
    // or
    :local {
    }
    ```

  - 单个组件专属
    ```html
    <style lang="less" module></style>
    ```
  - 多个组件共用 (\*.module.less)
    ```js
    import style from './style.module.less'
    ```

### 【统一标签顺序】

- script --> template --> style，并使用空行分隔

### 【其它注意事项】

- 慎用 this\.\$refs、this\.\$parent、this\.\$root、provide/inject
  - this\.\$refs 一般用在第三方开源组件或内部公共库组件或非常稳定的组件，以调用显式声明的方法
  - 暴露给外部的方法要加 pub 前缀，如：this\.\$refs.pubFocus()
- 尽量不要在 watch 中变更数据，易造成死循环。数据变更应该交给用户交互事件或初始化的异步回调
- 组件中的 data 及 vuex 中的 state 应该可序列化，即不要存 undefined、function 等

### 【 <a target="_blank" href="https://cn.vuejs.org/v2/style-guide/">!!!其它则严格遵守 vue 官方风格指南</a>】

---

## vue-router

- url 定义准则：path 对应视图变化 (使用 kebab-case 命名法)，query 对应数据变化，hash 对应滚动条位置

  `如：/project-list?type=1&search=keyword#position`

- 路由的 name 值 (命名路由) 使用 kebab-case 命名法

- 视图跳转尽量使用声明式

  ```html
  <router-link :to="path | { path, ... }">使用声明式</router-link>

  <!-- 不推荐的做法 -->
  <a @click="$router.push(...)">使用命令式</a>
  ```

- 当组件依赖 \$route 时 (特别是 \$route.params)，要使用<a target="_blank" href="https://router.vuejs.org/zh/guide/essentials/passing-props.html">路由组件传参</a>，与 \$route 解耦

---

## vuex

- 需要由 vuex 管理的数据

  - 组件间共享的响应式数据
  - 组件间需要跨越传递的数据

- getter、mutation、action、module 使用驼峰命名法
- module 应避免嵌套，尽量扁平化
- module 应该启用命名空间 `namespaced: true`

---

## 模块复用

- 避免重复造轮子，多使用成熟的现成工具/类库/组件，如：lodash、qs、url-parse 等
- 模块设计原则：
  - 高内聚低耦合、可扩展
  - 永远不要去改变模块输入的数据，如：函数参数、组件 prop
  - …
- 方法接口的设计

  ```js
  // 参数类型与个数要保持稳定
  // 建议参数不要超过3个，且预留一个 options 对象，以提高扩展性
  // 方法尽量纯净 (纯函数思想)
  export function myMethod1(a, options) {} // 当必选参数只有一个时
  export function myMethod2(a, b, options) {} // 当必选参数只有两个时
  export function myMethod3(options) {} // 当必选参数有两个以上时
  export function myMethod4(options) {} // 当所有参数都是可选时

  // 有时为了提高灵活性，参数类型可以是两重，一重是期望值，另一重是返回期望值的函数 (可带参)
  export function myMethod5(a) {
    a = typeof a === 'function' ? a() : a
  }
  ```

- <span id="hash_Ex">扩展/包装第三方开源组件或内部公共库组件</span>
  - 使用 extends 混入 (相关命名需要加 ex 前缀，防止覆盖)
  - 使用<a target="_blank" href="https://cn.vuejs.org/v2/guide/render-function.html">函数式组件</a>包装

---

## 其它杂项

- IDE 统一使用 VSCode，并统一使用相关插件及配置
- js 变量声明尽量使用 const
- js 变量或对象属性使用驼峰命名法
- js 私有变量或对象私有属性使用 \_ 前缀 (注意: <a target="_blank" href="https://cn.vuejs.org/v2/style-guide">vue 组件属性不要使用 \_ 前缀</a>)

  ```js
  // 表明该变量仅在 createId 方法中使用 (与 createId 方法紧挨着)
  let _count = 0
  const createId = () => `${Date.now()}_${++_count}`

  // 适时使用立即执行函数可以简洁作用域及保护私有变量
  const createId = (() => {
    let count = 0
    return () => `${Date.now()}_${++count}`
  })()
  ```

- 导入模块时不要省略后缀（js 除外），利于 IDE 感知
- 当导入父层及以上目录中的模块时，建议使用'@'别名

  ```js
  // js
  import XxxXxx from '@/components/XxxXxx.vue'
  ```

  ```html
  <!-- template -->
  <img src="@/assets/logo.png" />
  ```

  ```less
  /* style */
  @import '~@/styles/vars.less';
  .xxx {
    background: url('~@/assets/logo.png');
  }
  ```

- **严格遵守 ESLint 语法校验**，警告级别的也要处理 (暂时用不到的代码可以先注释掉)
- css
  - 全局 class 使用 g- 前缀
  - CSS 选择器应避免深嵌套，尽量的扁平化
  - 关键选择器 (最右边) 避免使用通配符 \*

---

## 代码注释

- 文档头部注释

  - 脚本文件、样式文件

    ```js
    /**
     * 说明
     * @author 作者
     */
    ```

  - vue 文件
    ```html
    <!-- 说明 -->
    <!-- @author 作者 -->
    ```

- js 注释 (使用 <a target="_blank" href="https://jsdoc.app/">JSDoc 注释标准</a>，还能帮助 IDE 智能感知)

  - 注释格式

    ```js
    /**
     * 大的区块、文档头部、JSDoc
     */

    /* 一般的区块 */

    // 小的区块、行
    ```

  - <a target="_blank" href="https://jsdoc.app/howto-es2015-modules.html">ES 2015 Modules</a>

    ```js
    /**
     * 说明
     * @author 作者
     * @param {类型} a
     * @param {Object} options
     * @param {类型} options.x
     * @param {类型} options.y
     * @param {类型} [options.z] 可选参数
     */
    export function myMethod(a, options) {}

    /**
     * 类型参考：https://www.tslang.cn/docs/handbook/basic-types.html
     *
     * 基本
     * @param {boolean}
     * @param {number}
     * @param {string}
     * @param {1|2|3}
     * @param {'a'|'b'|'c'}
     *
     * 数组
     * @param {Array}
     * @param {Array<string>}
     *
     * 函数
     * @param {Function}
     * @param {(data)=>void}
     * @param {(data:Array)=>void|boolean}
     *
     * 对象
     * @param {Object}
     * @param {any}
     *
     * 联合
     * @param {number|string}
     * @param {boolean|(()=>boolean)}
     */
    ```

  - <a target="_blank" href="https://jsdoc.app/howto-es2015-classes.html">ES 2015 Classes</a>

  - 待完成的功能
    ```js
    /* TODO 说明 */
    ```

- css 注释

  - 全局样式一定要写注释

    ```less
    /* 说明 */
    .g-class1 {
    }

    /* 说明 */
    .g-class2 {
    }
    ```

- vue template

  - 适当使用注释与空行

    ```html
    <!-- 说明 -->
    <div>block1</div>

    <!-- 说明 -->
    <div>block2</div>
    ```

---

## 工程目录结构

```
|-- .env.development ------------ dev 环境变量
|-- .env.development.local ------ dev 本地环境变量 (被 git 忽略，需手动新建，用来重写部分环境变量)
|-- .env.production-stage ------- stage 环境变量
|-- .env.production ------------- prod 环境变量
|-- .env.test
|-- static-server.js ------------ 静态资源服务 (node 运行)，通常用于预览/检查打包结果
|-- README.html ----------------- 由 README.md 手动生成 (使用 VSCode 插件 Markdown Preview Enhanced)
|-- .vscode --------------------- 统一 VSCode 配置
|-- public
|   |-- favicon.ico
|   |-- index.html
|   |-- libs -------------------- 不支持模块化加载的 ES5 类库 (只能通过全局变量引用)
|-- src
    |-- main.js
    |-- App.vue
    |-- libs -------------------- 支持模块化加载但是无法通过 npm 安装的 ES5 类库
    |-- assets
    |-- styles
    |   |-- global.less
    |   |-- reset.less
    |   |-- vars.less ----------- less 全局变量
    |   |-- xxx.less
    |-- scripts
    |   |-- utils --------------- 通用方法
    |   |-- constants ----------- 常量 (多使用 Object.freeze)
    |   |-- xxx.js
    |   |-- http ---------------- axios 实例
    |       |-- index.js
    |       |-- http.js
    |       |-- createAxios.js
    |       |-- xxx.js
    |-- injects ----------------- vue 全局注册 (慎用)
    |   |-- index.js
    |   |-- $xxx.js
    |   |-- v-xxx.js
    |   |-- mixin_xxx.js
    |   |-- xxx.js
    |-- element-ui
    |   |-- index.js
    |   |-- rewrite ------------- 主题样式复写
    |       |-- index.less
    |       |-- xxx.less
    |-- vant
    |   |-- index.js
    |   |-- vars.less ----------- 内置变量复写
    |   |-- rewrite ------------- 主题样式复写
    |       |-- index.less
    |       |-- xxx.less
    |-- router
    |   |-- index.js
    |   |-- routes.js
    |   |-- registerInterceptor.js
    |-- store
    |   |-- index.js
    |   |-- root.js
    |   |-- xxx.js
    |-- api
    |   |-- xxx.js
    |   |-- mock ---------------- 模拟数据
    |       |-- index.js
    |       |-- createMock.js
    |       |-- xxx.js
    |-- components
    |   |-- BaseXxx.vue --------- 非业务通用组件
    |   |-- TheXxx.vue ---------- 单例组件
    |   |-- ExXxx.vue ----------- 扩展/包装第三方开源组件或内部公共库组件
    |   |-- XxxXxx.vue
    |   |-- ComponentExamples --- 非单例公共组件需要在这里写示例
    |   |   |-- index.vue
    |   |   |-- XxxXxx.vue
    |   |-- SvgIcon ------------- svg-sprite 图标组件
    |   |   |-- index.vue
    |   |   |-- icons
    |   |-- directives ---------- 可复用的自定义指令（局部注册）
    |   |   |-- xxx.js
    |   |-- mixins -------------- 可复用的混入（局部注册）
    |       |-- xxx.js
    |-- views
        |-- Xxx.vue
        |-- Xxx ----------------- 除了 api 和 vuex，其它的专属模块要内聚在同一目录下
            |-- index.vue
            |-- Xxx.vue --------- 子路由组件
            |-- xxx.js
            |-- xxx.module.less
            |-- components ------ 存放私有组件
```

---

## 前端部署

- 跨域处理

  - 使用代理或 <a target="_blank" href="https://www.baidu.com/s?wd=cors跨域">CORS</a>

- history 模式<a target="_blank" href="https://router.vuejs.org/zh/guide/essentials/history-mode.html">路由处理</a>

  - 如果 url 匹配的不是静态资源 (不带后缀的)，则返回 /index.html 页面

- 客户端缓存处理 (配置响应头)

  - 静态资源

    - 不缓存 `/index.html`

      ```
      Cache-Control: no-store
      ```

    - 强缓存 `/static-hash/**/*`

      ```
      Cache-Control: public,max-age=31536000
      ```

    - <a target="_blank" href="https://www.baidu.com/s?wd=http协商缓存">协商缓存</a> (默认)
      ```
      Cache-Control: no-cache
      Etag: xxx
      Last-Modified: xxx
      ```

  - XHR (解决 IE 缓存问题)
    ```
    Cache-Control: no-cache
    ```

- gzip 压缩

  - 静态资源：启用 gzip 压缩 (除了像素型图片)

  - XHR：发给客户端的响应数据超过指定阀值时应启用 gzip 压缩

---

## 给 UI 的建议

- 对于中后台项目，在画 UI 界面时，建议参考前端已选型的开源组件库，并推荐使用开源组件库提供的制图元件/模板，如：<a target="_blank" href="http://element-cn.eleme.io/#/zh-CN/resource">element-ui</a>
- 对于 H5 项目，如：<a target="_blank" href="https://youzan.github.io/vant/#/zh-CN/design">vant</a>

## 填坑 Q 群：901842001

- <a target="_blank" href="https://jq.qq.com/?_wv=1027&k=5soIl0R" title="901842001">点击链接加入群聊</a>
