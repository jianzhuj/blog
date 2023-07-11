使用双token
一个正常的jwt token 叫accessToken   半小时
一个refreshToken 用来刷新token 24小时

服务端返回两个token。

请求到后端时，有三种情况
1. accessToken没有过期,就正常执行。
2. ccessToken过期,refreshToken没有过期,就表示半小时内没有进行操作。两个时候,两个token都要刷新。
3. 两个token都过期,表示24小时内没有访问系统，这个时候直接踢回到登录页。