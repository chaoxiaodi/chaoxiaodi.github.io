---
layout: post
title: "给 vue-element-admin 登录页面增加验证码功能"
subtitle: '优化 vue-element-admin 登录页面'
author: "chaoxiaodi"
header-style: text
tags:
  - vue
  - js
  - element
  - canvas
---

### 前言

出于工作需要与个人兴趣相结合，正在通过 python(django) + vue (vue-element-admin) 完成一个前后端分离的运维管理平台；

首先感谢 花裤衩 大佬开源的基于 `vue` + `element-ui` 的管理后台前端的解决方案 [vue-element-admin](https://github.com/PanJiaChen/vue-element-admin.git)；

其次感谢 sun-xd 大佬开源的使用 `canvas` 来生成验证码的 `vue` 插件 [identify](https://github.com/sun-xd/identify.git)

我是一个秃头运维，并不是专业的开发人员；在能找到现成好用的轮子的情况下，一般是不考虑自己去实现的。
    
### 验证码插件

这个验证码插件实现的功能只是针对验证码绘制以及一些属性的设置

生成验证码以及校验验证码的业务逻辑需要自己完成

这也是我觉得这个插件好用的地方之一：一个插件只干一件事

针对验证码插件进行了一些小部分的修改： 代码在这 [vue-identify](https://github.com/chaoxiaodi/vue-identify)

由于插件只提供了验证码的绘制，如果需要生成验证码或者刷新验证码就需要调用对应的绘制方法 `drawPic`

个人习惯是尽量对轮子进行最小的改动，通过看源码发现此插件的的主要方法就是绘制 `drawPic` 

所以个人的实现方法是：通过父组件传递验证码内容给 identify 插件，在 identify 插件中对 验证码内容进行监听

如果验证码内容发生改变，就调用绘制方法进行一次验证码绘制。

得益于 vue 的变量绑定吧，不用来回进行父组件与子组件之间的事件就能完成验证码的功能了。
    
    watch: {
    identifyCode() {
      this.drawPic()
    }
    
针对验证码插件的介绍就这么多，感兴趣的可以自行查看原作者的源码。或我修改后的源码
    
### vue-element-admin 登录页

花裤衩 大佬的代码写的真的叼，再加上他的教学博客，让我这小白也能慢慢摸索着用起来他的整套解决方案

花裤衩 大佬教程在这 [手摸手，带你用vue撸后台 系列](https://juejin.cn/post/6844903476661583880)

模板里登录代码路径： src/views/login/index.vue

原始的登录模板简洁到极致 一个 element-ui 的 `el-form`中包含了：

一个 h2 的标题

一个 用户名输入框 + 密码输入框 的

一个登录按钮

一个我用不上的第三方登录按钮

我这专业人员的思路就是照猫画虎，思考了下验证码大概的逻辑首先按照密码框的样式增加一个验证码输入框；
    
    <el-form-item prop="identify">
        <span class="svg-container">
          <svg-icon icon-class="identify" />
        </span>
        <el-input
          ref="identify"
          v-model="loginForm.identify"
          placeholder="验证码"
          name="identify"
          tabindex="3"
          @keyup.enter.native="handleLogin"
        />
        <span class="show-pwd" @click="refreshCode">
          <SIdentify :identify-code="identifyCode" />
        </span>
     </el-form-item>

通过单击验证码可以实现刷新操作
    
### 验证码具体的逻辑
    
为了增加验证码的复杂性，从最开始设计的纯数字验证码升级为数字验证码+运算验证码

偷懒只进行 加 减 乘 的运算

随机生成纯数字验证码或计算验证码
    
    // 制作验证码
    makeCode() {
      this.identifyCode = ''
      const flag = this.randomNum(0, 2)
      // 随机制作纯数字验证码或计算验证码
      if (flag === 0) {
        this.makeNumCode(4)
      } else { this.makeComputeCode() }
      // console.log(this.identifyCode, this.resultIdentifyCode)
    }
    
生成纯数字验证码，可以传递参数生成具体的位数，我生成的是 4位数字的验证码
    
    // 纯数字验证码
    makeNumCode(l) {
      for (let i = 0; i < l; i++) {
        this.identifyCode += String(this.randomNum(i, 10))
      }
      this.resultIdentifyCode = this.identifyCode
    }
    
生成计算验证码，变量起名真的太难了~原谅我使用了 x y z如此无礼的变量
    
    // 计算验证码
    makeComputeCode() {
      const x = this.randomNum(1, 10) // 第一个数字
      const y = this.randomNum(1, 10) // 第二个数字
      const z = this.randomNum(0, 3) // 算数符号
      if (z === 0) {
        this.identifyCode = String(x) + '+' + String(y) + '='
        this.resultIdentifyCode = x + y
      }
      if (z === 1) {
        if (x > y) {
          this.identifyCode = String(x) + '-' + String(y) + '='
          this.resultIdentifyCode = x - y
        }
        if (x <= y) {
          this.identifyCode = String(y) + '-' + String(x) + '='
          this.resultIdentifyCode = y - x
        }
      }
      if (z === 2) {
        this.identifyCode = String(x) + '*' + String(y) + '='
        this.resultIdentifyCode = x * y
      }
    }
    
两个生成验证码的方法都调用了，生成随机数的方法，本来还想生成英文还一直在todo列表搁置
    
提到todo顺便提一句使用python实现的一个todolist的小工具
    
[pytodolist](https://github.com/chaoxiaodi/pytodolist)
    
    // 验证码abcdefghijklnmopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ
    randomNum(min, max) {
      return Math.floor(Math.random() * (max - min) + min)
    }
    
在页面挂载的时候进行验证码的绘制
    
    mounted() {
      this.makeCode()
    }
    
涉及到的两个变量 resultIdentifyCode、identifyCode

identifyCode：绑定到了验证码组件，即前文 watch 的

resultIdentifyCode：存放验证码的结果，最后根据输入框的进行对比
    
### 登录按钮逻辑修改

登录按钮代码修改
    
    // 登录按钮
    handleLogin() {
      this.$refs.loginForm.validate(valid => {
        if (valid && this.checkIdentifyCode()) {
          this.loading = true
          this.$store.dispatch('user/login', this.loginForm).then(() => {
            this.$router.push({ path: this.redirect || '/' })
            this.loading = false
          }).catch(() => {
            this.loading = false
          })
        } else {
          // console.log('error submit!!')
          return false
        }
      })
    }
     
如果表单验证并且验证码正确，开始执行对应的登录逻辑
    
验证码校验方法
    
    // 检查验证码
    checkIdentifyCode() {
      if (this.loginForm.identify !== String(this.resultIdentifyCode)) {
        this.$message({ message: '验证码错误！', type: 'error' })
        return false
      } else { return true }
    }

### 后记
    
以上就是针对 vue-element-admin 登录页面进行改造的全部代码
    
作为一个开发小白，只能慢慢摸索，站在前人的肩膀上进行探索
    
如果有任何想法欢迎反馈、讨论
