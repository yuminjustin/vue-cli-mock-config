# vue-cli
###vue-cli使用心得<br/><br/>
# vue-cli开发 build代码的注意事项
这里，我会记录自己在使用vue-cli开发项目遇到问题和解决方案。<br>
**============**<br>
2017-03-13 log<br>
使用官方套件build代码，chrome或者Firefox可以直接访问，IE呢？连IE11都歇菜了，很想跟IE说一句，去屎吧！<br>
1、使用了vuex，IE运行会报prosmise需要polyfill<br>
>  解决：添加babel-polyfill，要在入口文件import，而且必须在使用vuex的上面<br>

2、数据获取使用fetch，IE运行会报Request未定义<br>

> 解决：添加fetch-polyfill，要在入口文件import<br>

**============**<br>
# vue-cli-mock-config
**使用vue-cli，怎么配置mock的数据服务**<br/><br/>
使用vue-cli脚手架生成的开发环境，在涉及到数据获取是比较棘手，我平时使用fetch，你去GET数据是ok的，但POST就歇菜了，你会发现你的静态JSON数据文件会404<br/><br/>
因为POST的方式是必须由node提供服务响应才能获取到，那么怎么来实现呢？<br/><br/>
vue-cli会帮你安装express的服务器框架，有了它我们就能很快去配置一个我们自己的mock服务器了<br/><br/>
**配置步骤如下：**<br/><br/>
1.找到你已经生成好的build/dev-server.js<br/>
2.在里面添加express的路由即可<br/>
两个步骤就完成！<br/><br/>
**实例（我采用外置方式）：**<br/><br/>
**（build/dev-server.js）**<br/>
     
     var dataServer = require('./data_server')
     dataServer(app)
   
**（build/data_server.js）**<br/>  

     module.exports = function (app) {
         app.post('/test', function (req, res) {
              res.send('{"status":"is work"}')
         })
     }
  
**（客户端调用代码，ajax同样适用）**<br/>  
 
      let req = new Request("/test", {
            method: 'POST',
            headers: {
              'Content-Type': 'application/x-www-form-urlencoded'
           },
           body:"a=1&b=2" /*提交的数据 type String*/
      })

      fetch(req).then((response) => response.json()).then(
              (data) => sfn(data)
      ).catch(
              (e) => errfn(e)
      )
	  
使用上述方式一个简易的服务器已经搭建好了，可是在服务端却无法获取fecth转过来的数据，要么undefined<br/><br/>
这时我们就需要使用body-parser了，这时一个express的中间件，用来处理客户端传来的数据，一般会出现空对象的现象，这个要处理好提交的Content-Type [文档](https://github.com/expressjs/body-parser)<br/><br/>
所以data_server的配置就应该变成这样：<br/><br/>

**（build/data_server.js）**<br/>

      var bodyParser = require('body-parser')
	  module.exports = function (app) {    
	  
        app.use(bodyParser()); //配置express
        app.use(bodyParser.json()); 
        app.use(bodyParser.urlencoded({
           extended: true
        }));
        app.use(bodyParser.json({
           type: 'application/*+json'
        }))
  
        app.post('/test', function (req, res) {
		    //这时我们就能在客户端看到 刚才传过来的数据
		    res.send(req.body)
        })
      }

服务端数据接收正常，那么要把本地的json数据传给客户端才算真正完成。这时我们就要借助node的fs文件模块<br/><br/>

**（build/data_server.js）**<br/>
   
       var bodyParser = require('body-parser')
	   var fs = require('fs');
	   
	   module.exports = function (app) {    
	  
        app.use(bodyParser()); //配置express
        app.use(bodyParser.json()); 
        app.use(bodyParser.urlencoded({
           extended: true
        }));
        app.use(bodyParser.json({
           type: 'application/*+json'
        }))
  
         app.post('/test', function (req, res) {
           var str = JSON.stringify(req.body);
           if (str.indexOf('first') > -1) {/*这里我是根据字符串作区分*/
               var file = "E:\\test.json";  /*文件地址  \ 符号要转译*/
               var result = JSON.parse(fs.readFileSync(file));  /*采用同步的方式读取数据*/
               res.send(result)
           }
           else res.send('{"status":"is empty"}')  //默认数据
         })
       }
	  
一个在vue-cli下的mock服务器就做好了<br/><br/>
有一个遗留问题，vue的路由是hash方式，我们在配置express路由是不用去理会两个路由的冲突，而react-router默认就history的方式，这会和express路由冲突，必须要做特殊处理，这方面我还没有测试过，待使用后会来补充。

