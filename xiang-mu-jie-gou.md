# 项目结构

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

