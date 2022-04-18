#Java 
Java递归删除文件夹代码，调用下面的方法，下面方法2选1


```java
public void controller(String url){
	deleteFile(new File(url));
}
```


```java
# 讲解版
private void deleteFile(File file) {
	log.info("要删除的文件 -> {}", file.getName());
	//判断是否为目录
	if (file.isDirectory()) {
		log.info("{}是目录，不删除，判断是否为空目录",file.getName());
		//判断是否为空目录
		if (file.list().length == 0) {
			log.info("{}是空目录，删除",file.getName());
			file.delete();
		} else {
			//不是空目录，则遍历这个目录下的所有文件
			log.info("{}不是空目录，遍历该目录下的所有文件",file.getName());
			for (File value : file.listFiles()) {
				//递归判断该目录下的所有文件是否是文件
				deleteFile(value);
			}
			log.info("{}文件夹的长度为{}",file.getName(),file.list().length);
			//在遍历完该目录下的所有文件并删除后，再删除该目录本身
			if (file.delete()) {
				log.info("{}文件夹删除成功!!",file.getName());
			}
		}
	} else {
		log.info("{}不是目录",file.getName());
		if (file.delete()) {
			log.info("{}删除成功",file.getName());
		}
	}
}
```

```java
# 简化版
private void deleteFile(File file) {  
    //判断是否为目录  
 	if (file.isDirectory()) {  
        //判断是否为空目录  
 		if (file.list().length != 0) {  
            //不是空目录，则遍历这个目录下的所有文件  
			 for (File value : file.listFiles()) {  
				deleteFile(value);//递归判断该目录下的所有文件是否是文件  
			 }  
        }  
    }  
    file.delete(); //在遍历完该目录下的所有文件并删除后，再删除该目录本身  
}
```