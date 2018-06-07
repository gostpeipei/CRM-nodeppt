#### settings
```markdown
title: CRM项目串讲
speaker: 张宝晨
theme: moon
```

[slide]
# CRM项目串讲
### by: 张宝晨

[slide]
## 主要内容
----
* 项目流程简介
* 项目技术栈
* 项目代码结构目录
* 内部封装插件的使用
* 项目开发/测试/上线流程概述
* 存在的问题
* 可优化部分

[slide]
<h2><span class="yellow">1.项目流程简介</span></h2> 

[slide]
## CRM: Customer Relationship Management - 客户管理系统
### 平台功能：营销和销售阶段的客户关系管理

[slide]
## 过程概述
1. 线索录入(目标客户群的信息), 客服跟进<i class="fa fa-arrow-down"></i><br> {:&.rollIn}
  - 行业、联系方式、来源、状态等
2. 直销、渠道人员介入, 创建商机 <i class="fa fa-arrow-down"></i><br>
  - 目标客户转为意向客户
3. 签订合同, 成为客户 <i class="fa fa-arrow-down"></i>
4. 建立各种订单<br>
  - 包括：充值、领款、退款、返现、返货...


[slide]
<h3 style="text-align: left;">其他一些其他独立模块</h3>

* 运营管理模块：管理各种类型的广告项目
* 业务管理模块：线索废弃库、垫款资金池、代理商...
* 报表管理模块：查询和导出相关报表 (客户消耗、销售业绩、资金池...)
* 系统管理模块：人员管理、权限分配等

[slide]
<h2><span class="yellow">2.项目技术栈</span></h2> 

[slide]
## `前端主要技术栈`

> CRM: 目前项目未采用前后端分离，页面采用服务端渲染，前端依赖Node环境，后端接口也由Node编写，使用express框架。


---
分类| 应用| 备注
:-------|:------:|
模板引擎 | swig | 配合路由，由后端返回对应html页面及相关css、js文件
路由 | express.Router() | 
构建工具 | gulp | 代码编译压缩打包生成一条龙
预编译 | less | 会由gulp脚本编译成css
代码规范检测 | jshint | 代码规范检测
第三方库/工具 | jQuery/Bootstrap | 
非第三方工具 | iTable/iFilter/MJJS | 内部封装的插件/方法

[slide]
<h2><span class="yellow">3.项目代码结构目录</span></h2>

[slide]
<pre><code class="markdown">/* 项目一级目录 */
  |-- bin/www（脚本入口文件）
  |-- common/MJJS.js（存放了配置路由时封装的一些方法）
  |-- node_modules/（使用npm包管理工具安装的模块）
  |-- package.json (npm包配置文件)
  |-- public/（静态资源文件,包括样式,字体文件,图片,js）
    |-- css/ (less编译后的样式文件)
    |-- file/ (一些表格文件模板)
    |-- font/ (项目里用到的图标字体文件)
    |-- fonts/ (项目里用到的图标字体文件，似乎没用到)
    |-- img/ (项目中的图片文件)
    |-- js/ (项目所有页面对应的js文件、静态资源、封装的方法等)
      |-- lib/ (jQuery,bootstrap,MJJS)
      |-- util/ (jQuery插件, bootstrap插件, iTable, iFilter等)
    |-- less (样式文件)
  |-- routes/ (存放对应路由页面的js文件)
  |-- views/ (存放对应路由页面的html文件)
  |-- config.js (项目环境配置文件)
  |-- config.route.js (项目路由配置文件)
  |-- .jshintrc (js代码规范检测工具配置文件)
  |-- app.js (包含node express框架的一些配置项)
  |-- gulpfile.js (gulp打包任务脚本)
</code>
</pre>



[slide]
<h2><span class="yellow">4.内部封装插件的使用(MJJS, iFilter, iTable)</span></h2>

[slide]
## MJJS.js
> MJJS对原生js的String/Number/Array/Date等内置对象进行了扩展，封装了项目中用到的各种可复用的方法(页面加载、插件初始化等)，是项目的核心文件。

[slide]
## iFilter.js [根据需要加载搜索栏的过滤选项]
<pre>
<code class="javascript">/* Step 1 */
//在MJJS.form中去扩展了iFilter的方法, 初始化搜索栏   MJJS.js--1318行
MJJS.form.filter = function(parent, opts) {
  if (typeof(parent)==='string'&&parent[0]==='#'&&$(parent).length===1&&opts) {
    MJJS.load(['iFilter'], function() {
      $(parent).iFilter(opts);
    });
  }
}
</code>
</pre>

[slide]
## iFilter.js [根据需要加载搜索栏的过滤选项]
<pre>
<code class="javascript">/* Step 2 */
// 先在页面中定义ID的最外层元素 &lt;div id="advSearch" class="adv-search" &gt;&lt;/div&gt;
MJJS.form.filter('#advSearch', {
  cacheKey: 缓存对应的key，自行设定，比如充值单的key就是prepaidIndex，
            获取对应列表的缓存数据,
  cacheAble: 是否开启缓存, 默认关闭,
  loadCacheDataFlag: 是否读取本地缓存的数据，有的从其他页面跳转到当前页面，
                     会带入一些参数并查询，此时不需要读取本地的数据，可以设置为false,
  params: [筛选时用到的组件],
  load: function(o) {
    // 页面加载时的回调
  },
  reload: function(o) {
    // 每次加载后的回调，比如改变筛选项后的回调函数
  }
});

//之后页面的搜索栏会自动渲染出来
</code>
</pre>

[slide]
## iTable.js [根据需要加载table列表]
<pre>
<code class="javascript">/* Step 1 */
//在MJJS.ui中去扩展了iTable的方法, 初始化搜索栏   MJJS.js--1439行
MJJS.ui.iTable = function(parent, opts) {
  if (typeof(parent)==='string'&&parent[0]==='#'&&$(parent).length===1&&opts) {
    MJJS.load(['iTable'], function() {
      $(parent).iTable(opts);
    });
  }
}
</code>
</pre>

[slide]
## iTable.js [根据需要加载table列表]
<pre>
<code class="javascript">/* Step 2 */
// 先在页面中定义ID的外层table元素 &lt;table id="advTable"&gt;&lt;/table&gt;,并且写死thead
MJJS.ui.iTable('#advTable', {
  url: //接口地址,
  searchEmpty: //没有数据时候展示的内容,
  default: MJJS.common.defaultTable //定义了接口返回字段的成功状态码,分页信息,数据读取路径
  columns: [/* 列表中每一列需要渲染的数据的字段名称 */],
  render: {
    0:列表中某一列需要在接口返回的数据基础上,进行的一系列操作,转换等,可以写在render中,render的优先级高于columns,
    1:,
    2:
  },
  data: function(o){
    //除页码和页面长度的信息之外,需要另外传给后台接口的字段,放在data函数中的参数里
    MJJS.data.objMergeTable(_table, cluePar);
  },
  load: function(table) {
    //页面加载时的回调;
  },
  error: function(err) {
    //发生错误时的回调
  }
});
</code>
</pre>

[slide]
<h2><span class="yellow">5.项目开发/测试/上线流程概述</span></h2>

[slide]
<h3 style="text-align: left;">代码仓库: <span class="yellow" style="font-size: 0.8em;">http:&frasl;&frasl;stash.weimob.com/scm/mjad/mjad_crm.git</span></h3>
<h3 style="text-align: left;">后台接口: <span class="yellow" style="font-size: 0.8em;">http:&frasl;&frasl;115.159.219.170:8081/swagger-ui.html</span></h3>
<h3 style="text-align: left;"> 生产环境: <span class="yellow" style="font-size: 0.8em;">http:&frasl;&frasl;crm.mjmobi.com</span></h3>
<h3 style="text-align: left;"> dev环境: <span class="yellow" style="font-size: 0.8em;">http:&frasl;&frasl;mjcrm-dev.weimob.com </span></h3>
<h3 style="text-align: left;"> 本地环境: <span class="yellow" style="font-size: 0.8em;">http:&frasl;&frasl;localhost:3014</span></h3>

<h3 style="text-align: left;">`开发流程:`</h3>
1. stash仓库clone代码
2. 建立自己的分支
3. 在自己的分支上提交代码
4. 之后再merge到master分支上
5. 测试环境 --> 使用Jenkins发布master分支到测试环境
6. 正式环境 --> 在online分支下，gulp打包压缩后生成build文件夹 


[slide]
<h2><span class="yellow">6.目前存在的问题及可优化部分</span></h2>

[slide]
<h3 align="left"><code>问题A</code> 资源依赖混乱</h3>

<p align="left">
<span style="width:10%;text-align:left;"><strong>举例: </strong></span>
<span style="width:80%; display: inline-block;vertical-align: top;text-align: left;">echart/highchart本质上功能一样,项目中两个都有引入,并且重复引用,lib/和util/custom下存在多个</span>
</p>

<p align="left">
<span style="width:10%;text-align:left;"><strong>原因: </strong></span>
<span style="width:80%; display: inline-block;vertical-align: top;text-align: left;">项目前后许多人接手,各有各的风格,没有形成统一规范</span>
</p>

<p align="left">
<span style="width:10%;text-align:left;"><strong>解决方案: </strong></span>
<span style="width:80%; display: inline-block;vertical-align: top;text-align: left;">制定严格的规范, 如:给定外部依赖静态资源固定位置</span>
</p>

[slide]
<h3 align="left"><code>问题B</code> 部分代码变量命名不方便阅读</h3>
<pre><code style="max-height: 400px;"><strong>原始代码: </strong>
MJJS.namespace = function(name, sep) {
  var s = name.split(sep || '.'),
    d = {},
    o = function(a, b, c) {
      if (c < b.length) {
        if (!a[b[c]]) {
          a[b[c]] = {};
        }
        d = a[b[c]];
        o(a[b[c]], b, c + 1);
      }
    };
  o(window, s, 0);
  return d;
};
<strong>更改后代码: </strong>
MJJS.namespace = function(name, sep) {
  var nameArr = name.split(sep || '.'),
    obj = {},
    process = function(global, nameArr, startIndex) {
      if (startIndex < nameArr.length) {
        if (!global[nameArr[startIndex]]) {
          global[nameArr[startIndex]] = {};
        }
        obj = global[nameArr[startIndex]];
        process(global[nameArr[startIndex]], nameArr, startIndex + 1);
      }
    };
  process(window, nameArr, 0);
  return obj;
};
</code></pre>
<p align="left">
<span style="width:10%;text-align:left;"><strong>原因: </strong></span>
<span style="width:70%; display: inline-block;vertical-align: top;text-align: left;">书写习惯？编写方便？</span>
<!-- <img src="/imgs/issue_b.png" height="80" style="position:absolute;"> -->
</p>
<p align="left">
<span style="width:10%;text-align:left;"><strong>解决方案: </strong></span>
<span style="width:80%; display: inline-block;vertical-align: top;text-align: left;">尽量使用语义明确的变量命名</span>
</p>

[slide]
<h3 align="left"><code>问题C</code> 第三方静态资源和项目页面对应的js混在一起不容易区分</h3>

<p align="left">
<span style="width:10%;text-align:left;"><strong>原因: </strong></span>
<span style="width:60%; display: inline-block;vertical-align: top;text-align: left;">一直都是这样写，没时间去整理</span>
<img src="/imgs/issue_c.png" height="250" style="position:absolute;">
</p>
<p align="left">
<span style="width:10%;text-align:left;"><strong>解决方案: </strong></span>
<span style="width:55%; display: inline-block;vertical-align: top;text-align: left;">可尝试讲所有页面js在统一放在一层文件夹中,如：public/js/pages/</span>
</p>

[slide]
<h3 align="left"><code>问题D</code> 没有严格按照jshint语法检测标准书写代码</h3>
<p align="left">
<span style="width:10%;text-align:left;"><strong>举例: </strong></span>
<span style="width:60%; display: inline-block;vertical-align: top;text-align: left;">判断相等时最，都是用的==，不是===</span>
<img src="/imgs/issue_d.png" height="250" style="position:absolute;">
</p>
<p align="left">
<span style="width:10%;text-align:left;"><strong>原因: </strong></span>
<span style="width:60%; display: inline-block;vertical-align: top;text-align: left;">jshint报错不影响代码执行，被忽略</span>
</p>
<p align="left">
<span style="width:10%;text-align:left;"><strong>解决方案: </strong></span>
<span style="width:55%; display: inline-block;vertical-align: top;text-align: left;">手动修改</span>
</p>

[slide]
<h3 align="left"><code>问题E</code> ES6没有在项目中很好的使用 (已解决)</h3>
<p align="left">
<span style="width:10%;text-align:left;"><strong>原因: </strong></span>
<span style="width:60%; display: inline-block;vertical-align: top;text-align: left;">使用es6,gulp打包会报错</span>
<!-- <img src="/imgs/issue_e.png" height="250" style="position:absolute;"> -->
</p>
<p align="left">
<span style="width:10%;text-align:left;"><strong>解决方案: </strong></span>
<span style="width:60%; display: inline-block;vertical-align: top;text-align: left;">使用gulp-uglifyes包代替gulp-uglify包</span>
</p>
<p align="left">
<img src="/imgs/issue_e_2.png" width="400">
<img src="/imgs/issue_e.png" width="400">
</p>

[slide]
## Thanks!
