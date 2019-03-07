# jsonp

设计背景：
南京合同系统有通过品高门户进入合同系统的功能，也就是通过品高的代理域名：jsgcdl.njmetro.com.cn（以下简称代理域名）跳转到合同系统内网域名jsgc.njmetro.com.cn（以下简称内网域名）
我们知道，跨域访问会丢失session，在jsgcdl.njmetro.com.cn登录的用户信息无法带到jsgc.njmetro.com.cn这个地址中，所以我们这里简单的说一下使用jsonp怎么在获取账号信息

if (queding) {
                $.ajax({
                    url: "/sso/captcha/check.html", //请求的url地址
                    dataType: "json",
                    data: { "scode": neirong },
                    type : "POST",
                    beforeSend: function() {
                        //请求前的处理
                    },
                    success: function(req) {
                        //请求成功时处理
                        $(".yanzheng_p").text(req.msg);
                        $(".yanzheng_p").show();
                        if (req.success) {
                            // 验证成功进入系统	
                            setTimeout(function(){
                            	window.location= "http://jsgc.njmetro.com.cn/back.common.sysbase.Loading.d?oa=1&highkick=1";
                            }, 1000);
                        } 
                    },

                    error: function(s) {
                        alert("请求错误");
                    }
                });
            }
 上面这段代码是代理域名跳转到内网域名Loading页面的功能，我们在Loading页面的js做处理
 
 !function(self, arg) {
	// 判断是否是从品高门户跳转而来，从品高跳转来的cookie包含username
	if(1 == "${param['highkick']}"){
		jsonp_jsgcdl();
	}else{
		timer = setInterval("updateTmpProgressBar(1)",80);
		view.id("cmdGetPageindex").execute(function(map){
			var oa = "${param['oa']}";
			if(oa!=null && oa!=""){
				surl = map.returnUrl+"?oa="+oa;
			}else{
				surl = map.returnUrl;
			}
		});
	}
	
};

//访问品高的代理页面，获取账号信息，传到内网域名中保存信息
window.jsonp_jsgcdl = function(){
	$.ajax({
		//url:'http://localhost:8080/sso/captcha/setSession.html',
		url:'http://jsgcdl.njmetro.com.cn:8020/sso/captcha/setSession.html',
		type:'get',
		data:{},
		dataType: "jsonp",
		jsonpCallback:"successCallback",
		success: function(data){
			timer = setInterval("updateTmpProgressBar(1)",80);
			view.id("cmdGetPageindex").execute(function(map){
//				if(isNotEmpty(data.msg)){
					//jsonp_jsgc(data.msg,map.returnUrl);
					$.ajax({
						//url:'http://localhost:8080/loginSso.html',
						url:'http://jsgc.njmetro.com.cn/loginSso.html',
						type:'get',
						data:{'data':data.msg},
						dataType: "jsonp",
						jsonpCallback:"successCallback",
						success: function(data){
							var oa = "${param['oa']}";
							if(oa!=null && oa!=""){
								//surl = "http://jsgc.njmetro.com.cn/" + returnUrl+"?oa="+oa;
								surl = map.returnUrl+"?oa="+oa;
							}else{
								//surl = "http://jsgc.njmetro.com.cn/" + returnUrl;
								surl = map.returnUrl;
							}
							
						},
						error: function(){
							//window.location = "http://localhost:8080/back.common.sysbase.Loading.d?oa=1";
							//window.location = "http://jsgc.njmetro.com.cn/back.common.sysbase.Loading.d?oa=1";
						}
					});
			});
		},
		error: function(){
		}
	});
};

上面的含义大概是，在内网域名Loading的js方法中，使用jsonp的形式，去访问代理域名中的一个获取用户信息的方法

@RequestMapping(value = "setSession.html")
	@ResponseBody
	public String setSession(HttpServletRequest request,HttpServletResponse response) {
		String callRes = "";
		//获取回调函数名
//		SysUser user = UserUtil.getSessionUser();
		SysUser user = UserUtil.getSessionUser(request.getSession());
//		if(StringUtil.isNotEmpty(data)){
			Cookie cookie = new Cookie("username",user.getSloginid());
			cookie.setPath("/");
			cookie.setMaxAge(60);
			response.addCookie(cookie);
			
			callRes = "successCallback(" + JSON.toJSONString(new CheckResult(true, user.getSloginid())) + ")";
//		}
		return callRes;
	}
    
获取的信息放入cookie中，可以在内网域名下获取，然后再做一次登录，达到登录效果
