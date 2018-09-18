# 多页面应用到多包独立应用

## 多页与多包的区别

多页面应用实际上是运用了公用组件、方法、资源的一个传统web项目

多包独立应用实际上是综合上面有利元素的情况下，通过一次构建，生成数个完全独立的web项目

{% hint style="info" %}
之所以会存在多包，是因为我们自己项目实际具体的需求

具体从项目代码来说
{% endhint %}



## 目录结构

```text
|---build
|---config
|---dist
|---src
    |---assets    #资源
         |---css
             |---main.css  #公共css
             |---1px.css  #定制颜色的1px
             |---main.less  #公共less及vux样式配置
         |---font    #字体图标
         |---js
             |---main.js    #公共方法及配置
    |---components #组件
         |---cardGrid.vue  #主页组件
         |---cardDetailsInfo.vue  #类似交易清单，审核详情等一系列的公共组件
         |---checkCellList.vue  #类似待审核列表的公共组件
    |---views    #各个页面模块
         |---Home    #一级目录
            |---home    #二级目录
                 |---home.html
                 |---home.js
                 |---index.vue
         |---Inquiry    #一级目录
            |---account    #二级目录
                 |---account.html
                 |---account.js
                 |---accountInquiry.vue	
            |---details    #二级目录--带路由
                 |---details.html
                 |---details.js
                 |---detailsInfo.vue	
                 |---detailsIquiry.vue
                 |---index.vue
          ......
```

## Dist结构

常见多页面应用dist结构

```text
|---dist
    |---assets    #资源
         |---css
             |---commons
                 |---main.css 
             |---Home
                 |---XX.css 
             |---inquiry-account
                 |---XX.css 
             |---inquiry-details
                 |---XX.css
             ...
         |---font
         |---img
         |---js
             |---commons
                 |---main.js 
             |---Home
                 |---XX.js 
             |---inquiry-account
                 |---XX.js 
             |---inquiry-details
                 |---XX.js
             ...
    |---views    #页面模块
            |---Home
                 |---XX.html
            |---inquiry-account
                 |---XX.html
            |---inquiry-details
                 |---XX.html
            ...
```

企业银行dist结构

```text
|---dist
    |---commons    #公共资源
         |---font
         |---img
         ...
    |---views    #页面模块
         |---Home
            |---index.html
            |---index.a58e808f5780d4eeb40ffab7ec725bf7.css
            |---index.d7beb08946489d74630c.js
         |---inquiry-account
            |---index.html
            |---index.a58e808f5780d4eeb40ffab7ec725bf7.css
            |---index.d7beb08946489d74630c.js
         |---inquiry-details
            |---index.html
            |---index.a58e808f5780d4eeb40ffab7ec725bf7.css
            |---index.d7beb08946489d74630c.js
         ...
```

