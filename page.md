## 概述
主文档跟shadow DOM 的结构关系，可以用MDN上一张图来表示：
![shadowdom.svg](https://km.woa.com/gkm/api/img/cos-file-url?url=https%3A%2F%2Fkm-pro-1258638997.cos.ap-guangzhou.myqcloud.com%2Ffiles%2Fphotos%2Fpictures%2F202202%2F1645581743-0871-621595af15456-926489.svg&is_redirect=1)
- Document Tree：主文档（其实，如果存在shadow Tree 嵌套的情况，也可以理解为上一级的 dom tree）；
- Shadow Tree： shadow DOM 构成的dom树；  
- Flattened Tree： 浏览器渲染时候的 dom 树；  
## 一、默认表现：
1. 主文档与 shadow tree 中的样式，是互不影响的；
2. 主文档中，应用在 shadow host 节点上的可以被继承的样式，也同样被shadow tree 继承。由于 shadow tree 是需要挂载在一个节点，也就是 shadow host，它本身会继承主文档的一些基础样式（比如 font，background ，color等），对于 shadow tree 来讲，shadow host 节点就是它们的根，可以理解为shadow tree 的 “html”根元素，因此，也符合dom 的样式继承规则。

代码：
```html
<style type="text/css">
    body {
        background: #ccc;
    }
    h1 {
        color: red;
    }
</style>
<body>
  <!-- background 将应用 body 样式-->
  <shadow-sample></shadow-sample>
  <!-- shadow host 是 h1, font-size/color 继承了 h1 的样式，字体红色。-->
  <h1>
      <shadow-sample></shadow-sample>
  </h1>
  <script>
      customElements.define('shadow-sample', class extends HTMLElement {
        connectedCallback() {
          this.attachShadow({mode: 'open'});
          this.shadowRoot.innerHTML = `
              <span>span 元素</span>
          `;
        }
      });
  </script>
</body>        
```
完整举例，可以参考链接: [shadown使用](https://shufengwu.pages.woa.com/shadowdom/index.html)

## 二、:host 控制
1. 由于继承的原因，样式没有做到完全隔离，因此提供 :host 选择器，代表shadow tree 的根节点（shadow root），可以通过它重置顶层的基础样式。
2. 语法：:host(compound-selector), compound-selector 是常规选择器的一个子集[参考链接](https://developer.mozilla.org/en-US/docs/Web/CSS/:host) 。

代码：
```html
<body>
<!-- :host  选中所有节点-->
<shadow-sample></shadow-sample>
<!-- :host(.reset-bg)  选中该节点-->
<shadow-sample class="reset-bg"></shadow-sample>
<!-- :host([reset-font-color])  选中该节点-->
<shadow-sample reset-font-color></shadow-sample>
</body>
```
```javascript
	customElements.define('shadow-sample', class extends HTMLElement {
	  connectedCallback() {
	    this.attachShadow({mode: 'open'});
	    this.shadowRoot.innerHTML = `
	    	<style>
	    		:host {
	    			all: initial; /* 重置为浏览器默认 */
	    		}
	    		:host(.reset-bg){
	    			/* 自定义元素默认是dispaly:inline,设置为block，才能正常包裹区域元素。 */
	    			display: block; 
	    			background: yellow;
	    			border: 1px solid red;
	    		}
	    		:host([reset-font-color]) {
	    			font-size: 16px;
	    		}
	    		:host([reset-font-color]) span{
	    			font-size: 12px;
	    			color: blue;
	    		}
	    		span {
	    			color: green;
	    		}
	    	</style>
		    <h1>H1元素</h1>
        <h2>H2元素</h2>
        <span>span 元素(字体/颜色)</span>
        <button>click button</button>
	    `;
	  }
	});
	</script>	
```
```
更多 :host 选择器样例
```css
	:host {
		opacity: 0.4;
		will-change: opacity;
		transition: opacity 300ms ease-in-out;
	}
	:host(:hover) {
		opacity: 1;
	}
	:host([disabled]) { /* style when host has disabled attribute. */
		background: grey;
		pointer-events: none;
		opacity: 0.4;
	}
	:host(.blue) {
		color: blue; /* color host when it has class="blue" */
	}
	:host(.pink) > #tabs {
		color: pink; /* color internal #tabs node when host has class="pink". */
	}
```
完整举例，可以参考链接: [:host使用](https://shufengwu.pages.woa.com/shadowdom/host.html)

## 三、slot 控制
1. slot 标签的内容，是在主文档定义的，所以它的样式应该是受到主文档的控制。
	- 相对于 shadow，slot里面的dom，我们称其为 light DOM。
	- light DOM 从效果上看，就像是 shadow DOM对它们的 引用。DOM结构被组织在shadow tree里面了。
2. shadow dom 如果需要更改 slot 内容的样式，可以有三种方式：
	- slot([name="xxx"]) —— name 属性控制，只针对slot节点本身；
	- ::slotted (compound-selector) —— 伪类选择器，只针对slot节点本身；
	- :host-context（compound-selector）—— 区分host节点所在的上下文，selector 通常是简单的 class 或者 标签，指向 shadow host 的某一级的父节点。 

代码：
```html
<!-- 具名 slot 的情况-->
<user-card>
  <!-- slot的内容，样式由主文档决定。-->
  <div slot="userName">
    <h1 class="name">H1元素</h1>
  </div>
  <h2>H2元素</h2>
  <div slot="ext" class="extCls">
    <span>span 元素</span>
    <button>click button</button>
    hello world.
  </div>
</user-card>
```
slot 选择器
```css
/* 控制所有slot元素 */
slot{
  display: block;
  background: pink;
}
/* 控制name属性为ext的slot元素 */
slot[name="ext"]{
  display:block;
  background: #fff;
}
/* DOESN'T WORK. 不支持选择子元素 */
slot[name="ext"] span {
  color: yellow;
}
```
::slot 选择器
```css
  ::slotted(*){
    display: block;
    background: pink;
  }
  /* selector 支持顶部 class */
  ::slotted(.extCls){ /* class 选择器权重大于类型 */
    background: #fff;
    border:1px solid red; 
  }
  /* selector 也支持顶部标签 */
  ::slotted(div){
    background: #00f;
  }

  /* DOESN'T WORK. 不支持选择 light DOM 子元素 */
  ::slotted(.extCls) button{
    color: blue;
  }

  /* DOESN'T WORK. 选择器不支持选择子元素 */
  ::slotted(.extCls button) {
    color: blue;
  }
```
:host-context 选择器：
```css
  :host-context(body.light-theme) slot{
    display: block;
    background: #fff;
    color: black;
  }
  /* 权重小于class */
  :host-context(div) slot {
    background: #ff0;
  }
```
```javascript
      customElements.define(
        "user-card",
        class extends HTMLElement {
          connectedCallback() {
            this.attachShadow({
              mode: "open",
            });
            this.shadowRoot.innerHTML = `
	    	  <style>
            ...
				  </style>
				  <div class="userInfo">
				  	<slot name="userName"></slot> 
				  	<slot></slot>
				  	<slot name="ext"></slot> 
				  </div>
	    `;
          }
        }
      );
```
完整举例，可以参考链接: [slot使用](https://shufengwu.pages.woa.com/shadowdom/slot.html)

## 四、css 变量传值
由于 在主文档中 css 变量 的定义，和shadow tree 是共享的，可以通过 css 变量的方式，提供一些基础的共享控制。
1. shadow DOM  可以继承主文档的 css 变量；
2. shadow DOM 可以重新定义/覆盖 主文档的 css变量，但是只影响自己，不影响主文档。

代码：
```css
/* 主文档变量定义 */
body {
  background: #fff;

  --field-font-color: green;
  --field-bg-color: #ccc;
}
span {
  color:  var(--field-font-color);
}
/* shadow dom 的定义 */
  :host {
    all: initial;
    /* 覆盖主文档的变量 */
    --field-font-color: red; 
    --local-field-font-color: red;
  }
  :host {
    /* 自定义元素默认是dispaly:inline,设置为block，才能正常包裹区域元素。 */
    display: block; 
    background: var(--field-bg-color, red); /* 可以直接引用主文档的css 变量 */
  }
  span {
    color: var(--field-font-color, blue); 
  }
  button {
    color: var(--local-field-font-color, blue); 
  }
```
完整举例，可以参考链接: [var使用](https://shufengwu.pages.woa.com/shadowdom/vars.html)

## 五、: :part()选择器
1. ::part(<ident>+ ) 提供了一种在主文档直接 控制 shadow DOM 元素的途径。shadow DOM可以在元素上面添加part属性并设置值，从而暴露该接口。
	- 语法： <TAG part="value1 value2"></TAG>，多个 value 可以用空格分隔。
	- <ident>+ 是一个属性值，大小写敏感；
	- 支持控制伪类；
2. 如果存在shadow DOM 嵌套 的情况，在子shadow DOM，可以用 exportparts 声明导出到顶层
	- 语法：<child-dom exportparts="originPartVal:  exportPartVal"></child-dom>，其中，originPartVal是在 child-dom定义 part值， exportPartVal 是重新export 到外层的新名称。
	- 如果originPartVal 跟 exportPartVal  一致，则可以简写为 <child-dom exportparts="originPartVa"></child-dom>
	- 多个exportVal，用逗号分隔.<child-dom exportparts="export1, export2"></child-dom>
	

代码：
```css
		/* 选择所有shadow tree 里面的part=text的节点 */
		::part(text) {
			color: blue;
		}
    /* 区分大小写 */
		::part(Text) {
			color: brown;
		}
		/* 可以指定自定义标签，也可以设置伪类 */
		shadow-sample::part(Text):hover {
			text-decoration: underline;
		}
		/* NOT WORKS! 只支持目标元素，不支持选择子元素 */
		shadow-sample::part(ext) span {
			color:  blue;
		}
```
```html
<!-- shadow DOM 嵌套情况-->
<shadow-sample-outer></shadow-sample-outer>
```    
```javascript
	customElements.define('shadow-sample-outer', class extends HTMLElement {
	  connectedCallback() {
	    this.attachShadow({mode: 'open'});
	    // 导出子dom tree 的part属性
	    this.shadowRoot.innerHTML = `
	    	<shadow-sample-inner exportparts="innerspan: text,Text"></shadow-sample-inner>
	    `;
	  }
	});

	customElements.define('shadow-sample-inner', class extends HTMLElement {
	  connectedCallback() {
	    this.attachShadow({mode: 'open'});
	    this.shadowRoot.innerHTML = `
        ...
        <span part="innerspan">span 元素</span>，<span part="Text">another span 元素</span>
        ...
	    `;
	  }
	});
```
完整举例，可以参考链接: [::part使用](https://shufengwu.pages.woa.com/shadowdom/part.html)
	
	### 附录
- https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_shadow_DOM
- https://caniuse.com/?search=%3A%3Apart
- https://javascript.info/shadow-dom-style
- https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_templates_and_slots
- https://developers.google.com/web/fundamentals/web-components/shadowdom
- https://developer.mozilla.org/en-US/docs/Web/CSS/:host-context()
- https://dev.to/webpadawan/css-shadow-parts-are-coming-mi5