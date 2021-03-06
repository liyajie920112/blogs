# 细说长谈文字超长展开收起功能
如果只是单纯的文字超长，多余文字...隐藏，那么问题还是比较简单，几行css代码就搞定了，如下：  
```css
overflow: hidden;
text-overflow: ellipsis;
display: -webkit-box;
-webkit-line-clamp: 3;
-webkit-box-orient: vertical;
```
当前要实现的是文字超长，页面在末尾显示“展开”和“收起”两中状态，点击能做响应动作，展开按钮状态出现的前提是要内容三行放不下了，才能出现展开按钮，否则不应该出现展开按钮。如下：  
收起的样式：    
![展开](https://img11.360buyimg.com/jdphoto/s748x450_jfs/t1/79525/37/6458/337137/5d47f0a9Ef2a5ae5a/a5d40ddc07261984.png)  

展开的样式：  
![收起](https://img11.360buyimg.com/jdphoto/s764x862_jfs/t1/53126/2/6935/228838/5d47f0aaEa0c9bcec/090eecd43f8ae6d1.png)


如果用上述的css来实现的，没办法捕捉到...的事件，没有办法再展开内容了。 接下来我们用css和js来实现这两种方式。   
之前我们尝试过自己计算文字是否超长来做展开和隐藏，但是问题不少。
- 头部有标签，得加入计算
- 内容有换行标签
- 单行可显示的字符数不是刚好对应上，如可以显示3个字符，那么“aa中"，则中字换行。
- ……

总而言之经过复杂的代码处理之后（计算标签宽度，处理换行等），也基本能达到要求，但是总差点意思，“展开”按钮不一定总在最右边，效果不是太好。接下来介绍下很稳的效果。

## js的实现
我们首先用js来实现展开收起功能。  

### getBoundingClientRect

返回元素的大小及其相对于视口的位置。
返回值是一个 DOMRect 对象，这个对象是由该元素的 getClientRects() 方法返回的一组矩形的集合, 即：是与该元素相关的CSS 边框集合 。
DOMRect 对象包含了一组用于描述边框的只读属性——left、top、right和bottom，单位为像素。除了 width 和 height 外的属性都是相对于视口的左上角位置而言的。
![alt](https://img11.360buyimg.com/jdphoto/s500x500_jfs/t1/55981/21/6969/25422/5d48233fE525c5b87/629f21ea0c8fa36b.png)

同上，我们执行代码：  

```js
document.querySelector(".txt").getBoundingClientRect();
```
![alt](https://img11.360buyimg.com/jdphoto/s1246x396_jfs/t1/44334/15/11121/71425/5d4822a4Ead129289/75e5cc5ebd54bd4b.png)

### 了解 getClientRects
返回一个指向客户端中每一个盒子的边界矩形的矩形集合。 返回值是ClientRect对象集合，该对象是与该元素相关的CSS边框。每个ClientRect对象包含一组描述该边框的只读属性——left、top、right和bottom，单位为像素，这些属性值是相对于视口的top-left的。即使当表格的标题在表格的边框外面，该标题仍会被计算在内。

### 实战 getClientRects

```js
var rectCollection = Element.getClientRects();
```
例子：   
```html
<div id="txt">我是一个小文本，<br>我是一个小文本</div>
```
执行代码：
```js
document.querySelector(".txt").getClientRects();
```
返回结果如下:
![getClientRects](https://img11.360buyimg.com/jdphoto/s896x448_jfs/t1/62018/9/6431/63987/5d47f59bEd7f06081/7f824122daf35786.png)
如果我们将html的div改成span
```html
<span id="txt">我是一个小文本，<br>我是一个小文本</span>
```
返回结果如下：
![span](https://img11.360buyimg.com/jdphoto/s1346x228_jfs/t1/76637/31/6415/74885/5d47f59bEa6fd92a4/81056e5431913235.png)

对于行内元素，元素内部的每一行都会有一个边框；对于块级元素，如果里面没有其他元素，一整块元素只有一个边框.
所以是div就返回了三个DOMRect，但是div只返回一个。

### 文字超长处理（一）
我们采用文本遮住的方式来实现显示展开和收起。   

在显示的文本外增加一个文本框
```html
<div class="wrap">
  <span class="txt">我是一个小文本，<br>我是一个小文本</span>
  <span class="dot open">...展开</span>
</div>
```
然后我们设定字体大小和行高。比如字体是18，行高1.5。假如显示三行，则我们设置外层最大为18*3+9*2 = 72px;  如果是要动态文字大小，则同比计算即可。

```css
.wrap{
  width:100%;
  max-height:72px;
  position:relative;
  line-height:1.5;
  font-size:18px;
}
.dot{
    position: absolute;
    right: 0;
    bottom: 0;
    background: rgba(255,255,255,.9);
}

```
这个时候如果是文字超长，我们已经看到效果了。比如：    
![展开](https://img11.360buyimg.com/jdphoto/s397x106_jfs/t1/75001/4/8043/56637/5d5e5fdcEd1b0b01d/46365261c167364c.png)  

接下来我们需要把文字没有超长的时候，隐藏展开按钮即可。   
#### 第一步：判断外层的高度。
```js
document.querySelector(".wrap").getClientRects();

bottom: 258.359375
height: 72
left: 55
right: 399
top: 186.359375
width: 344
x: 55
y: 186.359375
```
如果height小于72，则直接隐藏，说明文字没有超长。

#### 第二步：获取内部文字高度。   

如果外层已经达到72px了，那么可能超长，也可能是刚好三行。    

```js
document.querySelector(".txt").getClientRects();
```
得到如下数据。
![txt](https://img11.360buyimg.com/jdphoto/s765x159_jfs/t1/53838/32/8484/62130/5d5e6242E8d6120a2/64ed387c555e2fff.png)  

- 如果行数大于三行，则是需要显示展开按钮。
- 如果小于等于三行，则不需要显示展开按钮

这里重点要看下，获取getClientRects的行数。要去掉换行等的空行，即Bottom值一样。
```js
function getRectsLength(rects) {
    var line = 0, lastBottom=0;
    for(var i=0,len=list.length;i<len;i++){
        if(list[i].bottom ==lastBottom){
            return;
        }
        lastBottom = list[i].bottom;
    		line++; 
		}
    return line;
}
```
上面的处理方式，使用外层的wrap，需要是一个块状元素。如果外层不是块状元素，比如如下，签名需要插入一个标签，文字挨着标签显示。则只能用第二种方式实现。  

![alt](https://img11.360buyimg.com/jdphoto/s424x194_jfs/t1/60138/16/7963/93807/5d5e68f9E4cbc3017/4328b3c59b9313cb.png)

### 文字超长处理（二）
使用getClientRects方法来处理文字超长，由于这个方法是实时返回页面元素的边界盒子，所以采用的是事后处理机制。

html页面内容，在每一个item里面，存储showTxt和hideTxt,开始内容都在showTxt里面。
```html
<span class="quan_feed_desc">
<span class="show_txt"></span>
<span class="dot">...</span>
<span class="hide_txt" style="display:none"></span>
<span class="quan_feed_more dot">展开</span>
</span>
```
开始渲染到页面的时候是整个内容都渲染上去,即渲染到上面的show_txt，然后再针对页面dom节点处理。当一页数据渲染完成之后，再执行handleText。   
获取没有处理过的列表项。  
```js
function handleText() {
    const dom = document.querySelectorAll(".quan_feed_desc:not([data-loaded]");
    if(dom.length==0) return;
    Array.from(dom).map((d)=>{
        handleDomLength(d);
    }); 
}
```
获取getClientRects的行数。要去掉换行等的空行，即Bottom值一样。
```js
function getRectsLength(rects) {
    var line = 0, lastBottom=0;
    for(var i=0,len=list.length;i<len;i++){
        if(list[i].bottom ==lastBottom){
            return;
        }
        lastBottom = list[i].bottom;
    		line++; 
		}
    return line;
}
```
单独处理某一项的行数。其中feedList是渲染的页面数据。循环处理字符，如果行数超过3行，则一直减少显示的文本数据，知道只有三行为止。这里有个优化点，加速回退的进程，减少回退次数，比如粗略计算文字的长度是否大幅高于3行，然后大幅回退文字。
```js
function handleDomLength(dom) {
    const item = feedList[dom.dataset.index];
    const rects = dom.getClientRects();
    var h = getRectsLength(rects);
    if(h<3||!item.showTxt){
        dom.dataset.loaded = 1;
        continue;
    }
    
    dom.querySelectorAll(".dot").forEach(function (el) {
        el.style.display="inline-block";//把点点和展开放开，为后面计算更真实
    });
    let t = item.showTxt;
    let showtxt = dom.querySelector(".show_txt");
    while(h>3&&t){
        let step=1,m;
        if(t.indexOf("<br/>")==0){//回退的时候，如果碰到换行要整体替换
            step = 5;
        }else if(t.indexOf("<br>")==0){
            step = 4;
        }else if(m = t.match(/\[[^\]]+\]/)){// [大哭]是表情，统一回退
            step = m[0].length;
        }
        t = t.slice(0,-step);
        showtxt.innerHTML = t;
        h = getLength(dom.getRectsLength());
    }
    item.hideTxt =item.showTxt.slice(0,t.length);
    item.showTxt = t;
    dom.dataset.loaded = 1;
}
```
这个处理方式，最大的好处就是很精准，基本上完美达到产品的需求，不太足的地方就是需要回退字符串，频繁操作dom文本。

## 另种js实现
  上述实现是每个文字回退，尽管在实现中我们可以加速回退，但是实现性能终究不是太好，
  
## css的实现
css使用float来实现，如下：
![alt](https://img11.360buyimg.com/jdphoto/s766x1020_jfs/t1/36141/23/15206/420981/5d480cc1Eac0dba7e/ba372899b981fcb2.png)  

有个三个盒子 div，粉色盒子左浮动，浅蓝色盒子和黄色盒子右浮动。  
当浅蓝色盒子的高度低于粉色盒子，黄色盒子仍会处于浅蓝色盒子右下方。  
如果浅蓝色盒子文本过多，高度超过了粉色盒子，则黄色盒子不会停留在右下方，而是掉到了粉色盒子下。  
使用float浮动原理的表现形式巧妙的展现这个功能。  

```html
<div class="wrap">
    <div class="text">
        我是中国人我是中国人我是中国人我是中国人我是中国人我是中国人我是中国人我是中国人我是中国人我是中国人我是中国人我是中国人我是中国人我是中国人
    </div>
</div>
```
这里只有一个文本div，然后使用了before和after来实现，动态的出现“...更多”。
```css
.wrap{
    height: 40px;
    line-height: 20px;
    overflow: hidden;
}
.wrap .text{
    float:right;
    margin-left: -5px;
    width: 100%;
    word-break: break-all;
}
.wrap::before{
    float:left;
    width: 5px;
    content:'';
    height: 40px;;
}
.wrap::after{
    float: right;
    height: 20px;
    line-height: 20px;
    width:4em;
    content:'...更多';
    margin-left:-4em;
    position: relative;
    text-align: right;
    left:100%;
    top:-20px;
    padding-right: 5px;
    background:-webkit-gradient(linear,left top,right top,from(rgba(255,255,255,.7)),to(white))
}
```
注意这里需要指定line-height，代码中然后指定after的top为向上移动一行。
