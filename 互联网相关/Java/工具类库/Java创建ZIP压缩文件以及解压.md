## Java将文件（文件夹）打包成压缩文件
```java
  
/**  
 * @param inputFileName 你要压缩的文件夹(整个完整路径)  
 * @param zipFileName 压缩后的文件(整个完整路径)  
 * @throws Exception  
 */  
public static Boolean zip(String inputFileName, String zipFileName) throws Exception {  
   zip(zipFileName, new File(inputFileName));  
   return true;  
}  
private static void zip(String zipFileName, File inputFile) throws Exception {  
   ZipOutputStream out = new ZipOutputStream(new FileOutputStream(zipFileName));  
   zip(out, inputFile, "");  
   out.flush();  
   out.close();  
}  
private static void zip(ZipOutputStream out, File f, String base) throws Exception {  
   if (f.isDirectory()) {  
      File[] fl = f.listFiles();  
      out.putNextEntry(new ZipEntry(base + "/"));  
      base = base.length() == 0 ? "" : base + "/";  
      for (int i = 0; i < fl.length; i++) {  
         zip(out, fl[i], base + fl[i].getName());  
      }  
   } else {  
      out.putNextEntry(new ZipEntry(base));  
      FileInputStream in = new FileInputStream(f);  
      int b;  
      while ((b = in.read()) != -1) {  
         out.write(b);  
      }  
      in.close();  
   }  
}

```


## Java将ZIP压缩文件解压
```java
/**  
 * zip文件解压  
 * @param inputFile 待解压文件夹/文件  
 * @param destDirPath 解压路径  
 */
public static void zipUncompress(String inputFile,String destDirPath) throws Exception {  
   File srcFile = new File(inputFile);//获取当前压缩文件  
   // 判断源文件是否存在  
   if (!srcFile.exists()) {  
      throw new Exception(srcFile.getPath() + "所指文件不存在");  
   }  
   ZipFile zipFile = new ZipFile(srcFile);//创建压缩文件对象  
   //开始解压  
   Enumeration<?> entries = zipFile.entries();  
   while (entries.hasMoreElements()) {  
      ZipEntry entry = (ZipEntry) entries.nextElement();  
      // 如果是文件夹，就创建个文件夹  
 	  if (entry.isDirectory()) {  
         String dirPath = destDirPath + "/" + entry.getName();  
         srcFile.mkdirs();  
      } else {  
         // 如果是文件，就先创建一个文件，然后用io流把内容copy过去  
 		 File targetFile = new File(destDirPath + "/" + entry.getName());  
         // 保证这个文件的父文件夹必须要存在  
 		 if (!targetFile.getParentFile().exists()) {  
            targetFile.getParentFile().mkdirs();  
         }  
         targetFile.createNewFile();  
         // 将压缩文件内容写入到这个文件中  
 		 InputStream is = zipFile.getInputStream(entry);  
         FileOutputStream fos = new FileOutputStream(targetFile);  
         int len;  
         byte[] buf = new byte[1024];  
         while ((len = is.read(buf)) != -1) {  
            fos.write(buf, 0, len);  
         }  
         // 关流顺序，先打开的后关闭  
 		 fos.close();  
         is.close();  
      }  
   }  
}
```