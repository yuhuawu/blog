title: 如何POST JSON数据到RESTful API
tags: programming

---

第一次做前端，第一次接触node.js，好多第一次啊。

###背景
打算做一个日报工具，收集团队的同学在jira、wiki和git上的action，自动生成汇总页面，加上UGC后，形成正式日报，发送出来。
希望这个工具，可以推动大家更多的使用jira、wiki、git，同时不给大家带来额外的工作量。
之前，是需要大家每天手动填写周报，邮件发出。很多jira、wiki和git上的内容，要重复填写一次，比较麻烦。

###具体问题
收集有多种方式，但抽象一下，就是主动和被动。
**主动**：像爬虫，爬取页面，抽取信息。
**被动**：构建订阅者，信息提供方推动信息。
前者适用面广，难度大，工作量大，需要考虑调度、解析等问题。
后者适用面小，需要对方系统支持，但系统已经结构化，工作量小，系统负荷小。
所幸，jira支持后者，它称之为[webhook][1]。

因此，我们的问题，就是构建一个API服务，针对接收到的信息，做相应的处理，达到我们的业务需求。

在开发API服务的过程，遇到的第一个问题：怎么构建一个工具，可以方便的发送JSON格式的数据，到我们的服务上，便于开发时的测试。类似Mock。

###发送JSON消息的几种方式 
#### 借助工具
我们可以借助已有的工具，来帮助我们post数据。
``` plain
世界这么大，工具总会有。
前人这么多，工具一定有。
善用google，善用tools。
```

##### 1、curl
```shell
curl -H "content-type: application/json; charset=UTF-8" -H "accept: */*" -H "via: 1.1 localhost (Apache-HttpClient/4.3.2 (cache))" -H "host: localhost:3000" -H "connection: Keep-Alive" -H "user-agent: Atlassian HttpClient 0.20.1 / JIRA-6.4.4 (64019) / Default" -X POST -d "{\"a\":1}" http://localhost:3000/hook/jira/project-1
```
``` plain
注意，-d 带上post的数据时，要注意"的转义(\")。
同时，header有多项时，要分开写，多个-H。
```

##### 2、postman
![postman](http://7xo1mp.com1.z0.glb.clouddn.com/postman.png)
这是一个chrome插件，对于我们测试，非常有用。而且可以共享collection。

需要post一段json，请参考下图配置。
![postman-post-data](http://7xo1mp.com1.z0.glb.clouddn.com/postman_post_data.png)
``` plain
这里是不需要转义的。但一定要增加一项header："content-type: application/json; charset=UTF-8"。
```

#### 自己写测试工具
我们需要写单元测试工具，还是得落到代码，可以使用restler。
这里强烈推荐[《Web Development with Node and Express》][3]，非常适合node.js的入门，step by step。里面关于测试的讲解，非常赞。后续我再专门介绍。
```javascript
var rest = require('restler');

var message = {
        timestamp:1447923260390,
        webhookEvent: 'jira:issue_created'
}

var base = 'http://localhost:3000';

rest.postJson(base+'/hook/jira/project-1', message, {timeout:10000}).on('timeout', function(ms){
  console.log('did not return within '+ms+' ms');
}).on('complete',function(data,response){
  console.log('get the response');
});
```
这里借助了restler，来发送post请求。

---

###总结
第一次做web开发，对json还不算特别熟悉，自己在看了[restler][2]的下面这段话，才顿悟。
``` plain
 The data to be added to the body of the request. Can be a string or any object. Note that if you want your request body to be JSON with the Content-Type: application/json, you need to JSON.stringify your object first. Otherwise, it will be sent as application/x-www-form-urlencoded and encoded accordingly. Also you can use json() and postJson() methods.
```

[1]: https://developer.atlassian.com/jiradev/jira-apis/webhooks
[2]: https://github.com/danwrong/restler
[3]: http://shop.oreilly.com/product/0636920032977.do






