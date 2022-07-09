# jwt  --> jsonWebToken

> Json web token (JWT), 是为了在网络应用环境间传递声明而执行的一种基于JSON的开放标准（(RFC 7519).该token被设计为紧凑且安全的，特别适用于分布式站点的单点登录（SSO）场景。JWT的声明一般被用来在身份提供者和服务提供者间传递被认证的用户身份信息，以便于从资源服务器获取资源，也可以增加一些额外的其它业务逻辑所必须的声明信息，该token也可直接被用于认证，也可被加密。

#### 安装

```shell

npm i jsonwebtoken 
yarn add jsonwebtoken
pnpm add jsonwebtoken
```

### 使用

```js
const jwt = require("jsonwebtoken");

// 颁发
const LIU_KEY = "liushipeng"; // 密钥

testRouter.get("/jwt", (ctx, next) => {
  const user = { id: 10, name: "liu" };
  const token = jwt.sign(user, LIU_KEY, {
    expiresIn: 30, // 过期时间，单位 秒
  });

  ctx.body = token;
});

// 验证
testRouter.get("/test/jwt", (ctx, next) => {
  const authorization = ctx.headers.authorization;
  const token = authorization.replace("Bearer ", "");
  console.log(token);  // { id: 10, name: "liu" };
  //   {
  //     id: 10,
  //     name: "liu",
  //     iat: 1657357110,  // 设置时间
  //     exp: 1657357140,  // 过期时间
  //   };

  try {
    const result = jwt.verify(token, LIU_KEY);
    ctx.body = result;
  } catch (error) {
    ctx.body = "token 无效";
  }
});
```



###  服务器集群方案

 防止服务器集群大家都可以获取私钥颁发 token，如果一个被攻击，全部泄露，私钥只有一个人可以颁发，别人解析都用公钥

#### 使用 openssl 来做密钥验证

```shell
# window下可以使用 git bash
# 进入项目文件夹 
openssl // 命令
genrsa -out private.key 1024  // 生成私钥一个 1024 的密钥
rsa -in private.key -pubout -out public.key  // 使用私钥生成一个公钥
```

#### 使用

```js
const fs = require("fs");
const path = require("path");
const jwt = require("jsonwebtoken");

const PRIVATE_KEY = fs.readFileSync(
  path.resolve(__dirname, "../../keys/private.key")
);
const PUBLIC_KEY = fs.readFileSync(
  path.resolve(__dirname, "../../keys/public.key")
);

// 颁发 token 公钥私钥
testRouter.get("/jwt2", (ctx, next) => {
  const user = { id: 10, name: "liu" };
  // 可以 传递一个 buffer 的密钥
  const token = jwt.sign(user, PRIVATE_KEY, {
    expiresIn: 30, // 过期时间，单位 秒
    algorithm: "RS256", // 指定非对称加密算法
  });

  ctx.body = token;
});

// 验证 token 公钥私钥
testRouter.get("/test/jwt2", (ctx, next) => {
  const authorization = ctx.headers.authorization;
  const token = authorization.replace("Bearer ", "");
  //   {
  //     id: 10,
  //     name: "liu",
  //     iat: 1657357110,  // 设置时间
  //     exp: 1657357140,  // 过期时间
  //   };

  try {
    const result = jwt.verify(token, PUBLIC_KEY, {
      algorithm: ["RS256"], // 指定非对称加密算法,可以传递多个算法
    });
    ctx.body = result;
  } catch (error) {
    ctx.body = "token 无效";
  }
});
```

