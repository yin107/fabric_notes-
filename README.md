# 在路径上添加自定义文字
# input框

```javascript
<input type="color">
```
为用户提供了指定颜色的用户界面，或者可视化颜色选择器

```javascript

<div>
    <input type="color" id="head" name="head"
           value="#e66465">
    <label for="head">Head</label>
</div>

```
![在这里插入图片描述](https://img-blog.csdnimg.cn/05d5bb982c05492ebac7fa676e6d5d4e.png)

```javascript
	<input type="color">
	<p>我是一个颜色的显示</p>
 <script>
		let input=document.querySelector('input')
		let p=document.querySelector('p')
		input.oninput=function(){//每次颜色更改都会触发input事件
   console.log('正在选色中');
		}
		input.onchange=function(e){//用户关闭选色器之后就会触发change事件
console.log(e.target.value);
p.style.color=e.target.value
		}
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/47849be196ec401f82c81ab54a562cbb.png)
demo4和demo5是关于图片处理的，不知道干什么，暂时应该还不需要先不学
# input导入图片

```javascript
	<input type="file" accept="image/*">

```

> input标签中有一个accept属性，accept属性只能与\<input type="file"\>配合使用，它规定能够通过文件上传进行提交的文件类型
> 上传图片时，选择 accept="image/*"  但是有时候文件选择窗口打开会比较慢，如果仅仅是上传图片，可以使用\<input type="file" accept="image/png, image/jpeg, image/gif, image/jpg">
> 其他选择总结：
> 

```javascript
<input type="file" webkitdirectory directory multiple/>
        <label for="file">上传文件夹</label>
        <br>
        <input type="file" accept="application/pdf"/>
        <label for="file">上传pdf文件</label>
        <br>
        <input type="file" accept="audio/x-mpeg"/>
        <label for="file">上传mp3文件</label>
        <br>
        <input type="file" accept="text/html"/>
        <label for="file">上传html文件</label>
```

> 多个属性值使用逗号分隔\<input accept="audio/*,video/*,image/*">
## 将上传的图片添加到canvas中
[看博文](https://blog.csdn.net/qq_45387575/article/details/127800859)

# 画刷-铅笔相关的设置

```javascript
		let canvas=new fabric.Canvas('canvas',{
			width:400,
			height:300,
			isDrawingMode:true,//开启绘画模式
			// freeDrawingBrush:new fabric.PencilBrush({
			// 	decimate:8,})
		})
let pencilBrush=new fabric.PencilBrush(canvas)//创建铅笔  需要传入canvas
pencilBrush.initialize(canvas)//初始化铅笔
canvas.freeDrawingBrush=pencilBrush//将画笔设置为铅笔
pencilBrush.color='red'//设置铅笔的颜色
```
更多设置可以参考[掘金-铅笔](https://juejin.cn/post/7145662816022691847)
## before:path:created
准备是生成路线时触发。

```javascript
canvas.on('before:path:created',function(opt){
	console.log(opt.path);
})
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/452c04a519c9427e8ea56bf5584a4d7d.png)
可以拿到绘制的路径

> ‘M’：表示移动命令，表示开始的坐标点
>  “L”:表示达到的点
## path:created
路线生成完毕时触发

```javascript
canvas.on('path:created', function(opt) {
  console.log(opt.path)
})
```

## 实现
### fabric.util.getPathSegmentsInfo
运行经过解析和简化的路径，并提取一些信息，信息是每个命令的长度和起点

```javascript
canvas.on('before:path:created',function(opt){
	var path=opt.path//获取整个对象
	var pathInfo=fabric.util.getPathSegmentsInfo(path.path)//获取路径 M L 等等
	console.log(path.path,pathInfo);
})
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/402057e6de2941e19a344cabe15c73ba.png![在这里插入图片描述](https://img-blog.csdnimg.cn/d86e5e12f7a449a09265fee1c0b1b289.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/d570722927b04a7dbea4eb9c48074c80.png)

```javascript
	<style>
	#canvas{
		border:1px solid black
	}
	</style>
</head>
<body>
	<canvas id="canvas"></canvas>
	<input type="file" accept="image/png, image/jpeg, image/gif, image/jpg" id="input">
	<script src='./fabricdemo4.js'></script>
	<script>
		let canvas=new fabric.Canvas('canvas',{
			width:400,
			height:300,
			isDrawingMode:true,//开启绘画模式
			// freeDrawingBrush:new fabric.PencilBrush({
			// 	decimate:8,})
		})
let pencilBrush=new fabric.PencilBrush(canvas)//创建铅笔  需要传入canvas
pencilBrush.initialize(canvas)//初始化铅笔
canvas.freeDrawingBrush=pencilBrush//将画笔设置为铅笔
pencilBrush.color='red'//设置铅笔的颜色
pencilBrush.decimate=8 //修改拐角的平滑度，数值越大就越平滑

canvas.on('before:path:created',function(opt){
	var path=opt.path//获取整个对象
	var pathInfo=fabric.util.getPathSegmentsInfo(path.path)//获取路径 M L 等等 并且对路径进行了解析
path.segmentsInfo=pathInfo//添加变化的路径属性
var pathLength=pathInfo[pathInfo.length-1].length//获取所绘制路径的总长度
var text='this is a demo, today is 2022-11-12'
var fontSize=2.5*pathLength/text.length
var text=new fabric.Text(text,{
	fontSize:fontSize,
	path:path,//文字的路径
	top:path.top,
	left:path.left
})
canvas.add(text)
})
canvas.on('path:created',function(opt){//路线生成完毕时候触发
	canvas.remove(opt.path)
})
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/55d8f30bd5794f04beeb1e492b69ccc4.png)

```javascript
fabric.Object.prototype.objectCaching=true   //开启对象的缓存有什么用处？
```

