---
layout: false
title: 前端练习1
date: 2022-01-21 23:07:48
tags: [html,css]
categories : [前端]
description : 一个紫色玻璃质感的登录界面
language: zh_CN
encoding: utf-8
---
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Login</title>
</head>
<style type="text/css">
* {
    padding:0;
    margin: 0;
}
html {
    height: 100%;
}
body{ 
    background-image:linear-gradient(to bottom right,rgb(114,135,254),rgb(130,88,186));
}
body .login-container{
    width: 600px;
    margin: 0 auto;
    margin-top: 20%;
    border-radius: 15px;
    box-shadow: 0 10px 50px 0px rgb(59,45,159);
    background-color: rgb(95,76,194);
}
body .login-container .left-container{
    display: inline-block;
    width: 315px;
    border-top-left-radius: 15px;
    border-bottom-left-radius: 15px;
    padding: 60px;
    background-image: linear-gradient(to bottom right,rgb(118,76,163),rgb(92,103,211));
}
}
body .login-container .left-container .title{
    color: aliceblue;
    font-size: 18px;
}
body .login-container .left-container .title  span{
    border-bottom:3px solid rgb(237,211,22);
}
body .login-container .left-container .input-container{
    padding: 10px 0;
}
body .login-container .left-container .input-container input{
    border: 0;
    background: none;
    outline: 0;
    color: #fff;
    margin: 20px 0;
    display: block;
    width: 100%;
    padding: 5px 0;
    transition: .2s;
    border-bottom-color:#fff;
    border-bottom: 1px solid rgb(199,191,219);
}
body .login-container .left-container .input-container input :hover{
    border-bottom-color: #fff;
}
::-moz-input-placeholder{
    color: rgb(199, 191, 219);
}
body .login-container .left-container .message-container{
    font-size:14px;
    transition: .2s;
    color: rgb(199, 191, 219);
    cursor: pointer;
}
body .login-container .left-container .message-container :hover{
    color: #fff;
}
body .login-container .right-container{
    width : 145px;
    display: inline-block;
    height: calc(100%-120px);
    vertical-align: top;
    padding: 60px 0;
}
body .login-container .right-container .regist-container{
    text-align: center;
    color:#fff;
    font-size: 18px;
    font-weight:200;
}
body .login-container .right-container .regist-container .register{
    border-bottom: 3px solid rgb(237,211,22);
}
    color: aliceblue;
body .login-container .right-container .action-container {
    color : #fff;
    width: 145px;
    padding-top: 59px;
    padding-left: 46px;
    position: relative;
}
body .login-container .right-container .action-container :hover{
    background-color: rgb(237,211,22);
}
body .login-container .right-container .action-container span{
    border: 1px solid rgb(237,211,22);
    padding: 10px;
    display: inline-block;
    line-height: 25px;
    border-radius: 125px;
    position: absolute;
}
</style>

<body>
    <div class="login-container">
        <div class="left-container">
            <div class="title">
                <span>登录</span>
            </div>
            <div class="input-container">
                <input type="text" name="username" placeholder=" 用户名">
                <input type="password" name="password" placeholder=" 密码">
            </div>
            <div class="message-container">
                <span>忘记密码</span>
            </div>
        </div>
        <div class="right-container">
            <div class="regist-container">
                <span class="register">注册</span>
            </div>
            <div class="action-container" style="padding-left: 45px;padding-top: 60px;">
                <span>提交</span>
            </div>
        </div>
    </div>
</body>
<script type="text/javascript" >
	alert("初次尝试，请多指教");
</script>
</html>
