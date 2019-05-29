​		有一段时间没有搞微信开发了 ，今天突然要改一下程序！ 回头一看 微信的帮助文档太tm的稀烂的，太难懂了，这做个笔记以后看着方便。

​		微信有2个ACCESS_TOKEN：

1、基础接口的token 获取接口是：

https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid=APPID&secret=APPSECRET

2、用户网页授权access_token获取接口地址是：

https://api.weixin.qq.com/sns/oauth2/access_token?appid=APPID&secret=SECRET&code=CODE&grant_type=authorization_code

​		网页授权access_token需要通过code去获取，code是怎么来的，是通过调用下面接口来获取的：

https://open.weixin.qq.com/connect/oauth2/authorize?appid=APPID&redirect_uri=REDIRECT_URI&response_type=code&scope=SCOPE&state=STATE#wechat_redirect

​		注意这个接口中有个参数scope默认有2个值snsapi_base和snsapi_userinfo，这个接口会根据scope来生成不同的code并且获取不同作用的access_token，不管scope传什么值都能在得到对应access_token的同时得到open_id， 如果你只需要得到opend_id那使用snsapi_base参数到此结束了，如果需要获取用户的其他信息比如昵称、地址就要snsapi_userinfo会弹出授权。

3、怎么获取用户信息那就调用下面接口

https://api.weixin.qq.com/sns/userinfo?access_token={0}&openid={1}&lang=zh_CN

​		很明显这个接口中的access_token是第二步获取code的时候scope参数传snsapi_userinfo来换取的access_token。

4、微信还有一个获取用户基本信息的接口，但是这个接口需要你关注了公众号

https://api.weixin.qq.com/cgi-bin/user/info?access_token=ACCESS_TOKEN&openid=OPENID&lang=zh_CN

​		（此接口的access_token 是接口基础调用access_token 不是网页授权access_token）

​		微信的解释：是在用户和公众号产生消息交互或关注后事件推送后，才能根据用户OpenID来获取用户基本信息。这个接口，包括其他微信接口，都是需要该用户（即openid）关注了公众号后，才能调用成功的。