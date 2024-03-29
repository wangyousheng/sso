# sso使用指南
## 修改host:
>127.0.0.1  www.a.com 

>127.0.0.1  www.b.com 

>127.0.0.1  www.sso.com 
## sso单独部署为www.sso.com
利用window.postMessage跨域请求
## 子系统分别部署为www.a.com www.b.com
```
window.addEventListener('message',function(e){
	if(e.source!=window.parent) return;
	localStorage.setItem("tokenData",e.data);
},false);
```

## sso页面
```
<title>SSO首页</title>
<meta charset="utf-8">
<h1>sso www.sso.com/sso.html</h1>
<p>
    <button onclick="login();" id="login">同步登录</button>
</p>

<iframe src="http://www.a.com:8082" style="height: 0px;width: 0px;display: none;"></iframe>
<iframe src="http://www.b.com:8084" style="height: 0px;width: 0px;display: none;"></iframe>
<script>

// 同步登录
function login(){
	var loginData={userName:'测试兄弟'};
	//登录
	//...
	
	//登录结果	Data
    var data=JSON.stringify({
		status:true,
		userName:'测试兄弟',
        msg:'登录成功',
        token:'sso token'
    });
	localStorage.setItem("tokenData",data);
    document.getElementsByTagName('iframe')[0].contentWindow.postMessage(data,'*');
	document.getElementsByTagName('iframe')[1].contentWindow.postMessage(data,'*');
	
	var referUrl = getQuery("referUrl");
	window.location.href=referUrl;
}
function getQuery(name){
	var reg = new RegExp("(^|&)"+ name +"=([^&]*)(&|$)");
	var r = window.location.search.substr(1).match(reg);
	if (r!=null) return unescape(r[2]); return null;
}
</script>
```

## 子系统A页面
```
<title>A首页</title>
<meta charset="utf-8">
<h1>A首页 www.a.com:8082</h1>
<p>
    <p>用户信息：<span id="userInfo">未登录</span></p>
	<button onclick="login();" id="login">去sso登录</button>
</p>
<script>
// 去sso登录
function login(){
	window.location.href="http://www.sso.com/sso.html?referUrl="+window.location.href;
}
//登出
function logout(){
//todo
}
//注册监听 有消息从父级(SSO页面请求)传来时 存贮 tokenData
window.addEventListener('message',function(e){
	if(e.source!=window.parent) return;
	localStorage.setItem("tokenData",e.data);
},false);

 window.onload = function() {
	//显示用户信息
	var tokenData=localStorage.getItem('tokenData');
	if(tokenData){
		document.getElementById('login').style.display='none';
	}
	document.getElementById('userInfo').innerHTML =tokenData;
}
</script>
```

## 子系统B页面
```
<title>B首页</title>
<meta charset="utf-8">
<h1>B首页 www.b.com:8084</h1>
<p>
    <p>用户信息：<span id="userInfo">未登录</span></p>
	<button onclick="login();" id="login">去sso登录</button>
    </p>
</p>
<script>
// 去sso登录
function login(){
	window.location.href="http://www.sso.com/sso.html?referUrl="+window.location.href;
}
//登出
function logout(){
//todo
}
//注册监听 有消息从父级(SSO页面请求)传来时 存贮 tokenData
window.addEventListener('message',function(e){
	if(e.source!=window.parent) return;
	localStorage.setItem("tokenData",e.data);
},false);

 window.onload = function() {
	//显示用户信息
	var tokenData=localStorage.getItem('tokenData');
	if(tokenData){
		document.getElementById('login').style.display='none';
	}
	document.getElementById('userInfo').innerHTML =tokenData;
}
</script>
```
## 流程
* 启动所有系统
* 进入子系统A首页时，提示去sso登录
* 点击按钮，进入sso （A系统url会带到sso的url参数上，方便登录后跳回A系统）
* sso登录，获取到用户信息data，对(所有子系统列表)iframe发送data；子系统监听收到的data后保存
* 跳回子系统A
* 进入子系统B首页时（第四步已经存入用户信息），提示已经登录
# 存在的问题
## 如果子系统比较多，那一次登录几十秒，尴尬了
## 解决
* 启动所有系统
* 进入子系统A首页时，提示去sso登录
* 点击按钮，进入sso （A系统url会带到sso的url参数上，方便登录后跳回A系统）
* sso登录，获取到用户信息data，保存用户信息到sso域，对(当前子系统)iframe发送data；子系统监听收到的data后保存
* 跳回子系统A
* 进入子系统B首页时，发现没有用户信息，跳转到sso，携带B系统url
* 获取到sso域的用户信息，后台认证，成功之后，对（当前子系统）iframe发送data，子系统监听收到的data后保存;
* 自动跳转回B系统
