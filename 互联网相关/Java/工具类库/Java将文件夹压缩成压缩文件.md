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
