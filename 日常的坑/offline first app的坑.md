## 业务接口参数检查问题

由于offline first app需要对很多访问后端的业务请求做对应的本地话的存储，用户在offline时做了操作，我们就会存储起来，在用户online的时候去访问后端再更新本地数据库。

对于用户来说，他操作的时候就要确保这个操作是正确的，所以这些请求在后端那边做的参数检查，在移动端也要做一次，否则就会发生一种情况：用户offline的时候操作成功了，等用户online的时候请求后端，然后后端报错了。这时就很尴尬了，如果处理的不好，会发现这个上传任务一直不成功，用户本地有数据，一直传不到后端。即使可以重新修改之前的操作数据，但是可能用户已经脱离了业务操作的场景了，你让他重新改，他未必能改得了，原来的数据可能已经忘了。



## 多个bean之间的转换很麻烦

离线优先app基本上要有三个bean层，一个是后端传来的bean层，一个db的bean层，一个是内存中使用的bean层。

意味着一个从后端拿的对象要转换两次才能使用。同样，一个内存中产生的bean，要转换两次才能给后端传递过去。

这个过程很容易出错，要小心处理，尽量把每个bean都做成充血模型，做好封装，注意代码复用。