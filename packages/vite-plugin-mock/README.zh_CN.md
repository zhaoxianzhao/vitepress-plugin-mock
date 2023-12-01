# forked from vbenjs/vite-plugin-mock [github](https://github.com/vbenjs/vite-plugin-mock)

::: tip
vitepress 内使用 mock 数据；处理打包 Mock.XHR.prototype.\_\_send = Mock.XHR.prototype.send 报错
:::

# vite-plugin-mock

**中文** | [English](./README.md)

[![npm][npm-img]][npm-url] [![node][node-img]][node-url]

提供本地和生产模拟服务。

vite 的数据模拟插件，是基于 vite.js 开发的。 并同时支持本地环境和生产环境。 Connect 服务中间件在本地使用，mockjs 在生产环境中使用。

## Production environment problem description

The current production environment cannot support the acquisition of `headers` and the acquisition of `restful`Url format parameters. So there are those two formats that need to be used in the production environment.

### 安装 (yarn or npm)

**node version:** >=12.0.0

**vite version:** >=2.0.0

```bash
yarn add mockjs
# or
npm i  mockjs -S
# or
pnpm add mockjs
```

and

```bash
yarn add vite-plugin-mock -D
# or
npm i vite-plugin-mock -D
# or
pnpm add vite-plugin-mock -D
```

## 使用

**开发环境**

开发环境是使用 Connect 中间件实现的。

与生产环境不同，您可以在 Google Chrome 控制台中查看网络请求记录

- vite.config.ts 配置

```ts
import DefaultTheme from 'vitepress/theme'
import { createProdMockServer } from 'vite-plugin-mock/client'
// mock文件
import tableArray from './mock/index'
function setupProdMockServer() {
  createProdMockServer([...tableArray])
}

export default {
  ...DefaultTheme,
  enhanceApp(ctx: any) {
    DefaultTheme.enhanceApp(ctx)
    setupProdMockServer()
  },
}
```

`/mock/index`

```ts
// test.ts

import { MockMethod, MockConfig } from 'vite-plugin-mock'
export default [
  {
    url: '/api/get',
    method: 'get',
    response: ({ query }) => {
      return {
        code: 0,
        data: {
          name: 'vben',
        },
      }
    },
  },
  {
    url: '/api/post',
    method: 'post',
    timeout: 2000,
    response: {
      code: 0,
      data: {
        name: 'vben',
      },
    },
  },
  {
    url: '/api/text',
    method: 'post',
    rawResponse: async (req, res) => {
      let reqbody = ''
      await new Promise((resolve) => {
        req.on('data', (chunk) => {
          reqbody += chunk
        })
        req.on('end', () => resolve(undefined))
      })
      res.setHeader('Content-Type', 'text/plain')
      res.statusCode = 200
      res.end(`hello, ${reqbody}`)
    },
  },
] as MockMethod[]

export default function (config: MockConfig) {
  return [
    {
      url: '/api/text',
      method: 'post',
      rawResponse: async (req, res) => {
        let reqbody = ''
        await new Promise((resolve) => {
          req.on('data', (chunk) => {
            reqbody += chunk
          })
          req.on('end', () => resolve(undefined))
        })
        res.setHeader('Content-Type', 'text/plain')
        res.statusCode = 200
        res.end(`hello, ${reqbody}`)
      },
    },
  ]
}
```

### MockMethod

```ts
{
  // 请求地址
  url: string;
  // 请求方式
  method?: MethodType;
  // 设置超时时间
  timeout?: number;
  // 状态吗
  statusCode?:number;
  // 响应数据（JSON）
  response?: ((opt: { [key: string]: string; body: Record<string,any>; query:  Record<string,any>, headers: Record<string, any>; }) => any) | any;
  // 响应（非JSON）
  rawResponse?: (req: IncomingMessage, res: ServerResponse) => void;
}

```

## 在生产环境中的使用

创建`mockProdServer.ts` 文件

```ts
//  mockProdServer.ts
import { createProdMockServer } from 'vite-plugin-mock/client'

// 逐一导入您的mock.ts文件
// 如果使用vite.mock.config.ts，只需直接导入文件
// 可以使用 import.meta.glob功能来进行全部导入
import testModule from '../mock/test'

export function setupProdMockServer() {
  createProdMockServer([...testModule])
}
```

配置 `vite-plugin-mock`

```ts
import { viteMockServe } from 'vite-plugin-mock'

import { UserConfigExport, ConfigEnv } from 'vite'

export default ({ command }: ConfigEnv): UserConfigExport => {
  return {
    plugins: [
      viteMockServe({
        mockPath: 'mock',
        // 根据项目配置。可以配置在.env文件
        enable: true,
      }),
    ],
  }
}
```

### 示例

**运行示例**

```bash
pnpm install

# ts example
cd ./examples/ts-examples

pnpm run serve

# js example

cd ./examples/js-examples

pnpm run serve
```

## 示例项目

[Vben Admin](https://github.com/anncwb/vue-vben-admin)

## 注意事项

- 无法在 mock.ts 文件中使用 node 模块，否则生产环境将失败
- 模拟数据如果用于生产环境，仅适用于某些测试环境。 不要在正式环境中打开它，以避免不必要的错误。 同时，在生产环境中，它可能会影响正常的 Ajax 请求，例如文件上传/下载失败等。

## License

MIT

[npm-img]: https://img.shields.io/npm/v/vite-plugin-mock.svg
[npm-url]: https://npmjs.com/package/vite-plugin-mock
[node-img]: https://img.shields.io/node/v/vite-plugin-mock.svg
[node-url]: https://nodejs.org/en/about/releases/
