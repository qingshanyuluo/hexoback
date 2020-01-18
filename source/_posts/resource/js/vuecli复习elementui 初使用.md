---
title: vuecli复习elementui 初使用
abbrlink: hkhlc06699c0
date: 2019-12-21 10:43:23
tag:
    - vue
    - npm
    - node
    - element ui
category: js
---
> 实训作业，数据库的管理系统，应付老师使用mybtais+springboot+vue+element ui 搭建了一个小区的物业管理系统，复习了一下这些技术栈，不过js确实把我迷的啊~~

> 后端就是简单的curd 前端是单页面应用，没有保持登录的功能，主要是token技术还没熟练，所以这次主要是ui上的工作

之前学vue的时候功力尚浅，只知然不知其所以然，这次有了新的理解，

### 项目目录：
.     
├── README.md   
├── babel.config.js   
├── node_modules/    
├── package-lock.json    
├── package.json    
├── public/   
├── src/   
└── vue.config.js  

笔者没有细致的学过node npm 对于其中的理解不求精准专业，但求易懂，望网友包容

## 总的理解：
node的出现使得js跳出浏览器的范围，js可以变身类似与python这样的语言操作文件，监听网络等等，vuecli就是类似于这样的一个“软件”，npm 安装包的方式很是粗暴 全局的存在用户目录下，项目的直接存在项目下（所以gitignore一定要把node_modules排除）
### vue.config.js 
一个js文件，是自己新加的，vuecli应该认识，可以读取里面的设置
### src/
代码
### public/
favicon和一个基础的html内容提示：  
We're sorry but wuliu doesn't work properly without JavaScript enabled. Please enable it to continue
还有个id=app的div
### package.js&package-lock.json
package.json 这个文件是 npm init 时创建的一个文件，会记录当前整个项目中的一些基础信息。而 package-lock.json 这个文件却是 node_modules 文件夹或者 package.json 文件发生变化时自动生成的。这个文件主要功能是确定当前安装的包的依赖，以便后续重新安装的时候生成相同的依赖，而忽略项目开发过程中有些依赖已经发生的更新。
### node_modules
安装项目域的node模块
### babel
babel官网正中间一行黄色大字写着“babel is a javascript compiler”，翻译一下就是babel是一个javascript转译器。为什么会有babel存在呢？原因是javascript在不断的发展，但是浏览器的发展速度跟不上。以es6为例，es6中为javascript增加了箭头函数、块级作用域等新的语法和Symbol、Promise等新的数据类型，但是这些语法和数据类型并不能够马上被现在的浏览器全部支持，为了能在现有的浏览器上使用js新的语法和新的数据类型，就需要使用一个转译器，将javascript中新增的特性转为现代浏览器能理解的形式。babel就是做这个方面的转化工作。

# src
```
.
├── App.vue
├── assets
│   └── logo.png
├── components
│   ├── Complaint.vue
│   ├── Express.vue
│   ├── NeedPay.vue
│   ├── NeedRepair.vue
│   ├── NewRepair.vue
│   ├── Paid.vue
│   ├── ShowInfo.vue
│   ├── ShowProperty.vue
│   ├── ShowUser.vue
│   ├── UploadInfo.vue
│   ├── UploadProperty.vue
│   └── admin
├── main.js
├── router
│   └── index.js
└── views
    ├── Admin.vue
    ├── Login.vue
    ├── Main.vue
    ├── Register.vue
    └── ULogin.vue
```

-------
任何.vue文件分三个部分：
```html
<template>
  
</template>

<script>
export default {

}
</script>

<style>

</style>
```
这里主要是script部分 export是 node部分的语法[参考](https://www.liaoxuefeng.com/wiki/1022910821149312/1023027697415616)
将模块暴露出去

查看Admin.vue是如何引用其他模块的:

```html
<Express/>
```
```javascript
import Express from '../components/admin/Express'

export default {
  name: 'Admin',
  components: {
    Express
  }
}
```

## Vue的学习可以查看官方文档
说几个要点：
1. mvvm
MVVM是一种设计思想。Model 层代表数据模型，也可以在Model中定义数据修改和操作的业务逻辑；View 代表UI 组件，它负责将数据模型转化成UI 展现出来，ViewModel 是一个同步View 和 Model的对象，是业务逻辑层（一切js可视业务逻辑，比如表单按钮提交，自定义事件的注册和处理逻辑都在viewmodel里面负责监控俩边的数据）在MVVM架构下，View 和 Model 之间并没有直接的联系，而是通过ViewModel进行交互，Model 和 ViewModel 之间的交互是双向的， 因此View 数据的变化会同步到Model中，而Model 数据的变化也会立即反应到View 上。这种自动同步是因为ViewModel中的属性实现了Observer，当属性变更时都能触发对应的操作。ViewModel 通过双向数据绑定把 View 层和 Model 层连接了起来，而View 和 Model 之间的同步工作完全是自动的，无需人为干涉，因此开发者只需关注业务逻辑，不需要手动操作DOM, 不需要关注数据状态的同步问题，复杂的数据状态维护完全由 MVVM 来统一管理。
2. 组件化 组件化是开始就吸引我的，保证代码的重用性，使得前端开发现代化。

## element ui
ui框架使得我的开发变得很容易，现在基本上回不去了，是单体开发的重要快捷方法，文档易懂

## axio
```js
var _this = this
let param = new URLSearchParams()
param.append('name', this.form.username)
param.append('pwd', this.form.password)
this.$axios.post('login', param)
  .then(function (response) {
    console.info(response.data)
    if (response.data.type === 0) {
      _this.$router.push({ name: 'Main', params: { user: response.data } })
    } else {
      _this.$alert('请使用管理员平台登录', '提示', {})
    }
  })
```
this的指向的，js的大坑，除了上面的方法，还可使用箭头函数，不过js的报错还是不够友好难找到错误，学好js多用多看chrome的控制台，
## http post
params 是参数，是请求头的部分，springmvc用@RequestParam 接收
其他的js对象在提交的时候则是放在body部分以json形式传输在springmvc中可以用@RequestBody接收

使用postman生成代码如下
```http
POST /express/get/4 HTTP/1.1
Host: localhost:8081
Content-Type: application/x-www-form-urlencoded
username: abc
Cache-Control: no-cache
Postman-Token: 227f3304-2477-4cb6-ae6f-dee9931d11f1

{
	"owner_id": 4,
	"create_time": "2019-12-17T16:00:00.000Z"
}
```
空行后是body部分
而param在前面

### [项目代码](https://github.com/qingshanyuluo/wuyefront)
