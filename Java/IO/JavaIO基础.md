# IO流

IO即 `Input/Output`， 输入和输出，数据输入到计算机内存的过程即输入，反之输出到外部存储（比如数据库，文件，远程主机）的过程即输出，数据传输过程类似于水流，因此称为IO流。IO流在Java中分为输入流和输出流，而根据数据的处理方式又分为字节流和字符流。

Java IO流的40多个类都是从如下4个抽象类基类中派生出来的。
- `InputStream`/`Reader`：所有的输入流的基类，前者是字节输入流，后者是字符输入流。
- `OutputStream`/`Writer`：所有输出流的基类，前者是字节输出流，后者是字符输出流。

# 字节流

## InputStream（字节输入流）

`InputStream` 用于从源头（通常是文件）读取数据（字节信息）到内存中，`java.io.InputStream`抽象类是所有字节输入流的父类。

`InputStream`常用方法：
- `read()`：返回输入流中下一个字节的数据，返回的值介于0到255之间。如果未读取任何字节，则代码返回`-1`，表示文件结束。
- `read(byte[] b)`：从输入流中读取一些字节存储到数组`b`中，如果数组`b`的长度为零，则不读取。如果没有可用字节读取，返回`-1`。如果有可用字节读取，则最多读取的字节数最多等于`b.length`，返回读取的字节数。这个方法等价于`read(b, 0, b.length)`。
- `read(byte[] b, int off, int len)`：在`read(byte[] b)`方法的基础上增加了`off` 参数（偏移量）和`len` 参数（要读取的最大字节数）。
- `skip(long n)`：忽略输入流中的n个字符，返回实际忽略的字节数。
- `available()`：返回输入流中可以读取的字节数。
- `close()`：关闭输入流释放相关的系统资源。

从Java9开始，`InputStream`新增加了多个实用的方法：

- `readAllBytes()`：读取输入流中的所有字节，返回字节数组。
- `readNBytes(byte[] b, int off, int len)`：阻塞直到读取`len` 个字符。
- `transferTo(OutputStream out)`：将所有字节从一个输入流传递到一个输出流。

`FileInputStream`是一个比较常用的字节输入流对象，可直接指定文件路径，可以直接读取单字节数据，也可以读取至字节数组中。

`FileInputStream`：
```java
try (InputStream fis = new FileInputStream("input.txt")) {
    System.out.println("Number of remaining bytes:" + fis.available());
    int content;
    long skip = fis.skip(2);
    System.out.println("The actual number of bytes skipped:" + skip);
    System.out.println("The content read from file:");
    while((content = fis.read()) != -1) {
        System.out.println((char) content);
    }
} catch(IOException e) {
    e.printStackTrace();
}
```

不过，一般我们不会直接单独使用 `FileInputStream`，通常会配合`BufferedInputStream`（字节缓冲输入流，后文会讲到）来使用。

像下面这段代码在我们的项目中比较常见，我们通过`readAllBytes()` 读取输入流所有字节并将其直接赋值给一个`String` 对象。

```java
// 新建一个 BufferedInputStream 对象
BufferedInputStream bufferedInputStream = new BufferedInputStream(new FileInputStream("path"));
// 读取文件的内容并复制到 String 对象中
String result = new String(bufferedInputStream.readAllBytes());
System.out.println(result);
```

`DataInputStream` 用于读取指定类型数据，不能单独使用，必须结合其他流，比如`FileInputStream`。

```java
FileInputStream fileInputStream = new FileInputStream("path");
// 必须将fileInputStream 作为构造参数才能使用
DataInputStream dataInputStream = new DataInputStream(fileInputStream);
// 可以读取任意具体的数据类型
dataInputStream.readBoolean();
dataInputStream.readInt();
dataInputStream.readUTF();
```

`ObjectInputStream` 用于从输入流中读取Java对象（反序列化），`ObjectOutputStream` 用于将对象写入到输出流（序列化）

```java
ObjectInputStream input = new ObjectInputStream(new FileInputStream("path"));
MyClass object = (MyClass) input.readObject();
input.close();
```

另外，用于序列化和反序列化的类必须实现 `Serializable` 接口，对象中如果有属性不想被序列化，使用 `transient` 修饰。

## OutputStream(字节输出流)

`OutputStream`用于将数据（字节信息）写入到目的地（通常是文件），`java.io.OutputStream`抽象类是所有字节输出流的父类。

`OutputStream`常用方法：
- `write(int b)`：将特定字节写入输出流。
- `write(byte[] b)`：将数组`b`写入到输出流，等价于`write(b, 0, b.length)`。
- `write(byte[] b, int off, int len)`：在`write(byte[] b)`方法的基础上增加了`off`参数（偏移量）和`len`参数（要读取的最大字节数）。
- `flush()`：刷新此输出流并强制写出所有缓冲的输出字节。
- `close()`：关闭输出流释放相关系统资源。

`FileOutputStream` 是最常用的字节输出流对象，可直接指定文件路径，可以直接输出单字节数据，也可以输出指定的字节数组。

`FileInputStream`：
```java
// 将 字节 输出到path文件
try (FileOutputStream output = new FileOutputStream("path")) {
    byte[] array = "Momochi Kiruya".getBytes();
    output.write(array);
} catch(IOException e) {
    e.printStackTrace();
}
```

类似于 `FileInputStream`， `FileOutputStream` 通常也会配合 `BufferedOutputStream` （字节缓冲输出流，后文会讲到）来使用。
```java
FileOutputStream fileOutputStream = new FileOutputStream("path");
BufferedOutputStream bos = new BufferedOutputStream(fileOutputStream);
```

`DataOutputStream` 用于写入指定类型数据不能单独使用，必须结合其它流，比如`FileOutputStream`。

```java
FileOutputStream fileOutputStream = new FileOutputStream("path");
DataOutputStream dataOutputStream = new DataOutputStream(fileOutputStream);

dataOutputStream.writeBoolean(true);
dataOutputStream.writeByte(1);
```

`ObjectInputStream` 用于从输入流中读取 Java对象（`ObjectInputStream`，反序列化），`ObjectOutputStream`将对象写入到输出流（`ObjectOutputStream`，序列化）。

```java
ObjectOutputStream output = new ObjectOutputStream(new FileOutputStream("path"));
Message message = new Message("Kiruya", "Kokoro");
output.writeObject(message);
```

## 字符流

不管是文件读写还是网络发送接收，信息的最小存储单元都是字节，那**为什么I/O流操作要分为字节流操作和字符流操作呢？**

- 字符流是由Java虚拟机将字节转换得到的，这个过程消耗时间。
- 如果我们不知道编码类型，很容易出现乱码问题。
- 字节流相对更底层，在处理多种数据时更具通用性，字符流在特定的文本处理场景下能提供更好的效率和便利性。
- 字符流主要处理字符数据，更适合处理文本类型的数据，能自动进行字符编码转换。字节流可以处理各种类型的数据，包括二进制数据等。

I/O流就干脆提供了一个直接操作字符的接口，方便我们平时对字符进行流操作。如果音频文件、图片等媒体文件用字节流比较好，如果涉及到字符的话使用字符流比较好。

字符流默认采用的是`Unicode`编码，我们可以通过构造方法自定义编码。常用字符编码所占字节数：`utf8`:英文占`1`字节，中文`3`字节。`unicode`：任何字符都占`2`字节。`gbk`：英文占`1`字节，中文占`2`字节。

## Reader （字符输入流）

`Reader`用于从源头（通常是文件）读取数据（字符信息）到内存中，`java.io.Reader`抽象类是所有字符输入流的父类。

`Reader`用于读取文本，`InputStream` 用于读取原始字节。

`Reader`常用方法：
- `read()`：从输入流读取一个字符。
- `read(char[] cbuf)`：从输入流中读取一些字符，并将它们存储到字符数组`cbuf`中，等价于`read(cbuf, 0, cbuf.length)`。
- `read(char[] cbuf, int off, int len)`：在`read(char[] cbuf)`方法的基础上增加了 `off` 参数（偏移量）和`len` 参数（要读取的最大字符数）。
- `skip(long n)`：忽略输入流中的n个字符，返回实际忽略的字符数。
- `close()`：关闭输入流，并释放相关的系统资源。

`InputStreamReader` 是字节流转换为字符流的桥梁，其子类`FileReader`是基于该基础上的封装，可以直接操作字符文件。

```java
public class InputStreamReader extends Reader{}

public class FileReader extends InputStreamReader{}
```

`FileReader`：

```java
try (FileReader fileReader = new FileReader("path")) {
    int content;
    long skip = fileReader.skip(3);
    System.out.println("The actual number of bytes skipped" + skip);
    System.out.println("The content read from file:");
    while ((content = fileReader.read()) != -1) {
        System.out.println((char) content);
    }
} catch(IOException e) {
    e.printStackTrace();
}
```

## Writer（字符输出流）

`Writer` 用于将数据（字符信息）写入到目的地（通常是文件），`java.io.Writer`抽象类是所有字符输出流的父类。

`Writer` 常用方法：
- `write(int c)`：写入单个字符。
- `write(char[] cbuf)`：写入字符数组`cbuf`，等价于`write(cbuf, 0, cbuf.length)`。
- `write(char[] cbuf, int off, int len)`：在`write(char[] cbuf)`方法的基础上增加了`off`参数（偏移量）和`len` 参数（要读取的最大字符数）。
- `write(String str)`：写入字符串，等价于`write(str, 0, str.length)`。
- `write(String str, int off, int len)`：在`write(String str)`方法的基础上增加了`off`参数（偏移量）和`len`参数（要读取的最大字符数）。
- `append(CharSequence csq)`：将指定的字符序列附加到指定的`Writer`对象并返回该`Writer`对象。
- `append(char c)`：将指定的字符附加到指定的`Writer`对象并返回该`Writer`对象。
- `flush()`：刷新此输出流并强制写出所有缓冲的输出字符。
- `close()`：关闭输出流释放相关的系统资源。

`OutputStreamWriter`是字符流转换为字节流的桥梁，其子类`FileWriter`是基于该基础上的封装，可以直接将字符写入到文件。

```java
public class OutputStreamWriter extends Writer {}

public class FileWriter extends OutputStreamWriter {}
```

`FileWriter`：
```java
try (Writer output = new FileWriter("path")) {
    output.write("Kiruya");
} catch (IOEXception e) {
    e.printStackTrace();
}
```

## 字节缓冲流

IO操作是很消耗性能的，缓冲流将数据加载至缓冲区，一次性读取/写入多个字节，从而避免频繁的IO操作，提高流的传输效率。

字节缓冲流这里采用了装饰器模式来增强`InputStream`和`OutputStream`子类对象的功能。

举个例子，我们可以通过`BufferedInputStream` （字节缓冲输入流）来增强`FileInputStream`的功能。
```java
BufferedInputStream bufferedInputStream = new BufferedInputStream(new FileInputStream("path"));
```

字节流和字节缓冲流的性能差别主要体现在我们使用两者的时候都是调用 `write(int b)` 和 `read()` 这两个一次只读取一个字节的方法的时候，由于字节缓冲流内部有缓冲区（字节数组），因此，字节缓冲流会先将读取到的字节存放在缓存区，大幅减少IO次数，提高读取效率。

```java
@Test
void copy_pdf_to_another_pdf_buffer_stream() {
    // 记录开始时间
    long start = System.currentTimeMillis();
    try (BufferedInputStream bis = new BufferedInputStream(new FileInputStream("path"));
         BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream("path2"))) {
        int content;
        while ((content = bis.read()) != -1) {
            bos.write(content);
        }
    } catch (IOException e) {
        e.printStackTrace();
    }
    // 记录结束时间
    long end = System.currentTimeMillis();
    System.out.println("使用缓冲流复制文件总耗时:" + (end - start) + " 毫秒");
}

@Test
void copy_pdf_to_another_pdf_stream() {
    // 记录开始时间
    long start = System.currentTimeMillis();
    try (FileInputStream fis = new FileInputStream("path");
         FileOutputStream fos = new FileOutputStream("path2")) {
        int content;
        while ((content = fis.read()) != -1) {
            fos.write(content);
        }
    } catch (IOException e) {
        e.printStackTrace();
    }
    // 记录结束时间
    long end = System.currentTimeMillis();
    System.out.println("使用普通流复制文件总耗时:" + (end - start) + " 毫秒");
}
```

如果是调用 `read(byte[] b)` 和 `write(byte[] b, int off, int len)` 这两个写入一个字节数组的方法的话，只要字节数组的大小合适，两者的性能差距其实不大，基本可以忽略。

## BufferedInputStream（字节缓冲输入流）

`BufferedInputStream` 从源头（通常是文件）读取数据（字节信息）到内存的过程中不会一个字节一个字节的读取，而是会先将读取到的字节存放在缓冲区，并从内部缓冲区中单独读取字节，这样大幅减少了IO次数，提高了读取效率。

`BufferedInputStream` 内部维护了一个缓冲区，这个缓冲区实际就是一个字节数组：
```java
public class BufferedInputStream extends FilterInputStream {
    // 内部缓冲区数组
    protected volatile byte[] buf;
    // 缓冲区的默认大小
    private static int DEFAULT_BUFFER_SIZE = 8192;
    // 使用默认的缓冲区大小
    public BufferedInputStream(InputStream in) {
        this(in, DEFAULT_BUFFER_SIZE);
    }
    // 自定义缓冲区大小
    public BufferedInputSTream(InputStream in, int size) {
        super(in);
        if (size <= 0) {
            throw new IllegalArgumentException("Buffer size <= 0");
        }
        buf = new byte[size];
    }
}
```

缓冲区的大小默认为 8192 字节，当然，你也可以通过 `BufferedInputStream(InputStream in, int size)` 这个构造方法来指定缓冲区的大小。

## BufferedOutputStream（字节缓冲输出流）

`BufferedOutputStream` 将数据（字节信息）写入到目的地（通常是文件）的过程中不会一个字节一个字节的写入，而是会先将要写入的字节存放在缓存区，并从内部缓冲区中单独写入字节。这样大幅减少了IO次数，提高了读取效率。

```java
try (BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream("path"))) {
    byte[] array = "Kiruya".getBytes();
    bos.write(array);
} catch(IOException e) {
    e.printStackTrace();
}
```

类似于 `BufferedInputStream`，`BufferedOutputStream` 内部也维护了一个缓冲区，并且，这个缓冲区的大小也是8192字节。

## 字符缓冲流

`BufferedReader`（字符缓冲输入流）和 `BufferedWriter`（字符缓冲输出流）类似于 `BufferedInputStream`（字节缓冲输入流）和 `BufferedOutputStream` （字节缓冲输入流），内部维护了一个字节数组作为缓冲区。不过前者主要用来操作字符信息。

## 打印流

```java
System.out.println("Hello World!");
```

`System.out` 实际是用于获取一个 `PrintStream` 对象，`print` 方法实际调用的是 `PrintStream` 对象的`write` 方法。

`PrintStream` 属于字节打印流，与之对应的是 `PrintWriter` （字符打印流）。`PrintStream` 是 `OutputStream` 的子类， `PrintWriter` 是`Writer` 的子类。

```java
public class PrintStream extends FilterOutputStream 
    implements Appendable, Closeable{

}

public class PrintWriter extends Writer {}
```

## 随机访问流

这里要介绍的随机访问流指的是支持随意跳转到文件的任意位置进行读写的`RandomAccessFile`。

`RandomAccessFile` 的构造方法如下,`mode`可以指定（读写模式）：
```java
public RandomAccessFile(File file, String mode) throws FileNotFoundException {
    this(file, mode, false);
}

private RandomAccessFile(File file, String mode, boolean openAndDelete)
    throws FileNotFoundExceptino {

}
```

读写模式主要有下面四种：
- `r`：只读模式
- `rw`：读写模式
- `rws`：读写，同步更新对“文件的内容”或“元数据”的修改到外部存储设备。
- `rwd`：读写，同步更新对“文件的内容”的修改到外部存储设备

文件内容指的是文件中实际保存的数据，元数据则是用来描述文件属性比如文件的大小信息、创建和修改时间。

`RandomAccessFile` 中有一个文件指针用来表示下一个将要被写入或者读取的字节所处的位置。我们可以通过`RandomAccessFile`的`seek(long pos)` 方法来设置文件指针的偏移量（距文件开头`pos`个字节处）。如果想要获取文件指针当前的位置的话，可以使用`getFilePointer()`方法。

`RandomAccessFile`：文件内容为"ABCDEFG"
```java
RandomAccessFile randomAccessFile = new RandomAccessFile(new File("path"), "rw");
System.out.println("读取之前的偏移量：" + randomAccessFile.getFilePointer() + ",当前读取到的字符" + (char) randomAccessFile.read() + "，读取之后的偏移量：" + randomAccessFile.getFilePointer());
// 指针当前偏移量为 6
randomAccessFile.seek(6);
System.out.println("读取之前的偏移量：" + randomAccessFile.getFilePointer() + ",当前读取到的字符" + (char) randomAccessFile.read() + "，读取之后的偏移量：" + randomAccessFile.getFilePointer());
// 从偏移量 7 的位置开始往后写入字节数据
randomAccessFile.write(new byte[]{'H', 'I', 'J', 'K'});
// 指针当前偏移量为 0，回到起始位置
randomAccessFile.seek(0);
System.out.println("读取之前的偏移量：" + randomAccessFile.getFilePointer() + ",当前读取到的字符" + (char) randomAccessFile.read() + "，读取之后的偏移量：" + randomAccessFile.getFilePointer());

```

`RandomAccessFile`的`write`方法在写入对象的时候如果对应的位置已经有数据的话，会将其覆盖掉。
```java
RandomAccessFile randomAccessFile = new RandomAccessFile(new File("path"), "rw");
randomAccessFile.write(new byte[] {'H', 'I', 'J', 'K'});
```

假设运行上面这段程序之前文件内容变为`ABCD`，运行之后则变为`HIJK`。

`RandomAccessFile`比较常见的一个应用就是实现大文件的 **断点续传**，何为断点续传，就是上传文件中途暂停或者失败（比如遇到网络问题）之后，不需要重新上传，只需要上传那些未成功上传的文件分片即可。

分片（先将文件切分成多个文件分片）上传是断点续传的基础。

`RandomAccessFile` 可以帮助我们合并文件分片：

![alt text](image.png)
