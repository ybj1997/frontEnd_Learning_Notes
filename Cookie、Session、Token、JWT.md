# 一、Cookie

- #### HTTP是无状态协议：因此需要Cookie来告知服务器，请求报文是否来自于统一服务器

- #### Cookie存储在浏览器客户端：普通ajax请求和jsonp跨域请求默认携带Cookie，而cors跨域请求和fetch请求默认不携带Cookie需要额外配置

- #### Cookie不允许跨域：每个Cookie绑定单一的域名，通过配置domain可以实现一级和二级域名共享

|    属性    |                             说明                             |
| :--------: | :----------------------------------------------------------: |
| name=value | 必须是字符串类型（Unicode字符用字符编码；二进制类型用BASE64编码） |
|   domain   |              指定Cookie所属域名，默认是当前域名              |
|    Path    |         指定路径**下**的路由可以访问（默认为 ‘ / ’）         |
|   MaxAge   | **正数**：失效倒计时；<br/>**负数**：临时Cookie，浏览器关闭时失效；<br/>**0**：删除Cookie |
|   secure   | 默认为false。<br/>当 secure 值为 true 时，cookie 在 HTTP 中是无效，在 HTTPS 中才有效。 |
|  httpOnly  | 设置了 httpOnly 属性，则无法通过 JS 脚本 读取到该 cookie 的信息，可以一定程度防止XSS攻击<br/>但还是能通过 Application 中手动修改 cookie |

# 二、Session

- Seesion是另一种记录浏览器与服务器**会话状态**的机制
- Session基于Cookie实现，Session保存在服务器端，SessionID保存在本地Cookie中



![1637407097284](C:\Users\AllenYan\AppData\Roaming\Typora\typora-user-images\1637407097284.png)

# 三、比较Session与Cookie

1. #### 安全性：由于Session存储在服务器端，Cookie存储在客户端，因此Session安全性更好

2. #### 存值类型：Cookie只支持字符串类型；Session支持任意类型

3. #### 有效期：Cookie可以设置有效期，Session有效期较短，客户端关闭或者超时都会失效

4. #### 内存值：Cookie数据大小不超过4K，Session更大，但是占用更多服务器资源

   

# 四、Token与JWT

## 1.Token

- #### 访问资源接口（API）时所需要的资源凭证（而Cookie、Session是记录会话状态的机制）

- #### 用服务器端计算验证Token的时间来换取Session占用内存的代价

## 2.JWT（JSON WEB TOKEN）

- ####  是目前最流行的**跨域认证**解决方案 

- #### 由header-payload-signature组成

  ```javascript
  HMACSHA256(
    base64UrlEncode(header) + "." +
    base64UrlEncode(payload),
    secret)
  //HMACSHA256由header指定签名算法
  //header和payload是JSON对象，用Base64URL编码成字符串
  ```

- #### 使用方式

```javascript
// 支持 JWT/OAuth2.0 的 axios 配置
export  function axios_JWT(){
    axios.interceptors.request.use(config => {
        // 在 localStorage 获取 token
        let token = localStorage.getItem("token");

        // 如果存在则设置请求头
        if (token) {
            config.headers['Authorization'] = token;
            console.log(config)
        }
        return config;
    });
}
```

## 3.Token与JWT的异同点

#### 相同点：

- 都是资源访问凭证
- 都是无状态机制
- 都可以记录用户信息（{username，password}）

#### 不同点：

Token：服务器端，在收到Token后，需要查询数据库获取用户信息，验证此Token是否有效

JWT：Token和payload加密存储与客户端，服务器只需要用密钥解密即可，不需要查询数据库

# 五、常用的前后端鉴权

#### 1.Session-Cookie

2.Token验证（JWT）

3.OAuth2.0