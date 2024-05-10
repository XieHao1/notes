# 定义全局变量和封装AJAX

## 1.定义全局变量

为了在移动端项目上集中管理URL路径，我们可以在`main.js`文件中用全局变量的语法，定义全局的URL地址，这样更加便于维护。

```javascript
let baseUrl = "http://localhost:8888/emos-wx-api"
// 向VUE对象中绑定全局变量
Vue.prototype.uni = {
	register: baseUrl + "/user/register"
}
```



## 2.封装Ajax

移动端通过Ajax向服务端提交请求，然后接收到的响应分若干种情况:

1.如果用户没有登陆系统，就跳转到登陆页面。

2.如果用户权限不够，就显示提示信息。

3.如果后端出现异常，就提示异常信息。

4.如果后端验证令牌不正确，就提示信息。

5.如果后端正常处理请求，还要判断响应中是否有Token。如果令牌刷新了，还要在本地存储Token。

如果移动端每次发出Ajax，都要做这么多的判断，我们的重复性劳动太多了。所以尽可能的把Ajax封装起来，减少重复性的劳动。

```javascript
//绑定全局请求调用
Vue.prototype.ajax = function(url,method,data,fun){
	uni.request({
		url:url,
		method:method,
		data:data,
		header:{
			token:uni.getStorageSync("token")
		},
		success:(resp) => {
			//statusCode是HTTP状态码
			if(resp.statusCode === 401){
				uni.navigateTo({
					url:"/pages/login/login"
				})
			}else if(resp.statusCode === 200 && resp.data.code === 200){
				let data = resp.data
				//如果返回的数据中携带token，则保存
				 if(data.hasOwnProperty("token")){
					 console.log(resp.data)
					 let token = data.token
					 uni.setStorageSync("token",token)
				 }
				 fun(resp)
			}else{
				uni.showToast({
					icon:'none',
					title:resp.data
				})
			}
		}
	})
}
```



## 3.调用

```javascript
that.ajax(that.url.register, "POST", data, function(resp) {
		console.log(resp)
		let permission = resp.data.data.permission
		uni.setStorageSync("permission", permission)
	    console.log(permission)
})
```

