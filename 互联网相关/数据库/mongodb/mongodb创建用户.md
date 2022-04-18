#MongoDB 
## 选择数据库
```
use dbName;
```


## 创建用户

```
db.createUser({
	user:"yunzhou",
	pwd:"yunzhou",
	roles:[{
		role:"userAdminAnyDatabase",db:"admin"
	}]
})
```

输入以上这些命令的时候会报以下错误：

```
2021-08-18T22:57:47.388+0800 E QUERY    [thread1] Error: couldn't add user: Use of SCRAM-SHA-256 requires undigested passwords :
_getErrorWithCode@src/mongo/shell/utils.js:25:13
DB.prototype.createUser@src/mongo/shell/db.js:1437:15
@(shell):1:1
```

提示你需要按照 `SCRAM-SHA-256` 的加密方式来创建密码
解决方法，指定该用户在创建的时候使用不同的加密方式：
```
db.createUser({
	user:"yunzhou",
	pwd:"yunzhou",
	roles:[{
		role:"userAdminAnyDatabase",db:"admin"
	}],
	mechanisms:["SCRAM-SHA-1"]
})
```

使用以上方法之后就会成功，使用 `db.getUsers()` 函数或者 `show users` 查询该用户是否创建成功
```
> db.getUsers()
[
	{
		"_id" : "usv_platform.yunzhou",
		"userId" : UUID("eeb3c137-2712-4bff-9c92-8b3afbf27791"),
		"user" : "yunzhou",
		"db" : "usv_platform",
		"roles" : [
			{
				"role" : "userAdminAnyDatabase",
				"db" : "admin"
			}
		],
		"mechanisms" : [
			"SCRAM-SHA-1"
		]
	}
]
```
