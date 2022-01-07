2022-01-05
09:48:32
author:陈建浩
#Go 

--- 
## JWT在Go中的实现
### 安装jwt-go
安装jwt在go中实现的开源库 `jwt-go`

```bash
go get -u github.com/dgrijalva/jwt-go@latest
```


### 准备
使用 jwt-go 库生成 token，我们需要定义需求（claims），也就是说我们需要通过 jwt 传输的数据；这里的结构体根据自己的情况来定义，这里我只定义id和用户名；
除了自己定义的字段，还需要添加在 `jwt-go` 包中预定义的 `jwt.StandardClaims` 字段
```go
type Claims struct {
	ID int64	
	Username string
	jwt.StandardClaims
}
```
其中  `jwt.StandardClaims` 中的字段有
```go
type StandardClaims struct {
	Audience  string `json:"aud,omitempty"`
	ExpiresAt int64  `json:"exp,omitempty"`
	Id        string `json:"jti,omitempty"`
	IssuedAt  int64  `json:"iat,omitempty"`
	Issuer    string `json:"iss,omitempty"`
	NotBefore int64  `json:"nbf,omitempty"`
	Subject   string `json:"sub,omitempty"`
}
```

```go
## 以上字段的意思
  Audience  接收jwt的一方
  ExpiresAt jwt的过期时间，必填
  Id        id
  IssuedAt  jwt的签发时间
  Issuer    jwt的签发者
  NotBefore 定义在什么时间之前，该jwt都是不可用的
  Subject   jwt所面对的用户
```

### 生成token
在 `jwt-go`库中生成token需要用到以下2个方法：
- jwt.NewWithClaims
- SignedString

**jwt.NewWithClaims()方法**
```go
func NewWithClaims(method SigningMethod, claims Claims) *Token
```
该方法需要传入 `SigningMethod` 类型参数和自定义 `Claims` 参数；
 `SigningMethod` 类型参数代表这 `crypth.Hash` 加密算法的方案，取值有3个，分别是
 ```go
jwt.SigningMethodHS256
jwt.SigningMethodHS384
jwt.SigningMethodHS512
 ```

 参数 2 是 `Claims`，包含自定义类型和 StandardClaim，StandardClaim 嵌入在自定义类型中，以方便对标准声明进行编码，解析和验证。

 **SignedString()方法**
 上面的 `jwt.NewWithClaims()`方法会返回一个 token指针，通过该指针调用SignedString方法
 SignedString 方法根据传入的空接口类型参数 key，返回完整的签名令牌。
 ```go
func (t *Token) SignedString(key interface{}) (string, error) 
```

### 生成token实例
```go
// 生成jwt

func GenerateToken() (string, error) {
	//获取当前时间	
	nowTime := time.Now()
	
	//设置过器时间，这里设置300秒也就是5分钟	
	expireTime := nowTime.Add(300 * time.Second)	  
	
	//这里设置签发者	
	issuer := "frank"	  
	
	//声明对象	
	claims := Claims{	
		ID: 10001,	
		Username: "frank",	
		StandardClaims: jwt.StandardClaims{	
			ExpiresAt: expireTime.Unix(),	
			Issuer: issuer,	
		},	
	}	  
	
	//使用NewWithClaims声明加密算法和Claim对象，返回*jwt.Token	
	//然后使用*jwt.Token 来调用SignedString方法，传入密钥参数，获取完整的token和错误对象	
	//密钥这里使用 golang，建议设置为常量值方便调用
	token, err := jwt.NewWithClaims(jwt.SigningMethodHS256, claims).SignedString([]byte("golang"))	
	return token, err
}
```

### 解析token
使用 jwt-go 库解析 token，主要用到两个方法，分别用通过与解析传入的 token 字符串，和根据 MyCustomClaims 结构体定义的相关属性要求进行校验。
用到以下2个方法：
- ParseWithClaims()
- Valid()

**ParseWithClaims()方法**

```go
func jwt.ParseWithClaims(tokenString string, claims jwt.Claims, keyFunc jwt.Keyfunc) (*jwt.Token, error)
```
传入token、Cliam空对象指针以及一个函数对象；该函数对象传入token，返回密钥数组


**Valid()方法**
方法用于校验鉴权的声明

实例代码：
```go
//解析token
func ParseToken(token string) (*Claims, error) {

	//传入token、自定义的Claim对象指针以及一个函数对象	
	//该函数对象返回密钥的byte数组、nil预定义字符串	
	tokenClaims, err := jwt.ParseWithClaims(token, &Claims{}, func(token *jwt.Token) (interface{}, error) {	
		return []byte("golang1"), nil	
	})
	  	
	//判断是否出错	
	if err != nil {	
		return nil, err	
	}	  
	
	// 调用Valid()方法	
	if tokenClaims != nil {	
		if claims, ok := tokenClaims.Claims.(*Claims); ok && tokenClaims.Valid {	
			return claims, nil	
		}	
	}	
	return nil, err
}
```



# 参考文章
[Golang语言使用 jwt-go 库生成和解析 token](https://cloud.tencent.com/developer/article/1770768)
[golang之JWT实现](https://studygolang.com/articles/28967)
[golang iris的jwt实践，获取jwt、携带jwt、验证jwt、设置过期时间、自定义错误处理函数、格式化错误返回](https://studygolang.com/articles/25165)
