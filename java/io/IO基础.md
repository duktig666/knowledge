# IO相关

## 流的分类

1. 操作数据单位：字节流、字符流
2. 数据的流向：输入流、输出流
3. 流的角色：节点流、处理流

### 图示

#### 按操作方式分类结构图：

![image-20200616203751649](https://gitee.com/koala010/typora/raw/master/img/20200726214205.png)

#### 按操作对象分类结构图：

![image-20200616203828060](https://gitee.com/koala010/typora/raw/master/img/20200726214208.png)



## 流的体系结构

| 抽象基类     | 节点流（或文件流）                            | 缓冲流（处理流的一种）                                     |
| ------------ | --------------------------------------------- | ---------------------------------------------------------- |
| InputStream  | FileInputStream   (read(byte[] buffer))       | BufferedInputStream (read(byte[] buffer))                  |
| OutputStream | FileOutputStream  (write(byte[] buffer,0,len) | BufferedOutputStream (write(byte[] buffer,0,len) / flush() |
| Reader       | FileReader (read(char[] cbuf))                | BufferedReader (read(char[] cbuf) / readLine())            |
| Writer       | FileWriter (write(char[] cbuf,0,len)          | BufferedWriter (write(char[] cbuf,0,len) / flush()         |

![image-20200616210127872](F:\知识库\Java\IO\IO.assets\image-20200616210127872.png)

## IO原理

- I/O是Input/Output的缩写， I/O技术是非常实用的技术， 用于处理设备之间的数据传输。 如读/写文件，网络通讯等。
- Java程序中，对于数据的输入/输出操作以“流(stream)” 的方式进行。
- java.io包下提供了各种“流”类和接口，用以获取不同种类的数据，并通过标准的方法输入或输出数据。
- 输入input： 读取外部数据（磁盘、光盘等存储设备的数据）到程序（内存）中。
- 输出output： 将程序（内存）数据输出到磁盘、光盘等存储设备中。



## 节点流和处理流

> 节点流：直接从数据源或目的地读写数据

![image-20200616210233212](https://gitee.com/koala010/typora/raw/master/img/20200726214214.png)

> 处理流：不直接连接到数据源或目的地，而是“连接” 在已存在的流（节点流或处理流）之上，通过对数据的处理为程序提供更为强大的读写功能。

![image-20200616210242248](https://gitee.com/koala010/typora/raw/master/img/20200726214217.png)

### 节点流(或文件流)：注意点

- 定义文件路径时，注意：可以用“/”或者“\\”。
- 在写入一个文件时，如果使用构造器FileOutputStream(file)，则目录下有同名文件将被覆盖。
- 如果使用构造器FileOutputStream(file,true)，则目录下的同名文件不会被覆盖，在文件内容末尾追加内容。
- 在读取文件时，必须保证该文件已存在，否则报异常。
- 字节流操作字节，比如： .mp3， .avi， .rmvb， mp4， .jpg， .doc， .ppt
- 字符流操作字符，只能操作普通文本文件。 最常见的文本文件： .txt， .java， .c， .cpp 等语言的源代码。尤其注意.doc,excel,ppt这些不是文本文件。



### 处理流

#### 1.缓冲流

为了提高数据读写的速度， Java API提供了带缓冲功能的流类，在使用这些流类时，会创建一个内部缓冲区数组，**缺省使用8192个字节(8Kb)的缓冲区**。

![image-20200616210512913](https://gitee.com/koala010/typora/raw/master/img/20200726214221.png)

##### 缓冲流注意事项

- 当读取数据时，数据按块读入缓冲区，其后的读操作则直接访问缓冲区
- 当使用BufferedInputStream读取字节文件时， BufferedInputStream会一次性从文件中读取8192个(8Kb)， 存在缓冲区中， 直到缓冲区装满了， 才重新从文件中读取下一个8192个字节数组。
- 向流中写入字节时， 不会直接写到文件， 先写到缓冲区中直到缓冲区写满，BufferedOutputStream才会把缓冲区中的数据一次性写到文件里。使用方法flush()可以强制将缓冲区的内容全部写入输出流
- 关闭流的顺序和打开流的顺序相反。只要关闭最外层流即可， 关闭最外层流也会相应关闭内层节点流
- flush()方法的使用：手动将buffer中内容写入文件
- 如果是带缓冲区的流对象的close()方法， 不但会关闭流， 还会在关闭流之前刷新缓冲区， 关闭后不能再写出

![image-20200616210638626](https://gitee.com/koala010/typora/raw/master/img/20200726214224.png)



#### 2.转换流

- 转换流提供了在字节流和字符流之间的转换
- Java API提供了两个转换流：
  - InputStreamReader：将InputStream转换为Reader
  - OutputStreamWriter：将Writer转换为OutputStream
- 字节流中的数据都是字符时，转成字符流操作更高效。
- 很多时候我们使用转换流来处理文件乱码问题。实现编码和
  解码的功能。



##### 图解

![image-20200616210822816](https://gitee.com/koala010/typora/raw/master/img/20200726214231.png)



#### 3.标准输入、输出流

- System.in和System.out分别代表了系统标准的输入和输出设备
- 默认输入设备是： 键盘， 输出设备是：显示器
- System.in的类型是InputStream
- System.out的类型是PrintStream，其是OutputStream的子类FilterOutputStream 的子类
- 重定向：通过System类的setIn， setOut方法对默认设备进行改变。
  - public static void setIn(InputStream in)
  - public static void setOut(PrintStream out)



#### 4.打印流

实现将基本数据类型的数据格式转化为字符串输出

打印流： PrintStream和PrintWriter

- 提供了一系列重载的print()和println()方法，用于多种数据类型的输出
- PrintStream和PrintWriter的输出不会抛出IOException异常
- PrintStream和PrintWriter有自动flush功能
- PrintStream 打印的所有字符都使用平台的默认字符编码转换为字节。在需要写入字符而不是写入字节的情况下，应该使用 PrintWriter 类。
- System.out返回的是PrintStream的实例



#### 5.数据流

- 为了方便地操作Java语言的基本数据类型和String的数据，可以使用数据流。
- 数据流有两个类： (用于读取和写出基本数据类型、 String类的数据）
  - DataInputStream 和 DataOutputStream
  - 分别“套接”在 InputStream 和 OutputStream 子类的流上



#### 6.对象流

- ObjectInputStream和OjbectOutputSteam
  - 用于存储和读取基本数据类型数据或对象的处理流。它的强大之处就是可以把Java中的对象写入到数据源中，也能把对象从数据源中还原回来。
- 序列化： 用ObjectOutputStream类保存基本类型数据或对象的机制
- 反序列化： 用ObjectInputStream类读取基本类型数据或对象的机制
- ObjectOutputStream和ObjectInputStream不能序列化static和transient修饰的成员变量







# IO使用

## File类

1. File类的一个对象，代表一个文件或一个文件目录(俗称：文件夹)
2. java.io.File类： 文件和文件目录路径的抽象表示形式，与平台无关
3. File类声明在java.io包下
4. File类中涉及到关于文件或文件目录的创建、删除、重命名、修改时间、文件大小等方法，并未涉及到写入或读取文件内容的操作。如果需要读取或写入文件内容，必须使用IO流来完成。
5. 后续File类的对象常会作为参数传递到流的构造器中，指明读取或写入的"终点".

### 根据操作系统动态提供分隔符（跨平台）

```java
public static final String separator;
```

```java
File file1 = new File("d:\\atguigu\\info.txt");
File file2 = new File("d:" + File.separator + "atguigu" + File.separator + "info.txt");
File file3 = new File("d:/atguigu");
```



### 创建File类的实例

```java
File(String filePath)
File(String parentPath,String childPath)
File(File parentFile,String childPath)
```

```java
public void test1(){
    //构造器1
    File file1 = new File("hello.txt");//相对于当前module
    File file2 =  new File("D:\\workspace_idea1\\JavaSenior\\day08\\he.txt");

    System.out.println(file1);
    System.out.println(file2);

    //构造器2：
    File file3 = new File("D:\\workspace_idea1","JavaSenior");
    System.out.println(file3);

    //构造器3：
    File file4 = new File(file3,"hi.txt");
    System.out.println(file4);
}
```

### 常用方法

```
public String getAbsolutePath()：获取绝对路径
public String getPath() ：获取路径
public String getName() ：获取名称
public String getParent()：获取上层文件目录路径。若无，返回null
public long length() ：获取文件长度（即：字节数）。不能获取目录的长度。
public long lastModified() ：获取最后一次的修改时间，毫秒值

如下的两个方法适用于文件目录：
public String[] list() ：获取指定目录下的所有文件或者文件目录的名称数组
public File[] listFiles() ：获取指定目录下的所有文件或者文件目录的File数组
```



```java
public void test2(){
    File file1 = new File("hello.txt");
    File file2 = new File("d:\\io\\hi.txt");

    System.out.println(file1.getAbsolutePath());
    System.out.println(file1.getPath());
    System.out.println(file1.getName());
    System.out.println(file1.getParent());
    System.out.println(file1.length());
    System.out.println(new Date(file1.lastModified()));

    System.out.println();

    System.out.println(file2.getAbsolutePath());
    System.out.println(file2.getPath());
    System.out.println(file2.getName());
    System.out.println(file2.getParent());
    System.out.println(file2.length());
    System.out.println(file2.lastModified());
}
```

文件目录操作

```java
public void test3(){
    File file = new File("D:\\workspace_idea1\\JavaSenior");

    String[] list = file.list();
    for(String s : list){
    System.out.println(s);
    }

    System.out.println();

    File[] files = file.listFiles();
    for(File f : files){
    System.out.println(f);
    }
}
```



### 把文件重命名为指定的文件路径

```java
public boolean renameTo(File dest):把文件重命名为指定的文件路径
 比如：file1.renameTo(file2)为例：
    要想保证返回true,需要file1在硬盘中是存在的，且file2不能在硬盘中存在。
```

```java
public void test4(){
    File file1 = new File("hello.txt");
    File file2 = new File("D:\\io\\hi.txt");

    boolean renameTo = file2.renameTo(file1);
    System.out.println(renameTo);
}
```



### 判断文件或者目录等操作

```java
public boolean isDirectory()：判断是否是文件目录
public boolean isFile() ：判断是否是文件
public boolean exists() ：判断是否存在
public boolean canRead() ：判断是否可读
public boolean canWrite() ：判断是否可写
public boolean isHidden() ：判断是否隐藏
```

```java
public void test5(){
    File file1 = new File("hello.txt");
    file1 = new File("hello1.txt");

    System.out.println(file1.isDirectory());
    System.out.println(file1.isFile());
    System.out.println(file1.exists());
    System.out.println(file1.canRead());
    System.out.println(file1.canWrite());
    System.out.println(file1.isHidden());

    System.out.println();

    File file2 = new File("d:\\io");
    file2 = new File("d:\\io1");
    System.out.println(file2.isDirectory());
    System.out.println(file2.isFile());
    System.out.println(file2.exists());
    System.out.println(file2.canRead());
    System.out.println(file2.canWrite());
    System.out.println(file2.isHidden());

}
```



### 创建文件或者目录

```java
创建硬盘中对应的文件或文件目录
public boolean createNewFile() ：创建文件。若文件存在，则不创建，返回false
public boolean mkdir() ：创建文件目录。如果此文件目录存在，就不创建了。如果此文件目录的上层目录不存在，也不创建。
public boolean mkdirs() ：创建文件目录。如果此文件目录存在，就不创建了。如果上层文件目录不存在，一并创建

删除磁盘中的文件或文件目录
public boolean delete()：删除文件或者文件夹
    删除注意事项：Java中的删除不走回收站。
```

#### 文件操作

```java
public void test6() throws IOException {
    File file1 = new File("hi.txt");
    if(!file1.exists()){
    //文件的创建
    file1.createNewFile();
    System.out.println("创建成功");
    }else{//文件存在
    file1.delete();
    System.out.println("删除成功");
    }
}
```

#### 文件目录操作

```java
public void test7(){
    //文件目录的创建
    File file1 = new File("d:\\io\\io1\\io3");

    boolean mkdir = file1.mkdir();
    if(mkdir){
    System.out.println("创建成功1");
    }

    File file2 = new File("d:\\io\\io1\\io4");

    boolean mkdir1 = file2.mkdirs();
    if(mkdir1){
    System.out.println("创建成功2");
    }
    //要想删除成功，io4文件目录下不能有子目录或文件
    File file3 = new File("D:\\io\\io1\\io4");
    file3 = new File("D:\\io\\io1");
    System.out.println(file3.delete());
}
```



## FileInputStream和FileOutputStream的使用

### 使用场景

1. 对于文本文件(.txt,.java,.c,.cpp)，使用字符流处理
 2. 对于非文本文件(.jpg,.mp3,.mp4,.avi,.doc,.ppt,...)，使用字节流处理

### 使用字节流FileInputStream处理文本文件，可能出现乱码

```java
public void testFileInputStream() {
    FileInputStream fis = null;
    try {
        //1. 造文件
        File file = new File("hello.txt");

        //2.造流
        fis = new FileInputStream(file);

        //3.读数据
        byte[] buffer = new byte[5];
        int len;//记录每次读取的字节的个数
        while((len = fis.read(buffer)) != -1){

            String str = new String(buffer,0,len);
            System.out.print(str);

        }
    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        if(fis != null){
            //4.关闭资源
            try {
                fis.close();
            } catch (IOException e) {
                e.printStackTrace();
            }

        }
    }

}
```



### 实现对图片的复制操作

```java
public void testFileInputOutputStream()  {
    FileInputStream fis = null;
    FileOutputStream fos = null;
    try {
        //
        File srcFile = new File("爱情与友情.jpg");
        File destFile = new File("爱情与友情2.jpg");

        //
        fis = new FileInputStream(srcFile);
        fos = new FileOutputStream(destFile);

        //复制的过程
        byte[] buffer = new byte[5];
        int len;
        while((len = fis.read(buffer)) != -1){
            fos.write(buffer,0,len);
        }

    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        if(fos != null){
            //
            try {
                fos.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        if(fis != null){
            try {
                fis.close();
            } catch (IOException e) {
                e.printStackTrace();
            }

        }
    }

}
```



### 指定路径下文件的复制

```java
public void copyFile(String srcPath,String destPath){
    FileInputStream fis = null;
    FileOutputStream fos = null;
    try {
        //
        File srcFile = new File(srcPath);
        File destFile = new File(destPath);

        //
        fis = new FileInputStream(srcFile);
        fos = new FileOutputStream(destFile);

        //复制的过程
        byte[] buffer = new byte[1024];
        int len;
        while((len = fis.read(buffer)) != -1){
            fos.write(buffer,0,len);
        }

    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        if(fos != null){
            //
            try {
                fos.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        if(fis != null){
            try {
                fis.close();
            } catch (IOException e) {
                e.printStackTrace();
            }

        }
    }
}
```

```java
public void testCopyFile(){

        long start = System.currentTimeMillis();

        String srcPath = "C:\\Users\\Administrator\\Desktop\\01-视频.avi";
        String destPath = "C:\\Users\\Administrator\\Desktop\\02-视频.avi";


//        String srcPath = "hello.txt";
//        String destPath = "hello3.txt";

        copyFile(srcPath,destPath);


        long end = System.currentTimeMillis();

        System.out.println("复制操作花费的时间为：" + (end - start));//618

}
```



## FileReader和FileWriter的使用

#### 将hello.txt文件内容读入程序中，并输出到控制台

说明点：
    1. read()的理解：返回读入的一个字符。如果达到文件末尾，返回-1
    2. 异常的处理：为了保证流资源一定可以执行关闭操作。需要使用try-catch-finally处理
    3. 读入的文件一定要存在，否则就会报FileNotFoundException。



FileReader的重载方法读取文件注意事项：不要使用char[]数组的长度去遍历，否则会出现问题

```java
public void testFileReader(){
        FileReader fr = null;
        try {
            //1.实例化File类的对象，指明要操作的文件
            File file = new File("hello.txt");//相较于当前Module
            //2.提供具体的流
            fr = new FileReader(file);

            //3.数据的读入
            //read():返回读入的一个字符。如果达到文件末尾，返回-1
            //方式一：
//        int data = fr.read();
//        while(data != -1){
//            System.out.print((char)data);
//            data = fr.read();
//        }

            //方式二：语法上针对于方式一的修改
            int data;
            while((data = fr.read()) != -1){
                System.out.print((char)data);
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            //4.流的关闭操作
//            try {
//                if(fr != null)
//                    fr.close();
//            } catch (IOException e) {
//                e.printStackTrace();
//            }
            //或
            if(fr != null){
                try {
                    fr.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }

    }
```



### 对read()操作升级：使用read的重载方法

```java
public void testFileReader1()  {
        FileReader fr = null;
        try {
            //1.File类的实例化
            File file = new File("hello.txt");

            //2.FileReader流的实例化
            fr = new FileReader(file);

            //3.读入的操作
            //read(char[] cbuf):返回每次读入cbuf数组中的字符的个数。如果达到文件末尾，返回-1
            char[] cbuf = new char[5];
            int len;
            while((len = fr.read(cbuf)) != -1){
                //方式一：
                //错误的写法
//                for(int i = 0;i < cbuf.length;i++){
//                    System.out.print(cbuf[i]);
//                }
                //正确的写法
//                for(int i = 0;i < len;i++){
//                    System.out.print(cbuf[i]);
//                }
                //方式二：
                //错误的写法,对应着方式一的错误的写法
//                String str = new String(cbuf);
//                System.out.print(str);
                //正确的写法
                String str = new String(cbuf,0,len);
                System.out.print(str);
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if(fr != null){
                //4.资源的关闭
                try {
                    fr.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }

            }
        }

    }
```



### 从内存中写出数据到硬盘的文件里

说明：
    1. 输出操作，对应的File可以不存在的。并不会报异常

2. File对应的硬盘中的文件如果不存在，在输出的过程中，会自动创建此文件。
   File对应的硬盘中的文件如果存在：
          如果流使用的构造器是：FileWriter(file,false) / FileWriter(file):对原有文件的覆盖
           如果流使用的构造器是：FileWriter(file,true):不会对原有文件覆盖，而是在原有文件基础上追加内容



```java
public void testFileWriter() {
    FileWriter fw = null;
    try {
        //1.提供File类的对象，指明写出到的文件
        File file = new File("hello1.txt");

        //2.提供FileWriter的对象，用于数据的写出
        fw = new FileWriter(file,false);

        //3.写出的操作
        fw.write("I have a dream!\n");
        fw.write("you need to have a dream!");
    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        //4.流资源的关闭
        if(fw != null){

            try {
                fw.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }


}
```



```java
public void testFileReaderFileWriter() {
        FileReader fr = null;
        FileWriter fw = null;
        try {
            //1.创建File类的对象，指明读入和写出的文件
            File srcFile = new File("hello.txt");
            File destFile = new File("hello2.txt");

            //不能使用字符流来处理图片等字节数据
//            File srcFile = new File("爱情与友情.jpg");
//            File destFile = new File("爱情与友情1.jpg");


            //2.创建输入流和输出流的对象
            fr = new FileReader(srcFile);
            fw = new FileWriter(destFile);


            //3.数据的读入和写出操作
            char[] cbuf = new char[5];
            int len;//记录每次读入到cbuf数组中的字符的个数
            while((len = fr.read(cbuf)) != -1){
                //每次写出len个字符
                fw.write(cbuf,0,len);

            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            //4.关闭流资源
            //方式一：
//            try {
//                if(fw != null)
//                    fw.close();
//            } catch (IOException e) {
//                e.printStackTrace();
//            }finally{
//                try {
//                    if(fr != null)
//                        fr.close();
//                } catch (IOException e) {
//                    e.printStackTrace();
//                }
//            }
            //方式二：
            try {
                if(fw != null) {
                    fw.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }

            try {
                if(fr != null) {
                    fr.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }

        }

    }
```



## 缓冲流的使用

1.缓冲流：

```
BufferedInputStream
BufferedOutputStream
BufferedReader
BufferedWriter
```

2.作用：

- 提供流的读取、写入的速度

 *   提高读写速度的原因：内部提供了一个缓冲区

3. 处理流，就是“套接”在已有的流的基础上。



### 实现非文本文件的复制

```java
public void bufferedStreamTest() throws FileNotFoundException {
        BufferedInputStream bis = null;
        BufferedOutputStream bos = null;

        try {
            //1.造文件
            File srcFile = new File("爱情与友情.jpg");
            File destFile = new File("爱情与友情3.jpg");
            //2.造流
            //2.1 造节点流
            FileInputStream fis = new FileInputStream((srcFile));
            FileOutputStream fos = new FileOutputStream(destFile);
            //2.2 造缓冲流
            bis = new BufferedInputStream(fis);
            bos = new BufferedOutputStream(fos);

            //3.复制的细节：读取、写入
            byte[] buffer = new byte[10];
            int len;
            while((len = bis.read(buffer)) != -1){
                bos.write(buffer,0,len);

//                bos.flush();//刷新缓冲区

            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            //4.资源关闭
            //要求：先关闭外层的流，再关闭内层的流
            if(bos != null){
                try {
                    bos.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }

            }
            if(bis != null){
                try {
                    bis.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }

            }
            //说明：关闭外层流的同时，内层流也会自动的进行关闭。关于内层流的关闭，我们可以省略.
//        fos.close();
//        fis.close();
        }
    }
```



### 实现文件复制的方法

文件复制方法

```java
public void copyFileWithBuffered(String srcPath,String destPath){
        BufferedInputStream bis = null;
        BufferedOutputStream bos = null;

        try {
            //1.造文件
            File srcFile = new File(srcPath);
            File destFile = new File(destPath);
            //2.造流
            //2.1 造节点流
            FileInputStream fis = new FileInputStream((srcFile));
            FileOutputStream fos = new FileOutputStream(destFile);
            //2.2 造缓冲流
            bis = new BufferedInputStream(fis);
            bos = new BufferedOutputStream(fos);

            //3.复制的细节：读取、写入
            byte[] buffer = new byte[1024];
            int len;
            while((len = bis.read(buffer)) != -1){
                bos.write(buffer,0,len);
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            //4.资源关闭
            //要求：先关闭外层的流，再关闭内层的流
            if(bos != null){
                try {
                    bos.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }

            }
            if(bis != null){
                try {
                    bis.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }

            }
            //说明：关闭外层流的同时，内层流也会自动的进行关闭。关于内层流的关闭，我们可以省略.
//        fos.close();
//        fis.close();
        }
    }
```

测试文件复制方法

```Java
public void testCopyFileWithBuffered(){
    long start = System.currentTimeMillis();

    String srcPath = "C:\\Users\\Administrator\\Desktop\\01-视频.avi";
    String destPath = "C:\\Users\\Administrator\\Desktop\\03-视频.avi";


    copyFileWithBuffered(srcPath,destPath);


    long end = System.currentTimeMillis();

    System.out.println("复制操作花费的时间为：" + (end - start));//618 - 176
}
```



### 使用BufferedReader和BufferedWriter实现文本文件的复制

```java
public void testBufferedReaderBufferedWriter(){
        BufferedReader br = null;
        BufferedWriter bw = null;
        try {
            //创建文件和相应的流
            br = new BufferedReader(new FileReader(new File("dbcp.txt")));
            bw = new BufferedWriter(new FileWriter(new File("dbcp1.txt")));

            //读写操作
            //方式一：使用char[]数组
//            char[] cbuf = new char[1024];
//            int len;
//            while((len = br.read(cbuf)) != -1){
//                bw.write(cbuf,0,len);
//    //            bw.flush();
//            }

            //方式二：使用String
            String data;
            while((data = br.readLine()) != null){
                //方法一：
//                bw.write(data + "\n");//data中不包含换行符
                //方法二：
                bw.write(data);//data中不包含换行符
                bw.newLine();//提供换行的操作

            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            //关闭资源
            if(bw != null){

                try {
                    bw.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if(br != null){
                try {
                    br.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
```



## 标准的输入、输出流

```
1.标准的输入、输出流
1.1
System.in:标准的输入流，默认从键盘输入
System.out:标准的输出流，默认从控制台输出
1.2
System类的setIn(InputStream is) / setOut(PrintStream ps)方式重新指定输入和输出的流。

1.3练习：
从键盘输入字符串，要求将读取到的整行字符串转成大写输出。然后继续进行输入操作，
直至当输入“e”或者“exit”时，退出程序。

方法一：使用Scanner实现，调用next()返回一个字符串
方法二：使用System.in实现。System.in  --->  转换流 ---> BufferedReader的readLine()
```

```java
public static void main(String[] args) {
    BufferedReader br = null;
    try {
        InputStreamReader isr = new InputStreamReader(System.in);
        br = new BufferedReader(isr);

        while (true) {
            System.out.println("请输入字符串：");
            String data = br.readLine();
            if ("e".equalsIgnoreCase(data) || "exit".equalsIgnoreCase(data)) {
                System.out.println("程序结束");
                break;
            }

            String upperCase = data.toUpperCase();
            System.out.println(upperCase);

        }
    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        if (br != null) {
            try {
                br.close();
            } catch (IOException e) {
                e.printStackTrace();
            }

        }
    }
}
```



## 打印流PrintStream 和PrintWriter

```java
public void test2() {
    PrintStream ps = null;
    try {
        FileOutputStream fos = new FileOutputStream(new File("D:\\IO\\text.txt"));
        // 创建打印输出流,设置为自动刷新模式(写入换行符或字节 '\n' 时都会刷新输出缓冲区)
        ps = new PrintStream(fos, true);
        if (ps != null) {// 把标准输出流(控制台输出)改成文件
            System.setOut(ps);
        }


        for (int i = 0; i <= 255; i++) { // 输出ASCII字符
            System.out.print((char) i);
            if (i % 50 == 0) { // 每50个数据一行
                System.out.println(); // 换行
            }
        }


    } catch (FileNotFoundException e) {
        e.printStackTrace();
    } finally {
        if (ps != null) {
            ps.close();
        }
    }

}
```



## 数据流DataInputStream 和 DataOutputStream

作用：用于读取或写出基本数据类型的变量或字符串

```java
public void test3() throws IOException {
    //1.
    DataOutputStream dos = new DataOutputStream(new FileOutputStream("data.txt"));
    //2.
    dos.writeUTF("刘建辰");
    dos.flush();//刷新操作，将内存中的数据写入文件
    dos.writeInt(23);
    dos.flush();
    dos.writeBoolean(true);
    dos.flush();
    //3.
    dos.close();
}
```

将文件中存储的基本数据类型变量和字符串读取到内存中，保存在变量中。

注意点：读取不同类型的数据的顺序要与当初写入文件时，保存的数据的顺序一致！

```java
public void test4() throws IOException {
    //1.
    DataInputStream dis = new DataInputStream(new FileInputStream("data.txt"));
    //2.
    String name = dis.readUTF();
    int age = dis.readInt();
    boolean isMale = dis.readBoolean();

    System.out.println("name = " + name);
    System.out.println("age = " + age);
    System.out.println("isMale = " + isMale);

    //3.
    dis.close();

}
```



## 对象流ObjectInputStream和ObjectOutputStream

用于存储和读取基本数据类型数据或对象的处理流。它的强大之处就是可以把Java中的对象写入到数据源中，也能把对象从数据源中还原回来。

### 序列化

**将内存中的java对象保存到磁盘中或通过网络传输出去,使用ObjectOutputStream实现**

```java
public void testObjectOutputStream(){
    ObjectOutputStream oos = null;

    try {
        //1.
        oos = new ObjectOutputStream(new FileOutputStream("object.dat"));
        //2.
        oos.writeObject(new String("我爱北京天安门"));
        oos.flush();//刷新操作

        oos.writeObject(new Person("王铭",23));
        oos.flush();

        oos.writeObject(new Person("张学良",23,1001,new Account(5000)));
        oos.flush();

    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        if(oos != null){
            //3.
            try {
                oos.close();
            } catch (IOException e) {
                e.printStackTrace();
            }

        }
    }

}
```



### 反序列化

**将磁盘文件中的对象还原为内存中的一个java对象使用ObjectInputStream来实现**

```java
public void testObjectInputStream(){
    ObjectInputStream ois = null;
    try {
        ois = new ObjectInputStream(new FileInputStream("object.dat"));

        Object obj = ois.readObject();
        String str = (String) obj;

        Person p = (Person) ois.readObject();
        Person p1 = (Person) ois.readObject();

        System.out.println(str);
        System.out.println(p);
        System.out.println(p1);

    } catch (IOException e) {
        e.printStackTrace();
    } catch (ClassNotFoundException e) {
        e.printStackTrace();
    } finally {
        if(ois != null){
            try {
                ois.close();
            } catch (IOException e) {
                e.printStackTrace();
            }

        }
    }
}
```



## RandomAccessFile的使用



1. `RandomAccessFile`直接继承于`java.lang.Object`类，实现了`DataInput`和`DataOutput`接口
2. `RandomAccessFile`既可以作为一个输入流，又可以作为一个输出流

3. 如果`RandomAccessFile`作为输出流时，写出到的文件如果不存在，则在执行过程中自动创建。如果写出到的文件存在，则会对原有文件内容进行覆盖。（默认情况下，从头覆盖）

  4. 可以通过相关的操作，实现`RandomAccessFile`“插入”数据的效果



```java
public void test1() {

    RandomAccessFile raf1 = null;
    RandomAccessFile raf2 = null;
    try {
        //1.
        raf1 = new RandomAccessFile(new File("爱情与友情.jpg"),"r");
        raf2 = new RandomAccessFile(new File("爱情与友情1.jpg"),"rw");
        //2.
        byte[] buffer = new byte[1024];
        int len;
        while((len = raf1.read(buffer)) != -1){
            raf2.write(buffer,0,len);
        }
    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        //3.
        if(raf1 != null){
            try {
                raf1.close();
            } catch (IOException e) {
                e.printStackTrace();
            }

        }
        if(raf2 != null){
            try {
                raf2.close();
            } catch (IOException e) {
                e.printStackTrace();
            }

        }
    }
}
```



```java
public void test2() throws IOException {

    RandomAccessFile raf1 = new RandomAccessFile("hello.txt","rw");

    raf1.seek(3);//将指针调到角标为3的位置
    raf1.write("xyz".getBytes());//

    raf1.close();

}
```



使用RandomAccessFile实现数据的插入效果

```java
public void test3() throws IOException {

    RandomAccessFile raf1 = new RandomAccessFile("hello.txt","rw");

    raf1.seek(3);//将指针调到角标为3的位置
    //保存指针3后面的所有数据到StringBuilder中
    StringBuilder builder = new StringBuilder((int) new File("hello.txt").length());
    byte[] buffer = new byte[20];
    int len;
    while((len = raf1.read(buffer)) != -1){
        builder.append(new String(buffer,0,len)) ;
    }
    //调回指针，写入“xyz”
    raf1.seek(3);
    raf1.write("xyz".getBytes());

    //将StringBuilder中的数据写入到文件中
    raf1.write(builder.toString().getBytes());

    raf1.close();

    //思考：将StringBuilder替换为ByteArrayOutputStream
}
```



## Path、 Paths、 Files类的使用

### Path接口

```java
String toString() ： 返回调用 Path 对象的字符串表示形式
boolean startsWith(String path) : 判断是否以 path 路径开始
boolean endsWith(String path) : 判断是否以 path 路径结束
boolean isAbsolute() : 判断是否是绝对路径
Path getParent() ：返回Path对象包含整个路径，不包含 Path 对象指定的文件路径
Path getRoot() ：返回调用 Path 对象的根路径
Path getFileName() : 返回与调用 Path 对象关联的文件名
int getNameCount() : 返回Path 根目录后面元素的数量
Path getName(int idx) : 返回指定索引位置 idx 的路径名称
Path toAbsolutePath() : 作为绝对路径返回调用 Path 对象
Path resolve(Path p) :合并两个路径，返回合并后的路径对应的Path对象
File toFile(): 将Path转化为File类的对象
```



### Files类

Files工具类的使用：操作文件或目录的工具类

```java
 java.nio.file.Files 用于操作文件或目录的工具类。
     
Files常用方法：
 Path copy(Path src, Path dest, CopyOption … how) : 文件的复制
 Path createDirectory(Path path, FileAttribute<?> … attr) : 创建一个目录
 Path createFile(Path path, FileAttribute<?> … arr) : 创建一个文件
 void delete(Path path) : 删除一个文件/目录，如果不存在，执行报错
 void deleteIfExists(Path path) : Path对应的文件/目录如果存在，执行删除
 Path move(Path src, Path dest, CopyOption…how) : 将 src 移动到 dest 位置
 long size(Path path) : 返回 path 指定文件的大小

Files常用方法：用于判断
 boolean exists(Path path, LinkOption … opts) : 判断文件是否存在
 boolean isDirectory(Path path, LinkOption … opts) : 判断是否是目录
 boolean isRegularFile(Path path, LinkOption … opts) : 判断是否是文件
 boolean isHidden(Path path) : 判断是否是隐藏文件
 boolean isReadable(Path path) : 判断文件是否可读
 boolean isWritable(Path path) : 判断文件是否可写
 boolean notExists(Path path, LinkOption … opts) : 判断文件是否不存在

 Files常用方法：用于操作内容
 SeekableByteChannel newByteChannel(Path path, OpenOption…how) : 获取与指定文件的连
接， how 指定打开方式。
 DirectoryStream<Path> newDirectoryStream(Path path) : 打开 path 指定的目录
 InputStream newInputStream(Path path, OpenOption…how):获取 InputStream 对象
 OutputStream newOutputStream(Path path, OpenOption…how) : 获取 OutputStream 对象
```



```java
public class FilesTest {

   @Test
   public void test1() throws IOException{
      Path path1 = Paths.get("d:\\nio", "hello.txt");
      Path path2 = Paths.get("atguigu.txt");
      
//    Path copy(Path src, Path dest, CopyOption … how) : 文件的复制
      //要想复制成功，要求path1对应的物理上的文件存在。path1对应的文件没有要求。
//    Files.copy(path1, path2, StandardCopyOption.REPLACE_EXISTING);
      
//    Path createDirectory(Path path, FileAttribute<?> … attr) : 创建一个目录
      //要想执行成功，要求path对应的物理上的文件目录不存在。一旦存在，抛出异常。
      Path path3 = Paths.get("d:\\nio\\nio1");
//    Files.createDirectory(path3);
      
//    Path createFile(Path path, FileAttribute<?> … arr) : 创建一个文件
      //要想执行成功，要求path对应的物理上的文件不存在。一旦存在，抛出异常。
      Path path4 = Paths.get("d:\\nio\\hi.txt");
//    Files.createFile(path4);
      
//    void delete(Path path) : 删除一个文件/目录，如果不存在，执行报错
//    Files.delete(path4);
      
//    void deleteIfExists(Path path) : Path对应的文件/目录如果存在，执行删除.如果不存在，正常执行结束
      Files.deleteIfExists(path3);
      
//    Path move(Path src, Path dest, CopyOption…how) : 将 src 移动到 dest 位置
      //要想执行成功，src对应的物理上的文件需要存在，dest对应的文件没有要求。
//    Files.move(path1, path2, StandardCopyOption.ATOMIC_MOVE);
      
//    long size(Path path) : 返回 path 指定文件的大小
      long size = Files.size(path2);
      System.out.println(size);

   }

   @Test
   public void test2() throws IOException{
      Path path1 = Paths.get("d:\\nio", "hello.txt");
      Path path2 = Paths.get("atguigu.txt");
//    boolean exists(Path path, LinkOption … opts) : 判断文件是否存在
      System.out.println(Files.exists(path2, LinkOption.NOFOLLOW_LINKS));

//    boolean isDirectory(Path path, LinkOption … opts) : 判断是否是目录
      //不要求此path对应的物理文件存在。
      System.out.println(Files.isDirectory(path1, LinkOption.NOFOLLOW_LINKS));

//    boolean isRegularFile(Path path, LinkOption … opts) : 判断是否是文件

//    boolean isHidden(Path path) : 判断是否是隐藏文件
      //要求此path对应的物理上的文件需要存在。才可判断是否隐藏。否则，抛异常。
//    System.out.println(Files.isHidden(path1));

//    boolean isReadable(Path path) : 判断文件是否可读
      System.out.println(Files.isReadable(path1));
//    boolean isWritable(Path path) : 判断文件是否可写
      System.out.println(Files.isWritable(path1));
//    boolean notExists(Path path, LinkOption … opts) : 判断文件是否不存在
      System.out.println(Files.notExists(path1, LinkOption.NOFOLLOW_LINKS));
   }

   /**
    * StandardOpenOption.READ:表示对应的Channel是可读的。
    * StandardOpenOption.WRITE：表示对应的Channel是可写的。
    * StandardOpenOption.CREATE：如果要写出的文件不存在，则创建。如果存在，忽略
    * StandardOpenOption.CREATE_NEW：如果要写出的文件不存在，则创建。如果存在，抛异常
    *
    * @author shkstart 邮箱：shkstart@126.com
    * @throws IOException
    */
   @Test
   public void test3() throws IOException{
      Path path1 = Paths.get("d:\\nio", "hello.txt");

//    InputStream newInputStream(Path path, OpenOption…how):获取 InputStream 对象
      InputStream inputStream = Files.newInputStream(path1, StandardOpenOption.READ);

//    OutputStream newOutputStream(Path path, OpenOption…how) : 获取 OutputStream 对象
      OutputStream outputStream = Files.newOutputStream(path1, StandardOpenOption.WRITE,StandardOpenOption.CREATE);


//    SeekableByteChannel newByteChannel(Path path, OpenOption…how) : 获取与指定文件的连接，how 指定打开方式。
      SeekableByteChannel channel = Files.newByteChannel(path1, StandardOpenOption.READ,StandardOpenOption.WRITE,StandardOpenOption.CREATE);

//    DirectoryStream<Path>  newDirectoryStream(Path path) : 打开 path 指定的目录
      Path path2 = Paths.get("e:\\teach");
      DirectoryStream<Path> directoryStream = Files.newDirectoryStream(path2);
      Iterator<Path> iterator = directoryStream.iterator();
      while(iterator.hasNext()){
         System.out.println(iterator.next());
      }


   }
}
```



# 序列化

## 序列化机制

>  对象序列化机制允许把内存中的Java对象转换成平台无关的二进制流，从而允许把这种二进制流持久地保存在磁盘上，或通过网络将这种二进制流传输到另一个网络节点。当其它程序获取了这种二进制流，就可以恢复成原来的Java对象。

## 序列化的好处

在于可将任何实现了Serializable接口的对象转化为字节数据，使其在保存和传输时可被还原



## 序列化注意事项

- 序列化是 RMI（Remote Method Invoke – 远程方法调用）过程的参数和返回值都必须实现的机制，而 RMI 是 JavaEE 的基础。因此序列化机制是JavaEE 平台的基础

- 如果需要让某个对象支持序列化机制，则必须让对象所属的类及其属性是可序列化的，为了让某个类是可序列化的，该类必须实现如下两个接口之一。否则，会抛出NotSerializableException异常
  - Serializable
  - Externalizable

- 凡是实现Serializable接口的类都有一个表示序列化版本标识符的静态变量：
  - private static final long serialVersionUID;
  - serialVersionUID用来表明类的不同版本间的兼容性。 简言之，其目的是以序列化对象进行版本控制，有关各版本反序列化时是否兼容。
  - 如果类没有显示定义这个静态常量，它的值是Java运行时环境根据类的内部细节自动生成的。 若类的实例变量做了修改， serialVersionUID 可能发生变化。 故建议，显式声明。
- 简单来说， Java的序列化机制是通过在运行时判断类的serialVersionUID来验证版本一致性的。在进行反序列化时， JVM会把传来的字节流中的serialVersionUID与本地相应实体类的serialVersionUID进行比较，如果相同就认为是一致的，可以进行反序列化，否则就会出现序列化版本不一致的异常。 (InvalidCastException )



## 序列化的使用

### 序列化

若某个类实现了 Serializable 接口，该类的对象就是可序列化的：

- 创建一个 ObjectOutputStream

- 调用 ObjectOutputStream 对象的 writeObject(对象) 方法输出可序列化对象

- 注意写出一次，操作flush()一次、

  

### 反序列化

- 创建一个 ObjectInputStream
- 调用 readObject() 方法读取流中的对象

**强调： 如果某个类的属性不是基本数据类型或 String 类型，而是另一个引用类型，那么这个引用类型必须是可序列化的，否则拥有该类型的Field 的类也不能序列化**



### 示例

序列化：将对象写入到磁盘或者进行网络传输。
要求对象必须实现序列化

```java
ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(“data.txt"));
Person p = new Person("韩梅梅", 18, "中华大街", new Pet());
oos.writeObject(p);
oos.flush();
oos.close();
```

反序列化：将磁盘中的对象数据源读出。

```java
ObjectInputStream ois = new ObjectInputStream(new FileInputStream(“data.txt"));
Person p1 = (Person)ois.readObject();
System.out.println(p1.toString());
ois.close();
```

## 谈谈你对java.io.Serializable接口的理解，我们知道它用于序列化，是空方法接口，还有其它认识吗？

- 实现了Serializable接口的对象，可将它们转换成一系列字节，并可在以后完全恢复回原来的样子。 **这一过程亦可通过网络行。这意味着序列化机制能自动补偿操作系统间的差异。** 换句话说，可以先在Windows机器上创建一个对象，对其序列化，然后通过网络发给一台Unix机器，然后在那里准确无误地重新“装配”。不必关心数据在不同机器上如何表示，也不必关心字节的顺或者其他任何细节。
- 由于大部分作为参数的类如String、 Integer等都实现了java.io.Serializable的接口，也可以利用多态的性质，作为参数使接口更灵活。

## 