# vue-cli-mock-config
###使用vue-cli，怎么配置mock的数据服务<br/><br/>
使用vue-cli脚手架生成的开发环境，在涉及到数据层是比较棘手，我平时使用fetch，你去GET数据是ok的，但POST就歇菜了，你会发现你的静态数据会404<br/><br/>
因为POST的方式是必须由node服务提供响应才能获取到，那么怎么来实现呢？<br/><br/>
vue-cli会帮你安装express的服务器框架，有了它我们就能很快去配置一个我们自己的mock服务器了<br/><br/>
