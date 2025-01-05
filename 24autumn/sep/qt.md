## 配置流程

1. 除了用自带的qt creator启动、还可以配置cmakeList在vscode中启动

2. 注意在配置cmakelist的时候、需要同时更改里面的rc和qrc文件的路径和名称、否则会报错

```cmake
error MSB4035: 元素 <CustomBuild> 中缺少必需的特性“Include”，或该特性为空。
```

3. qt creator还可以配置copilot (见https://blog.csdn.net/weixin_45507142/article/details/131602800)

4. 需要改动mainwindow为qwidget