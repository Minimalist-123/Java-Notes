v-bind 主要用于属性绑定，比方你的class属性，style属性，value属性，href属性等等，只要是属性，就可以用v-bind指令进行绑定.这次主要介绍了VueJs中的V-bind指令,需要的朋友可以参考下


v-bind 主要用于属性绑定，Vue官方提供了一个简写方式 :bind，例如：

<!-- 完整语法 -->
<a v-bind:href="url"></a>
<!-- 缩写 -->
<a :href="url"></a>
一、概述
v-bind 主要用于属性绑定，比方你的class属性，style属性，value属性，href属性等等，只要是属性，就可以用v-bind指令进行绑定。
示例：

<!-- 绑定一个属性 -->
<img v-bind:src="imageSrc">
<!-- 缩写 -->
<img :src="imageSrc">
<!-- 内联字符串拼接 -->
<img :src="'/path/to/images/' + fileName">
<!-- class 绑定 -->
<div :class="{ red: isRed }"></div>
<div :class="[classA, classB]"></div>
<div :class="[classA, { classB: isB, classC: isC }]">
<!-- style 绑定 -->
<div :style="{ fontSize: size + 'px' }"></div>
<div :style="[styleObjectA, styleObjectB]"></div>
<!-- 绑定一个有属性的对象 -->
<div v-bind="{ id: someProp, 'other-attr': otherProp }"></div>
<!-- 通过 prop 修饰符绑定 DOM 属性 -->
<div v-bind:text-content.prop="text"></div>
<!-- prop 绑定。“prop”必须在 my-component 中声明。-->
<my-component :prop="someThing"></my-component>
<!-- 通过 $props 将父组件的 props 一起传给子组件 -->
<child-component v-bind="$props"></child-component>
<!-- XLink    欢迎加入全栈开发交流圈一起吹水聊天学习交流：864305860-->
<svg><a :xlink:special="foo"></a></svg>
二、绑定 HTML Class
对象语法
我们可以传给 v-bind:class 一个对象，以动态地切换 class

<div v-bind:class="{ active: isActive }"></div>
上面的语法表示 active 这个 class 存在与否将取决于数据属性 isActive 的 truthiness
可以在对象中传入更多属性来动态切换多个 class。此外，v-bind:class 指令也可以与普通的 class 属性共存。当有如下模板:

<div class="static"
  v-bind:class="{ active: isActive, 'text-danger': hasError }">
</div>
  和如下 data 
data: {
 isActive: true,
 hasError: false
}//欢迎加入全栈开发交流圈一起吹水聊天学习交流：864305860
结果渲染为：

<div class="static active"></div>
当 isActive 或者 hasError 变化时，class 列表将相应地更新。例如，如果 hasError 的值为 true，class 列表将变为 "static active text-danger"
绑定的数据对象不必内联定义在模板里

<div v-bind:class="classObject"></div>
data: {
 classObject: {
 active: true,
 'text-danger': false
 }
}//欢迎加入全栈开发交流圈一起吹水聊天学习交流：864305860
渲染的结果和上面一样。我们也可以在这里绑定一个返回对象的计算属性。这是一个常用且强大的模式：

<div v-bind:class="classObject"></div>
 
data: {
 isActive: true,
 error: null
},
computed: {
 classObject: function () {
 return {
  active: this.isActive && !this.error,
  'text-danger': this.error && this.error.type === 'fatal'
 }//欢迎加入全栈开发交流圈一起学习交流：864305860
 }//面向1-3年前端人员
}//帮助突破技术瓶颈，提升思维能力
数组语法
我们可以把一个数组传给 v-bind:class，以应用一个 class 列表

<div v-bind:class="[activeClass, errorClass]"></div>
data: {
 activeClass: 'active',
 errorClass: 'text-danger'
}
渲染为：

<div class="active text-danger"></div>
如果你也想根据条件切换列表中的 class，可以用三元表达式

<div v-bind:class="[isActive ? activeClass : '', errorClass]"></div>
这样写将始终添加 errorClass，但是只有在 isActive 是 truthy 时才添加 activeClass。
不过，当有多个条件 class 时这样写有些繁琐。所以在数组语法中也可以使用对象语法

<div v-bind:class="[{ active: isActive }, errorClass]"></div>
三、用在组件上
当在一个自定义组件上使用 class 属性时，这些类将被添加到该组件的根元素上面。这个元素上已经存在的类不会被覆盖。
例如，如果你声明了这个组件：

Vue.component('my-component', {
 template: '<p class="foo bar">Hi</p>'
})
然后在使用它的时候添加一些 class

<my-component class="baz boo"></my-component>
HTML 将被渲染为:

<p class="foo bar baz boo">Hi</p>
对于带数据绑定 class 也同样适用

<my-component v-bind:class="{ active: isActive }"></my-component>
当 isActive 为 truthy时，HTML 将被渲染成为

<p class="foo bar active">Hi</p>
四、绑定内联样式
对象语法
v-bind:style 的对象语法十分直观——看着非常像 CSS，但其实是一个 JavaScript 对象。CSS 属性名可以用驼峰式 (camelCase) 或短横线分隔 (kebab-case，记得用单引号括起来) 来命名：

<div v-bind:style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>
data: {
 activeColor: 'red',
 fontSize: 30
}//欢迎加入全栈开发交流圈一起学习交流：864305860
直接绑定到一个样式对象通常更好，这会让模板更清晰

<div v-bind:style="styleObject"></div> 
data: {
 styleObject: {
 color: 'red',
 fontSize: '13px'
 }
}//欢迎加入全栈开发交流圈一起吹水聊天学习交流：864305860
同样的，对象语法常常结合返回对象的计算属性使用
数组语法
v-bind:style 的数组语法可以将多个样式对象应用到同一个元素上

<div v-bind:style="[baseStyles, overridingStyles]"></div>
结语

感谢您的观看，如有不足之处，欢迎批评指正。

本次给大家推荐一个免费的学习群，里面概括移动应用网站开发，css，html，webpack，vue node angular以及面试资源等。
对web开发技术感兴趣的同学，欢迎加入Q群：864305860，不管你是小白还是大牛我都欢迎，还有大牛整理的一套高效率学习路线和教程与您免费分享，同时每天更新视频资料。
最后，祝大家早日学有所成，拿到满意offer，快速升职加薪，走上人生巅峰。

作者：前端攻城小牛
链接：https://www.jianshu.com/p/067223608b08
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
