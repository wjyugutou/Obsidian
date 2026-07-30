### The "from" argument must be of type string. Received undefined

#### 解决

| 解法	| 打在哪 |	性质 |
| -- | -- | -- |
| 降 Node 版本 |	触发器	治标，掩盖 bug |
| parallel: false(vue.config.js 中添加) |	移除有缺陷的环节（thread-loader）| 	消除 bug 触发路径，最小代价 |
| 升级 vue-cli5 / Vite |	换掉停更工具链	| 长期治本，工作量大 |

#### 原因

```js
原理：@vue/cli-plugin-babel@4.4.6 中 useThreads = NODE_ENV === 'production' && !!options.parallel，设 parallel: false 后生产构建不再注入 thread-loader，vue-loader 的 templateLoader 在主进程执行，this.rootContext 正常，path.relative 不再收到 undefined。
```

