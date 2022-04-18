#Java #SpringBoot 

# Serializable接口
定义：一个对象序列化的接口，一个类只有实现了Serializable接口，这个类的对象才能被序列化。

## 什么是序列化
**序列化**：把对象转化为可传输的字节序列过程称为序列化。

**反序列化**：把字节序列还原为对象的过程称为反序列化。

将对象的状态信息转换为可以存储或传输的形式的过程，在序列化期间，对象将其当前状态写入到**临时存储区或持久性存储区**，之后，便可以通过从存储区中读取或反序列化对象的状态信息，来重新创建该对象。

## 为什么要序列化
如果光看定义很难一下子理解序列化的意义，那么可以从另一个角度来推导出什么是序列化, 那么究竟序列化的目的是什么？

其实序列化最终的目的是为了对象可以**跨平台存储，和进行网络传输**。而进行跨平台存储和网络传输的方式就是**IO**，而我们的**IO**支持的数据格式就是字节数组。

因为单方面的只把对象转成字节数组还不行，因为没有规则的字节数组还是没办法把对象的本来面目还原回来的，所以必须在把对象转成字节数组的时候就制定一种规则**（序列化）**，那么从IO流里面读出数据的时候再以这种规则把对象还原回来**（反序列化）。**

示例：
如果我们要把一栋房子从一个地方运输到另一个地方去，**序列化**就是我把房子拆成一个个的砖块放到车子里，然后留下一张房子原来结构的图纸，**反序列化**就是我们把房子运输到了目的地以后，根据图纸把一块块砖头还原成房子原来面目的过程。

## 什么情况下需要序列化

通过上面我想你已经知道了凡是需要进行“跨平台存储”和”网络传输”的数据，都需要进行序列化。

本质上存储和网络传输 都需要经过 把一个对象状态保存成一种跨平台识别的字节格式，然后其他的平台才可以通过字节信息解析还原对象信息。

## Java怎么实现序列化？
Java实现序列化很简单，只要实现Serializable接口就可以了

```java
public class User implements Serializable{
 //年龄
 private int age;
 //名字
 private String name ;

 public int getAge() {
 return age;
    }
 public void setAge(int age) {
 this.age = age;
    }

 public String getName() {
 return name;
    }

 public void setName(String name) {
 this.name = name;
    }
}
```

把User对象写入输出流输出到文件：
```java
FileOutputStream fos = new FileOutputStream("D:\\temp.txt");
ObjectOutputStream oos = new ObjectOutputStream(fos);

User user = new User();
user.setAge(18);
user.setName("sandy");
oos.writeObject(user);

oos.flush();
oos.close();
```

再从文件流中读取文件：
```java
FileInputStream fis = new FileInputStream("D:\\temp.txt");

ObjectInputStream oin = new ObjectInputStream(fis);

User user = (User) oin.readObject();

System.out.println("name="+user.getName());
```

输出结果为：name=sandy

以上把User对象进行二进制的数据存储后，并从文件中读取数据出来转成User对象就是一个序列化和反序列化的过程。

## 序列化常见问题

1. **static属性不能被序列化**

能被序列化的是对象的状态，而static属性是整个类的状态，所以不能被序列化。

2. **Transient 属性不会被序列化**

Transient属性是修饰对象的属性的，一个对象中有一些属性我们不想被序列化传输到网络中，那么就可以在该属性是实用Transient关键字修饰，那么这个属性就不会被序列化。

一旦变量被transient修饰，变量将不再是对象持久化的一部分，该变量内容在序列化后无法获得访问。transient关键字只能修饰变量，而不能修饰方法和类。注意，本地变量是不能被transient关键字修饰的。变量如果是用户自定义类变量，则该类需要实现Serializable接口。

3. **父子类的序列化问题**

序列化是以正向递归的形式进行的，**如果父类实现了序列化那么其子类都将被序列化**；

子类实现了序列化而父类没实现序列化，那么只有子类的属性会进行序列化，而父类的属性是不会进行序列化的。