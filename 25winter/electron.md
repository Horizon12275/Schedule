## 1.23

1. 官方文档：[electron](https://www.electronjs.org/docs/latest/tutorial)，里面有一个简易的基于 nodejs 的例子，就是 createwindow、然后当 window 都关闭的时候，quit app。

2. 通过下面的方式进行 build and package 为可执行文件

```bash
npm install --save-dev @electron-forge/cli
npx electron-forge import
npm run make
```

3. 通过下面的方式进行 publish、注意需要在环境变量里设置 GITHUB_TOKEN，在 https://github.com/settings/tokens 界面可以找到 classic token ，给几个仓库权限就行

```bash
npm install --save-dev @electron-forge/publisher-github
```

```js
//forge.config.js
module.exports = {
  publishers: [
    {
      name: "@electron-forge/publisher-github",
      config: {
        repository: {
          owner: "github-user-name",
          name: "github-repo-name",
        },
        prerelease: false,
        draft: true,
      },
    },
  ],
};
```

```json
// package.json
  //...
  "scripts": {
    "start": "electron-forge start",
    "package": "electron-forge package",
    "make": "electron-forge make",
    "publish": "electron-forge publish"
  },
  //...
```

```bash
npm run publish
```

这里发现了一个小问题、如果用 vscode 的 ps 终端，会出现找不到环境变量的问题，可以用 cmd 终端代替

4. 也可以用 github action 进行自动发布
