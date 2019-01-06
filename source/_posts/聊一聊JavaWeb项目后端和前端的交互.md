---
title: 聊一聊JavaWeb项目后端和前端的交互
date: 2018-11-27 21:56:45
tags: Java Web
---
最近在做学校老师的一个钢管租赁网站的项目，我负责前端的开发。再加上期中考试和各种各样的作业博客也是好久没有更新了（=·=好吧其实是我懒）言归正传，这也算是我第一次参加正式的开发项目，前前后后改了不少次，但确实是学到了很多东西，这和自己平时在网上看看博客写写demo完全是不一样的。
<!--more-->
这个项目的后台是很早就已经开发得差不多了的。项目采取前后端代码分离的架构，即前端是完全的html格式文件，并非jsp模版文件，数据渲染一律采用ajax方式请求后端获取。今天就来聊一聊JavaWeb项目后端和前端的交互问题，一是增强记忆，二来最近身边的同学也在学习JavaWeb的开发（好像都很懵逼的样子，其实我也只了解一些皮毛）

## 后端Controller的编写
现在jave web后端的主流框架为SSM（Spring   SpringMVC    Mybatis）或者是SpringBoot + Mybatis。它们都是以MVC为基础的框架，在前后端代码分离的项目中，编写Controller层简单粗暴的说就是设计接口以便前端调用。下面是一段SpringBoot中Controller层的代码：
```
@Controller
@RequestMapping("/api")
public class DemoController {
	@RequestMapping(value=“/getinfo”,method=RequestMethod.GET)
	@ResponseBody
	private Map<String,Object> getInfo(){
		Map<String,Object> modelMap = new HashMap<String,Object>();
		...
		...
		...
		(调用service层的接口）
		return modelMap;
	}
}
```
这就是一个可以通过get方式请求的接口，运行项目，访问http://localhost:8080/projectname/api/getinfo 便可以获取json格式的数据。
如果是post方式的请求，那么将代码`@RequestMapping(value=“/getinfo”,method=RequestMethod.GET)`中的GET改为POST，并使getInfo方法接收一个参数`HttpServletRequest request` ，然后在方法内处理请求体获取需要的数据便可（具体处理在这里不详述）

## 前端ajax调用接口
这里便不仔细解释了，给出get与post两种方式的ajax请求模版（使用jquery）
具体的参数大家可以自行谷歌了解。
GET方式：
```
$.getJSON(initUrl, function(data) {
	...
	...	
});
```

POST方式：
```
$.ajax({
	url : Url,
	type : ‘POST’,
	data : formData,
	contentType : false,
	processData : false,
	cache : false,
	success : function(data) {
		if (data.success) {
			$.toast(‘提交成功！’);
		} else {
			$.toast(‘提交失败！’ + data.errMsg);
			console.log(data.errMsg);
		}
	}
});
```