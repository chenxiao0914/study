vue基本语法和概念；打包工具	webpack、gulp

vue.js只关注视图层

原生JS	-->	Jquery库	-->	前端模板引擎	-->	Angular.js/vue.js

框架：一套完整的解决方案；对项目的侵入性较大，如果更换框架，则需要重构

库：插件，体用一个小功能。

mvc：后端的分层开发概念

mvvm：前端视图层的概念，主要关注与视图层分离，分成三部分：Model、View、VM ViewModel；VM是V 和M 的调度者

<img src="D:%5Cagitstudy%5Cstudy%5Cfront-end%5Cvue%5Cimages%5Cmvvm.PNG" alt="mvvm"  />

~~~
开发版
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
生产版
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
~~~

# 一、基本指令

## 1、v-cloak、v-text、v-html、v-bind

~~~html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <script src="../lib/vue.js"></script>
    <style>
        [v-cloak] {
            display: none;
        }
    </style>
    <!-- v-cloak:解决插值表达式闪烁问题 
         v-text:没有闪烁问题,但是会替换标签原本的内容
         v-html:会替换标签原本的内容,同时会解析元素内容中的html标签元素
         v-bind:用于绑定属性,可简写成   :要绑定的属性
    -->
</head>
<body>
    <div id="app">
        <p v-cloak>+++{{ msg }}----</p>
        <h4 v-text="msg">======</h4>
        <div v-html="msg2">++++++</div>
        <input type="button" value="按钮" v-bind:title="title + '123'">
        <input type="button" value="按钮" :title="title">
    </div>
</body>

<script>
    new Vue({
        el: '#app',
        data: {
            msg: 'hello v-cloak',
            msg2: '<h1>hello v-html></h1>',
            title: '自定义标签'
        }
    });
</script>
</html>
~~~

## 2、v-on

监听事件	@mouseover 是简写

~~~html
<body>
    <div id="app">
        <input type="button" value="按钮" v-bind:title="title + '123'" v-on:click="show">
        <input type="button" value="按钮" :title="title" @mouseover="show1">
    </div>
</body>
<script>
    new Vue({
        el: '#app',
        data: {
            title: '自定义标签'
        },
        methods: {
            show:function(){
                alert('hello v-on:click');
            },
            show1:function(){
                alert('hello v-on:click1');
            }      
        }
    });
</script>
~~~

# 二、事件修饰符

stop、prevent、capture、self、once

stop：阻止冒泡

prevent：阻止 默认事件

capture：添加事件监听器时使用事件捕获模式（从外往内执行）

self：只当事件在该元素本事（比如不是子元素）触发时触发回调

once：事件只触发一次

## stop

~~~html
<body>
    <div id="app">
        <div class="inner" @click="innerHandle">
            <input type="button" value = "戳他" @click.stop="btnHandle">
        </div>
    </div>
</body>
<script>
    new Vue({
        el: '#app',
        data: {
            msg: ''
        },
        methods: {
            innerHandle(){
                alert("inner alert");
            },
            btnHandle(){
                alert("btn alert");
            }
        }    
    })
</script>
~~~

## prevent

~~~html
<a href="http:www.baidu.com" @click.prevent="clickLink">aaa</a>
~~~



## capture

点击按钮时，会先执行innerHandle方法

~~~html
 	<div id="app">
        <div class="inner" @click.capture="innerHandle">
            <input type="button" value = "戳他" @click="btnHandle">
        </div>
    </div>
~~~



## self

~~~html
	<div class="inner" @click.self="innerHandle">
        <input type="button" value = "戳他" @click="btnHandle">
    </div> 
~~~



## once

~~~html
    <div class="inner" @click.self.once="innerHandle">
        <input type="button" value = "戳他" @click="btnHandle">
    </div>
~~~

# 三、v-modle

v-model只能运用在表单元素中

~~~html
<body>
    <!-- v-bind 只能是实现数据的单向绑定，从 M 到 V 
        v-model 可实现数据的双向绑定，但是v-model只能运用在表单元素中
    -->
    <div id="app">
       <h4>{{ msg}}</h4>
       <input type="text" v-bind:value="msg">
       <input type="text" v-model="msg">
    </div>
</body>
<script>
    new Vue({
        el: '#app',
        data: {
            msg: 'qqqqqqqqq'
        },
        methods: {
            
        }    
    })
</script>
~~~

简易计算器

~~~html
<body>
    <div id="app">
        <input type="text" v-model="n1">
        <select v-model="opt">
            <option value="+">+</option>
            <option value="-">-</option>
            <option value="*">*</option>
            <option value="/">/</option>
        </select>
        <input type="text" v-model="n2">
        <input type="button" @click="cal" value="计算">
        <input type="text" v-model="result">
    </div>
</body>
    <script>
        new Vue({
            el:"#app",
            data:{
                n1:0,
                n2:0,
                opt:'+',
                result:0
            },
            methods:{
                cal(){
                    switch (this.opt) {
                        case '+':
                        this.result =  parseInt(this.n1) + parseInt(this.n2);
                            break;
                        case '-':
                        this.result =  parseInt(this.n1) - parseInt(this.n2);
                            break;
                        case '*':
                        this.result =  parseInt(this.n1) * parseInt(this.n2);
                            break;
                        case '/':
                        this.result =  parseInt(this.n1) / parseInt(this.n2);
                            break;
                    }
                }
            }   
        });
    </script>
~~~

