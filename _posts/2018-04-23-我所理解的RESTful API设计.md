

关于RESTful架构的解释，在我的另一篇文章[理解RESTful](https://github.com/sputnikW/web-notes/blob/master/%E7%90%86%E8%A7%A3RESTful.md)。  本文行文前,我是通过这篇文章（[Principles of good RESTful API Design](https://codeplanet.io/principles-good-restful-api-design/)）来理解关于RESTful API的。  
## 什么是RESTful API设计？
在另一篇文章关于RESTful的讨论中，可以看出，RESTful架构，是一个**以资源为基础，客户端通过服务器提供的URI（统一资源接口），从服务器获取或者操作目标资源**。而RESTful API设计，就是对URI该如何设计，以及URI如何和HTTP配合，共同完成工作。以及更重要的任务：如何达到RESTful目的（可拓展性，简明，可维护性，结构清晰，可移植性，可靠性等），以满足当今功能强大却又复杂的web程序。  
## RESTful API设计应该注意哪些方面？  
***注：这一部分我主要是在阅读了[Principles of good RESTful API Design](https://codeplanet.io/principles-good-restful-api-design/)这篇文章后，大致谈一下自己理解的一些要点，具体的详细解释，建议大家去看一下原文。***
### 对于API的设计：  
- **API应该被抽象成数据，而不应该和操作逻辑相关。**（一是因为URI本来就是表示资源的‘位置’的一个接口，二是对同一个资源，经常有多种不同的操作，比如对于用户这类资源，我们有查，增，改，删等动作，如果把这些动作都包含在API中，会导致混乱，例如会出现`‘增加用户’`，`‘删除用户’`，`‘修改用户’`，`‘检索用户’`这些API，很难管理，而`增/删/改/查`+`用户`就好多了）   
- **对于资源的操作，应该交给HTTP动词来指定。**（最常用的就是GET，POST，PUT，PATCH，DELETE）
- **为同一类资源的不同实例提供键（key）。如user/id** (这样的好处是可以区分一类资源的所有集合和单个资源。例如/books应该指代所有书的集合(collection),而/books/id则是指代具体的一本书。)    
- **资源的层次关系应该通过URI中的先后顺序来体现，例如：/animals/dog** (这样更能体现资源的层次关系，而不将所有的资源都表示在同一层。)  
- **对于大量的资源，可以在API中提供筛选或者整理，例如?page=2&limit=10,?sortby=time&order=asc** (每次都将所有请求的资源返回给API的调用者不见得是件好事，有时候提供筛选整理的功能是很有必要的。)  
- **对于不同的HTTP动词，API应该返回合适的数据。**（因为API调用者每次调用，都是想执行一些动作，对于API的设计者来说，我们就应该给他们返回完成调用之后可能会需要的数据。）  
  -  GET:返回查询的资源  
  -  POST：返回新添加的资源  
  -  PUT/PATCH：返回修改后的资源(PUT指修改整个资源，PATCH指修改资源的部分属性)  
  -  DELETE:返回空文档  
注：这其中需要注意的是，有些与资源相关的信息没必要返回，比如本次操作的时间戳，或者这次资源的大小，甚至有时候，我们可能不用返回整个资源，只需要提供一个ID，这样如果用户真的想做什么事情，利用ID完全可以做到。   
- **API的设计者应该提供简洁易懂的API文档，包含API使用示例，对返回的出错信息的解释**
  
### 设计中应该时刻遵循的原则  
- **尽可能小的对用户施加限制。**  
- **URI的设计应该经可能的易理解，方便使用。**
- **使用合适的响应状态码(HTTP response status)。**  （关于什么情况下应该使用什么状态码，在这不赘述了，可以参考[Mozilla文档-HTTP Response Status](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)）
- **当API库很庞大时，有必要版本控制，具体的细节可以参见文章开头提到的原文。**  
***  
  
## 我的RESTful API设计的实践：
在我的练习项目中，我尝试了RESTful，简要概括，我的web项目，是一个个人写信和寄信的网站，我的API设计如下表：  
![](https://github.com/sputnikW/web-notes/blob/master/images/URI%E8%AE%BE%E8%AE%A1.png)  
对于表格有一点需要说明：  
在注册/登陆/登出的URI中我使用了‘动词’，这是因为我观察了twitter和github的登陆界面的URL，他们都是使用类似的方式，我个人认为这是因为这种传统的方式已经被所有人所接受，并且只涉及用户认证，并不会带来特别大的问题，所以是可以接受的。  
  
注：本文所有观点均为个人观点，如果错误，感谢指正！  
4/20/2018 3:49:52 PM 

