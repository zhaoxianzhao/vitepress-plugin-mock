# forked from vbenjs/vite-plugin-mock [github](https://github.com/vbenjs/vite-plugin-mock)

# vite-plugin-mock

**English** | [中文](./README.zh_CN.md)

[![npm][npm-img]][npm-url] [![node][node-img]][node-url]

Provide local and prod mocks for vite.

A mock plugin for vite, developed based on mockjs. And support the local environment and production environment at the same time. Connect service middleware is used locally, mockjs is used online

### Install (yarn or npm)

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
yarn add vitepress-plugin-mock -D
# or
npm i vitepress-plugin-mock -D
# or
pnpm add vitepress-plugin-mock -D
```

- /mock/index.ts

```ts
import Mock from 'mockjs'
const { Random } = Mock
interface ResponseTypes {
  url: string
  method: string
  response: (_params: any) => void
}

const tableData: Array<ResponseTypes> = [
  {
    url: '/test/table1',
    method: 'get',
    response: ({ query }) => {
      return {
        code: 200,
        msg: 'success',
        [`data|${query.limit || 10}`]: [
          {
            id: '@increment',
            name: '@cname()',
            age: Random.integer(18, 40),
            address: '@city(true)',
            sex: Random.integer(0, 2),
            createTime: Random.date(),
            type: Random.integer(0, 2),
            status: Random.integer(0, 2),
            'content|1': '<span>@ctitle(12)</span>',
            tag: '@cword(2,4)',
            remark: '@cword(10,30)',
          },
        ],
        total: query.limit ? query.limit * 5 : 50,
      }
    },
  },
]
export default mock
```

- .vitepress\theme\index.ts

```ts
import DefaultTheme from 'vitepress/theme'
import { createProdMockServer } from 'vitepress-plugin-mock/client'
// mock文件
import tableData from './mock/index'
function setupProdMockServer() {
  createProdMockServer([...tableData])
}

export default {
  ...DefaultTheme,
  enhanceApp(ctx: any) {
    DefaultTheme.enhanceApp(ctx)
    setupProdMockServer()
  },
}
```
