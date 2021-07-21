# 示例项目

1）该示例以vue2.5.2和element-ui2.3.4搭建。<br/>
2）该项目开发时依赖node.js。<br/>
3）如果项目没有node_modules目录则需要通过npm install命令联网安装，或者其他方式获取后放到项目中。

## 构建命令

``` bash
# 安装第三方依赖
npm install

# 启动热部署开发模式，访问localhost:8080
npm run dev

# 构建打包压缩文件
npm run build

# 构建打包并查询模块分析报告
npm run build --report
```

## 项目说明

### 1.目录结构
```javascript
    -build                  // webpack相关配置文件目录
    -config                 // webpack相关配置文件目录
    -dist                   // webpack打包生成目录
    -node_modules           // 第三方模块目录，如果项目没有该目录则需要通过npm install命令联网安装，或者其他方式获取后放到项目中
    -src                    // 源码目录
        -components         // 业务代码页面组件目录
            -home           // 首页目录
            -login          // 登录页目录
            -main           // 三分页结构目录
            -organizational // 业务代码示例
            -system         // 业务代码示例
        -config             // 配置文件目录
        -locales            // 国际化文件目录
        -mock				// 模拟数据设置目录，当配置文件中mock为true时根据mock目录下的配置返回模拟数据
        -router             // 路由配置文件目录
        -static             // 静态文件目录，该目录下/js文件夹不会构建打包，直接复制到dist相应目录下
        -store              // 状态管理文件目录
        -utils              // 工具类目录，可导入后直接使用
        	-AuthUtil.js	// 提供了一些权限相关常用方法。
        	-Cube.js		// 提供了一些数据类型的常用方法，该工具类中的方法通过全局对象cube调用。
        	-DataHandle.js	// 提供了一些模型对象扩展方法。
        	-RESTClient.js  // 提供了基于axios的ajax请求方法。
        	-EntityContainer.js //提供了前后端交互数据格式解析。
```    
### 2.数据交互
```javascript
    1）在业务模块中引入DataHandle模块
        import DataHandle from '@/utils/DataHandle'

    2）在data()方法的return中添加：
        //配置数据加载信息
    	dataOption: {
	    	name: "tableData", //数据返回后要赋值的对象key
	    	searchName: "searchData", // 查询框绑定的数据对象key
	    	url: $config.gatewayURL + "/employee/", //请求url，删除数据都会以此url为基础自动追加delete
	    	primaryKey: "id", //主键key，默认为id
    		autoLoad: true //是否自动加载数据，默认true
        }
        
         /*
         dataOption可设置属性有：
            name                                数据返回后要赋值的对象key
	    	searchName                          查询框绑定的数据对象key
	    	url                                 请求url，删除数据都会以此url为基础自动追加delete
	    	primaryKey                          主键key，默认为id
    		autoLoad                            是否自动加载数据，默认true
            pageSize                            每页显示记录数
            pageIndex                           表示当前第几页
            itemCount                           总记录数，查询完成会自动赋值
            actions                             一个对象，改变url为基础后面追加的部分，key分别为：data、save、remove，值对应要追加的部分
            timeout                             请求的默认超时时间，以毫秒为单位，默认：60000
            customHeaders                       自定义请求的header
            loadByPost                          获取列表数据是否使用post方式提交查询服务，true：post方式 false：get方式，默认false
        */

    3）在声明周期的created方法中添加以下代码：
        var dataHandle = new DataHandle();
    	//初始化时会为this新增常用方法，以及根据dataOption加载数据
        dataHandle.init(this);

        /*初始化时为this新增常用方法有：
		submit(); 								提交表单数据
		currentChange;  						分页跳转
		search();								查询
		remove(items);							删除
		load();									加载数据
		showDialog(p_url, p_params, p_width);	显示模态弹出框
		hideDialog();							隐藏模态弹出框
		addTab(p_name, p_route, p_params);		添加选项卡
        msg(p_key, p_args);						获取国际化信息
        */

```    
### 3.权限控制
    1) 登录
        在src/config/index.js中可通过config.isRemote属性配置是否开启远程登录、退出及菜单获取。
        访问页面时，如果localStorage中没有token则跳转至登录页。
        点击登录按钮，从后台获取token、userName和菜单信息，存储在localStorage中，跳转至首页。
    2）菜单
        菜单信息在登录时进行获取，存储在store和localStorage中，当刷新页面时从localStorage中获取菜单设置到store中。
    3）路由。
        路由需在router下进行配置，路由使用懒加载方式配置组件，以便构建时代码分割。
        在路由跳转时，在路由守卫中判断是否登录，通过和菜单path进行对比判断使用具有权限。
    4) 请求
        请求后端数据时，src/utils/RESTClient会获取localStorage中的token设置在请求header中。
        如果后端返回header中sessionstate的值为timeout则登录过期，将清除localStorage中的token，并刷新页面跳转至登录页。
        如果后端返回值为authorizeFail，则提示没有访问权限。
    5）退出
        退出时清除token和菜单信息跳转时登录页。

### 4.传输加密
    1）可在src/config/index.js中配置加解密方法，在本项目中已编写一次一密传输示例可参考。
    2）在config.encrypt数组中通过url和key统一配置需要加密的数据。
    3）如果存在cube.encryptData加密方法和cube.decryptData解密方法，src/utils/RESTClient在请求和返回数据时会自动调用。
### 5.国际化
    src/locales目录下配置国际化信息，可通过全局方法cube.msg("key值")获取。
### 6.全局变量
   	 在webpack中已将部分模块设置为全局变量，在项目中不用引入可直接使用。
    1）src/utils/Cube.js                 全局变量名为cube，该模块封装了一些常用方法
    2）src/config/index.js               全局变量名为$config
    3）src/locales/zh_CN/messages.js     全局变量名为$messages
### 7.构建打包
   	 打包时根据懒加载进行代码分割。
### 8.其他事项
	跨域时后端需通过corsConfiguration.addExposedHeader("sessionstate, AUTH2");允许前端访问返回的自定义header
