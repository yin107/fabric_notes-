## 导入文件

```javascript
  <div>
        <input type="file" accept="application/pdf">
    </div>
```
### input type="file" 属性
#### multiple

```javascript
multiple="multiple" 
```
可以接受多个值的文件上传
#### accept

```javascript
accept="application/pdf"
```
限定文件上传的类型
也可以定义多个文件类型，用逗号进行隔开
但注意不要将这个属性作为唯一的验证工具，应该在服务器上对文件进行验证
![在这里插入图片描述](https://img-blog.csdnimg.cn/54cd933b342e40fc9fc3a4f01e3def06.png)

```javascript
<canvas id="canvas" width="300px" height="200px"></canvas>
```

```javascript
import { fabric } from 'fabric'
    mounted() {
        const canvas = this._canvas = new fabric.Canvas('canvas', {
            width:600,height:600 //发现在这里面设置canvas的高度和宽度才有效
        })
```

### 添加元素

```javascript
        const text = new fabric.Text('Upload PDF')
        canvas.add(text)
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/e70dcf26fd06418099a3355f59b142ab.png)
### 监听input事件
只要输入的值变化了就会触发input事件，当文件上传完成后就会触发
```javascript
        <input type="file" accept="application/pdf" @input="input">
            methods: {
        input() {
        console.log('2');
    },
```
因为是在methods里面定理的事件，需要处理canvas和text，因此需要对数据进行重新修改

```javascript
data() {
    return {
        text: {},
        canvas: {}
    }
    },
    mounted() {
        this.canvas = this._canvas = new fabric.Canvas('canvas', {
            width:600,height:600
        })
        this.text = this._text= new fabric.Text('Upload PDF')
        this.canvas.add(this.text)
    },
  
    methods: {
        input(e) {
            console.log(this.text);
            this.text.set('text','loading...')
        console.log(e.target);
    },
```
### 设备像素比

```javascript
const scale = 1 / window.devicePixelRatio
```
设备像素比返回了设备上的物理像素和当前设备CSS像素的一个壁纸，也就是说明浏览器多少屏幕实际的像素点被用来画了一个CSS像素点

> window.devicePixelRatio = 物理像素 / css像素
常用来修正canvas的像素比
### 安装pdfjs-dist
```javascript
npm install --save pdfjs-dist@2.0.943
```
Blob对象表示一个不可变、原始数据的类文件对象
## 自己理不清楚了，先全部拷贝试一下

```javascript
 <div>
        <input type="file" accept="application/pdf">
    </div>
    <canvas id="c" width="500" height="620">
    </canvas>
    <script src="//mozilla.github.io/pdf.js/build/pdf.js"></script>
    <script src="https://unpkg.com/fabric@4.6.0/dist/fabric.min.js"></script>
    <script>
        const Base64Prefix = "data:application/pdf;base64,";

        function getPdfHandler() {
            console.log(window['pdfjs-dist/build/pdf']);
            return window['pdfjs-dist/build/pdf'];
        }

        function readBlob(blob) {
            return new Promise((resolve, reject) => {
                const reader = new FileReader();
                reader.addEventListener('load', () => resolve(reader.result));
                reader.addEventListener('error', reject)
                reader.readAsDataURL(blob);
            })
        }

        async function printPDF(pdfData, pages) {
            const pdfjsLib = await getPdfHandler();
            pdfData = pdfData instanceof Blob ? await readBlob(pdfData) : pdfData;
            const data = atob(pdfData.startsWith(Base64Prefix) ? pdfData.substring(Base64Prefix.length) : pdfData);
            console.log(data);
            console.log(pdfjsLib);
            // Using DocumentInitParameters object to load binary data.
            const loadingTask = pdfjsLib.getDocument({
                data
            });
            return loadingTask.promise
                .then((pdf) => {
                    const numPages = pdf.numPages;
                    return new Array(numPages).fill(0)
                        .map((__, i) => {
                            const pageNumber = i + 1;
                            if (pages && pages.indexOf(pageNumber) == -1) {
                                return;
                            }
                            return pdf.getPage(pageNumber)
                                .then((page) => {
                                    //  retina scaling
                                    const viewport = page.getViewport({
                                        scale: window.devicePixelRatio
                                    });
                                    // Prepare canvas using PDF page dimensions
                                    const canvas = document.createElement('canvas');
                                    const context = canvas.getContext('2d');
                                    canvas.height = viewport.height
                                    canvas.width = viewport.width;
                                    // Render PDF page into canvas context
                                    const renderContext = {
                                        canvasContext: context,
                                        viewport: viewport
                                    };
                                    const renderTask = page.render(renderContext);
                                    return renderTask.promise.then(() => canvas);
                                });
                        });
                });
        }

        async function pdfToImage(pdfData, canvas) {
            const scale = 1 / window.devicePixelRatio;
            return (await printPDF(pdfData))
                .map(async c => {
                    canvas.add(new fabric.Image(await c, {
                        scaleX: scale,
                        scaleY: scale,
                    }));
                });
        }

        const canvas = this.__canvas = new fabric.Canvas('c');
        const text = new fabric.Text('Upload PDF');
        canvas.add(new fabric.Circle({
            radius: 100,
            fill: 'green'
        }), text);
        document.querySelector('input').addEventListener('change', async(e) => {
            text.set('text', 'loading...');
            canvas.requestRenderAll();
            pdfToImage(e.target.files[0], canvas)
                // await Promise.all(pdfToImage(e.target.files[0], canvas));
            canvas.remove(text);
        });
    </script>
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/724069e1c7874abd8b9d78f878bda0b6.png)
## 发现await和async和promise又不熟练
## PDFJS API-在线预览PDF插件
PDF.js可以实现在html下直接浏览pdf文档，可以将PDF文件渲染成canvas
PDF.js主要包含两个库文件，一个pdf.js负责API解析和pdf.worker.js负责核心解析
**在线引入**

```javascript
    <script src="//mozilla.github.io/pdf.js/build/pdf.js"></script>

```
### Base64、Blob、File之间的相互转换
Base64是网络上最长用于传输8bit字节码的编码方式之一，是一种基于64个可打印字符来表示二进制数据的方法
![在这里插入图片描述](https://img-blog.csdnimg.cn/c98198930af143e6994083c5d9b05e68.png)

Blob对象表示一个不可变、原始数据的类文件对象，它的数据可以按文本或二进制的格式进行读取
![在这里插入图片描述](https://img-blog.csdnimg.cn/048a7542afb045b88ad20827cf779061.png)

File接口提供有关文件的信息，并允许网页中的js访问其内容
### atob（）
可以对base-64编码的字符串进行解码
里面必须要传入一个以base64编码的字符串，不然就会报错

```javascript
Uncaught DOMException: Failed to execute 'atob' on 'Window': The string to be decoded is not correctly encoded.
```
## reader.onloadend()
**是一个异步任务** 要注意执行顺序
![在这里插入图片描述](https://img-blog.csdnimg.cn/6359a9a3e6864ea08a626855b0ffc33b.png)
现在是要等到reader.onload执行完毕之后，等到url赋值为一个base64编码的字符串之后，才对该字符串进行解码
**采用promise解决：**

```javascript
 _input.addEventListener('change', function(e) {
            console.log(e.target.files[0]); //拿到了这个文件

            new Promise((resolve, reject) => {
                let reader = new FileReader()
                    //    let url = ''
                reader.onloadend = function(e) { //读取完成触发，无论失败还是成功
                    //    console.log(this.result)
                    //    url = e.target.result
                    //  console.log(url); //base64格式
                    resolve(e.target.result)
                }
                reader.readAsDataURL(e.target.files[0]) //给filereader对象一个读取完成的方法，使用readAsDataURL会返回一个url，这个值就保存在事件对象的result中
            }).then(resolve => {
                console.log(resolve);
                let data = atob(resolve.startsWith(Base64Prefix) ? resolve.substring(Base64Prefix.length) : resolve)
                console.log(data);
            })

            let Base64Prefix = "data:application/pdf;base64,"
                //对刚才拿到的base64的字符串进行解码
                //    let data = atob(url.startsWith(Base64Prefix) ? url.substring(Base64Prefix.length) : url)
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/5701fe253ff7429cb4be4610f1a5de5a.png)
## map（）
该方法定义在js的数组中，它返回一个新的数组，数组中的元素为原始数组调用函数处理后的值，该方法不会对空数组进行检测，不会改变原始数组

```javascript
array.map(function(currentValue, index, arr), thisIndex)
```

> function(currentValue, index, arr) 必须为一个函数，数组中的每个元素都会执行这个函数
> currentValue ：必须，当前元素的值
> index：可选 ，当前元素的索引
> arr：可选，当前元素属于的数组对象
> thisValue：可选，对象作为该执行回调时使用，传递给函数，用作this的值

###  pdf.numPages
PDFJS里面的一个成员变量 pdf.numPages，如果需要显示整个PDF，那么久需要边丽丽他们，但是需要注意的是，在pdf.js中获取页面是一步的，因此需要使用promise来保证执行的顺序

### canvas.getContext()
返回canvas的上下文

```javascript
var ctx = canvas.getContext(contextType);
var ctx = canvas.getContext(contextType, contextAttributes);
```

> contextType 上下文类型，是一个使用何种上下文的DOMString
> 可以值为：'2d':建立一个二维渲染上下文   等

### fabric.Image()
参数为DOM元素
![在这里插入图片描述](https://img-blog.csdnimg.cn/e5cad98b1a6a4f0e9235cfd3d3e19ed8.png)
通过给定的图片地址添加
![在这里插入图片描述](https://img-blog.csdnimg.cn/b51859d690874d9b87047346fe5258af.png)

## 实现--仅通过promise

```javascript
    <style>
        #mycanvas {
            border: 1px solid black
        }
    </style>
</head>

<body>
    <div>
        <input type="file" accept="application/pdf" id='input'>
        <canvas id="mycanvas"></canvas>
    </div>
    <script src="//mozilla.github.io/pdf.js/build/pdf.js"></script>
    <script src="https://unpkg.com/fabric@4.6.0/dist/fabric.min.js"></script>
    <script>
        var canvas1 = new fabric.Canvas('mycanvas', {
            height: 800,
            width: 800
        })

        let newc = []
        let _input = document.querySelector('#input')
        _input.addEventListener('change', function(e) {
            //  console.log(e.target.files[0]); //拿到了这个文件
            new Promise((resolve, reject) => {
                let reader = new FileReader()
                reader.onloadend = function(e) { //读取完成触发，无论失败还是成功
                    resolve(e.target.result) //base64格式
                }
                reader.readAsDataURL(e.target.files[0]) //给filereader对象一个读取完成的方法，使用readAsDataURL会返回一个url，这个值就保存在事件对象的result中
            }).then(resolve => {
                let Base64Prefix = "data:application/pdf;base64,"
                let data = atob(resolve.startsWith(Base64Prefix) ? resolve.substring(Base64Prefix.length) : resolve) //对刚才拿到的base64的字符串进行解码
                    //    console.log(data);
                const loadingTask = pdfjsLib.getDocument({
                        data
                    }) //里面有一个promise属性
                loadingTask.promise.then(pdf => {
                    //      console.log('pdf 已经加载完成');
                    // console.log(pdf); //一个对象，里面包含pdf文件的一些信息,同时在其原型上面封装了可使用的api
                    const numPages = pdf.numPages //总的页数
                    let arr = new Array(numPages).fill(0)
                        //对每一页添加到canvas上
                    arr.map((cur, i) => {

                        const pageNumber = i + 1 //因此是要将导入的pdf文件中的每一页都渲染到canvas中
                        pdf.getPage(pageNumber).then(page => {

                            //     console.log('加载页面', page);
                            let scale = 1 / window.devicePixelRatio //得到在canvas中的渲染像素
                            let viewport = page.getViewport({
                                    scale: window.devicePixelRatio
                                }) //获取pdf尺寸
                                //准备canvas渲染pdf
                                //获取需要渲染的元素
                                //   const canvas = document.querySelector('#mycanvas')//就只能渲染一页
                                //因此应该创建多个元素，之后再添加到大的cnavas中
                            const canvas = document.createElement('canvas');
                            const context = canvas.getContext('2d')
                            canvas.height = viewport.height
                            canvas.width = viewport.width
                                //将pdf页面渲染到canvas上下文中
                            let renderContext = {
                                canvasContext: context,
                                viewport: viewport
                            }
                            let renderTask = page.render(renderContext)
                                //  document.body.appendChild(canvas) //直接在整个网页中添加新创建的canvas元素
                            let img = new fabric.Image(canvas) //将生成的canvas元素转化为Image    里面的canvas是一个DOM元素
                     canvas1.add(img)//添加到canvas中

                        })
                    })
                })
            })

        })
        var pdfjsLib = window['pdfjs-dist/build/pdf'] //一个有很多方法的对象
    </script>
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/d28e5e8f4587442abe50b05c25188c7b.png)
## 存在的问题
promise、async、await的使用
有些时候如果是异步代码，还没有请求到数据求返回去的话，那么这个值就肯定是undefined了


```javascript
   const Base64Prefix = "data:application/pdf;base64,";
        console.log(Base64Prefix);

        function getPdfHandler() {
            return window['pdfjs-dist/build/pdf'];
        }

        function readBlob(blob) {
            return new Promise((resolve, reject) => {
                const reader = new FileReader();
                reader.addEventListener('load', () => resolve(reader.result)); //文件读取成功完成时触发
                reader.addEventListener('error', reject) //出错时触发
                reader.readAsDataURL(blob); //将文件读取为DataURL
            })
        }

        async function printPDF(pdfData, pages) {
            const pdfjsLib = await getPdfHandler();
            pdfData = pdfData instanceof Blob ? await readBlob(pdfData) : pdfData; //得到base64表示的字符串，开头都是"data:application/pdf;base64,"
            //将base64编码的字符串进行解码
            const data = atob(pdfData.startsWith(Base64Prefix) ? pdfData.substring(Base64Prefix.length) : pdfData);
            console.log(data);
            // Using DocumentInitParameters object to load binary data.
            const loadingTask = pdfjsLib.getDocument({
                data
            });
            return loadingTask.promise
                .then((pdf) => {
                    const numPages = pdf.numPages;
                    return new Array(numPages).fill(0)
                        .map((_, i) => { //i是当前元素的索引号
                            const pageNumber = i + 1;
                            console.log(pages); //undefined的值
                            if (pages && pages.indexOf(pageNumber) == -1) { //这部分有什么用呢
                                return;
                            }
                            return pdf.getPage(pageNumber)
                                .then((page) => {
                                    //  retina scaling
                                    const viewport = page.getViewport({
                                        scale: window.devicePixelRatio
                                    });
                                    // Prepare canvas using PDF page dimensions
                                    const canvas = document.createElement('canvas');
                                    const context = canvas.getContext('2d');
                                    canvas.height = viewport.height
                                    canvas.width = viewport.width;
                                    // Render PDF page into canvas context
                                    const renderContext = {
                                        canvasContext: context,
                                        viewport: viewport
                                    };
                                    const renderTask = page.render(renderContext);
                                    console.log(renderTask.promise.then(() => canvas));
                                    return renderTask.promise.then(() => canvas);
                                });
                        });
                });
        }

        async function pdfToImage(pdfData, canvas) {
            const scale = 1 / window.devicePixelRatio;
            console.log(await printPDF(pdfData)); //返回的就是4个promise
            return (await printPDF(pdfData))
                .map(async c => {
                    console.log(c);
                    canvas.add(new fabric.Image(await c, {
                        scaleX: scale,
                        scaleY: scale,
                    }));
                });
        }

        const canvas = this.__canvas = new fabric.Canvas('c');
        const text = new fabric.Text('Upload PDF');
        canvas.add(new fabric.Circle({
            radius: 100,
            fill: 'green'
        }), text);
        document.querySelector('input').addEventListener('change', async(e) => {
            text.set('text', 'loading...');
            canvas.requestRenderAll();
            pdfToImage(e.target.files[0], canvas)
                //await Promise.all(pdfToImage(e.target.files[0], canvas));
            canvas.remove(text);
        });
    </script>
```
