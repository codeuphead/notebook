# npm 淘宝源

临时使用

```bash
npm --registry https://registry.npm.taobao.org install express
```

持久使用

```bash
npm config set registry https://registry.npm.taobao.org
```

确认源

`npm config get registry`

cnpm

```bash
npm install -g cnpm --registry=https://registry.npm.taobao.org
```

恢复使用

```bash
npm config set registry https://registry.npmjs.org
```
