# ChimeraCoffee-Web

### 原项目

https://github.com/macrozheng/mall-admin-web?tab=readme-ov-file

### 部署
```markdown
1. 下载nodejs并安装，最好使用v12.14.0版本，版本不对会导致npm install出错，下载地址：https://nodejs.org/dist/v12.14.0/node-v12.14.0-x64.msi

2. npm install

3. 地址路径为dev.env.js文件中的BASE_API为线上地址；原项目测试地址为：https://admin-api.macrozheng.com
npm run dev
```

### 项目布局

``` lua
src -- 源码目录
├── api -- axios网络请求定义
├── assets -- 静态图片资源文件
├── components -- 通用组件封装
├── icons -- svg矢量图片文件
├── router -- vue-router路由配置
├── store -- vuex的状态管理
├── styles -- 全局css样式
├── utils -- 工具类
└── views -- 前端页面
    ├── home -- 首页
    ├── layout -- 通用页面加载框架
    ├── login -- 登录页
    ├── oms -- 订单模块页面
    ├── pms -- 商品模块页面
    └── sms -- 营销模块页面
```


