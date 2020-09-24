

![IO](/Users/popmain/Desktop/record/note/image/IO.jpg)



|               |                                      |                                    |                                  |                            |
| ------------- | ------------------------------------ | ---------------------------------- | -------------------------------- | -------------------------- |
| 数据类型      | 基于字节的 Input                     | 基于字节的 Output                  | 基于字符的 Input                 | 基于字符的 Output          |
| 基础          | InputStream                          | OutputStream                       | Reader 、 InputStreamReader      | Writer、OutputStreamWriter |
| 数组          | ByteArrayInputStream                 | ByteArrayOutputStream              | CharArrayReader                  | CharArrayWriter            |
| Files         | FileInputStream、RandomAccessFile    | FileOutputStream、RandomAccessFile | FileReader                       | FileWriter                 |
| 管道          | PipedInputStream                     | PipedOutputStream                  | PipedReader                      | PipedWriter                |
| 缓冲          | BufferedInputStream                  | BufferedOutputStream               | BufferedReader                   | BufferedWriter             |
| 过滤          | FilterInputStream                    | FilterOutputStream                 | FilterReader                     | FilterWriter               |
| 解析          | PushbackInputStream、StreamTokenizer |                                    | PushbackReader、LineNumberReader |                            |
| 字符串        |                                      |                                    | StringReader                     | StringWriter               |
| 数据          | DataInputStream                      | DataOutputStream                   |                                  |                            |
| 数据 - 格式化 |                                      | PrintStream                        |                                  | PrintWriter                |
| 对象          | ObjectInputStream                    | ObjectOutputStream                 |                                  |                            |
| 组合多个流    | SequenceInputStream                  |                                    |                                  |                            |

# 选择



### 1. 按照用途进行分类

1.1 按照数据的来源（去向）分类
 •是文件：FileInputStream, FileOutputStream, FileReader, FileWriter
 •是byte[]：ByteArrayInputStream, ByteArrayOutputStream
 •是char[]：CharArrayReader, CharArrayWriter
 •是String：StringBufferInputStream(已过时，因为其只能用于String的每个字符都是8位的字符串), StringReader, StringWriter
 •是网络数据流：InputStream, OutputStream, Reader, Writer

1.2 按照格式化输出
 •需要格式化输出：PrintStream（输出字节），PrintWriter（输出字符）

1.3 按缓冲功能分
 •要缓冲：BufferedInputStream, BufferedOuputStream, BuffereaReader, BufferedWriter

1.4 按照数据格式分
 •二进制格式（只要确定不是纯文本格式的），InputStream, OutputStream, 及其所有带Stream子类
 •纯文本格式（比如英文/汉字/或其他编码文字）：Reader, Writer, 及其相关子类

1.5 按照输入输出分
 •输入：Reader， InputStream，及其相关子类
 •输出：Writer，OutputStream，及其相关子类

1.6 特殊需求
 •从Stream转化为Reader，Writer：InputStreamReader，OutputStreamWriter
 •对象输入输出流：ObjectInputStream，ObjectOutputStream
 •进程间通信：PipeInputStream，PipeOutputStream，PipeReader，PipeWriter
 •合并输入：SequenceInputStream
 •更特殊的需要：PushbackInputStream, PushbackReader, LineNumberInputStream, LineNumberReader

### 2 确定选用流对象的步骤

•确定原始数据的格式
 •确定是输入还是输出
 •是否需要转换流
 •数据的来源（去向）
 •是否需要缓冲
 •是否需要格式化输出

### 3 Java中Inputstream/OutputStream与Reader/Writer的区别

•Reader/Writer和InputStream/OutputStream分别是I/O库提供的两套平行独立的等级机构， •InputStream、OutputStream是用来处理8位元(**字节**)的流，也就是用于读写ASCII字符和二进制数据；Reader、Writer是用来处理16位元的流，也就是用于读写Unicode编码的字符。
 •在JAVA语言中，byte类型是8位的，char类型是16位的，所以在处理中文的时候需要用Reader和Writer。
 •两种等级机构下，有一道桥梁InputStreamReader、OutputStreamWriter负责进行InputStream到Reader的适配和由OutputStream到Writer的适配。

•在Java中，有不同类型的Reader/InputStream输入流对应于不同的数据源： •FileReader/FileInputStream 用于从文件输入；
 •CharArrayReader/ByteArrayInputStream 用于从程序中的字符数组输入；
 •StringReader/StringBufferInputStream 用于从程序中的字符串输入；
 •PipedReader/PipeInputStream 用于读取从另一个线程中的数据PipedWriter/PipeOutputStream 写入管道的数据。
 •相应的也有不同类型的Writer/OutputStream输出流对应于不同的数据源：FileWriter/FileOutputStream，CharArrayWriter/ByteArrayOutputStream，StringWriter，PipeWriter/PipedOutputStream。

•有两种没有对应Reader类型的InputStream输入流，用getInputStream()来读取数据。 •Socket 用于套接字；
 •URLConnection 用于 URL 连接。

### 4  流类的继承关系图

4.1 .继承自InputStream/OutputStream的流都是用于向程序中输入/输出数据，且数据的单位都是字节(byte=8bit)，如图，深色的为节点流，浅色的为处理流。

4.2 .继承自Reader/Writer的流都是用于向程序中输入/输出数据，且数据的单位都是字符(2byte=16bit)，如图，深色的为节点流，浅色的为处理流。

4.3 节点流类型
 •对文件操作的字符流有FileReader/FileWriter，
 •字节流有FileInputStream/FileOutputStream。

4.4 处理流类型
 •缓冲流：缓冲流要“套接”在相应的节点流之上，对读写的数据提供了缓冲的功能，提高了读写效率，同事增加了一些新的方法。 •字节缓冲流有BufferedInputStream/BufferedOutputStream，字符缓冲流有BufferedReader/BufferedWriter，字符缓冲流分别提供了读取和写入一行的方法ReadLine和NewLine方法。
 •对于输出地缓冲流，写出的数据，会先写入到内存中，再使用flush方法将内存中的数据刷到硬盘。所以，在使用字符缓冲流的时候，一定要先flush，然后再close，避免数据丢失。

•转换流：用于字节数据到字符数据之间的转换。 •字符流InputStreamReader/OutputStreamWriter。其中，InputStreamReader需要与InputStream“套接”，OutputStreamWriter需要与OutputStream“套接”。

•数据流：提供了读写Java中的基本数据类型的功能。 •DataInputStream和DataOutputStream分别继承自InputStream和OutputStream，需要“套接”在InputStream和OutputStream类型的节点流之上。

•对象流：用于直接将对象写入写出。 •流类有ObjectInputStream和ObjectOutputStream，本身这两个方法没什么，但是其要写出的对象有要求，该对象必须实现Serializable接口，来声明其是可以序列化的。否则，不能用对象流读写。



Practice

```java
package com.company;

import java.io.*;
import java.nio.Buffer;
import java.nio.ByteBuffer;
import java.nio.CharBuffer;
import java.nio.MappedByteBuffer;
import java.nio.channels.FileChannel;
import java.nio.charset.Charset;
import java.nio.charset.CharsetDecoder;

public class Main {
    public final static String FILE = "/Users/popmain/workspace/Java/src/com/company/data.txt";
    public final static String FILE_TEMP = "/Users/popmain/workspace/Java/src/com/company/temp.txt";
    public final static String IMAGE_FILE = "/Users/popmain/workspace/Java/src/com/company/image.jpg";
    public final static String IMAGE_COPY_FILE = "/Users/popmain/workspace/Java/src/com/company/image_copy.jpg";
    public static void main(String[] args) throws IOException {
        /**
         * 控制台输入
         */
        BufferedReader inputBufferReader = new BufferedReader(new InputStreamReader(System.in));
        char inputChar;
        do {
            inputChar = (char) inputBufferReader.read();
            if (inputChar != 10) {
                 System.out.println("Your input is " + inputChar);
            }
        } while (inputChar != 'q');


        /**
         * FileOutputStream
         */
        byte[] bytes = "FileOutputStream".getBytes();
        FileOutputStream fileOutputStream = new FileOutputStream(FILE);
        fileOutputStream.write(bytes);
        fileOutputStream.flush();
        fileOutputStream.close();


        /**
         * FileInputStream
         */
        FileInputStream fileInputStream = new FileInputStream(FILE);
        int c;
        while ((c = fileInputStream.read()) != -1) {
            System.out.print(c);
        }
        fileInputStream.close();

        /**
         * File Copy
         */
        FileInputStream imageInput = new FileInputStream(IMAGE_FILE);
        BufferedInputStream imageInputBuffer = new BufferedInputStream(imageInput);
        File copyFile = new File(IMAGE_COPY_FILE);
        if (!copyFile.exists()) {
            copyFile.createNewFile();
        }
        FileOutputStream copyImageOutStream = new FileOutputStream(copyFile);
        BufferedOutputStream copyOutputBuffer = new BufferedOutputStream(copyImageOutStream);
        int len = 0;
        byte[] bs = new byte[1024];
        long start0 = System.currentTimeMillis();
        while ((len = imageInputBuffer.read(bs)) != -1) {
            copyOutputBuffer.write(bs, 0, len);
        }
        System.out.println("带缓冲， copy cost :  " + (System.currentTimeMillis() - start0));
        imageInputBuffer.close();
        imageInput.close();
        copyOutputBuffer.close();
        copyImageOutStream.close();

        FileInputStream imageInput1 = new FileInputStream(IMAGE_FILE);
        FileOutputStream imageOutput2 = new FileOutputStream(IMAGE_COPY_FILE);
        start0 = System.currentTimeMillis();
        while ((len = imageInput1.read(bs)) != -1) {
            imageOutput2.write(bs, 0, len);
        }
        System.out.println("没缓冲， copy cost :  " + (System.currentTimeMillis() - start0));
        imageInput1.close();
        imageOutput2.close();

        /** Reader与Writer提供兼容Unicode与面向字符的I/O功能 **/
        /**
         * FileWriter
         */
        FileWriter fileWriter = new FileWriter(FILE);
        BufferedWriter bufferedWriter = new BufferedWriter(fileWriter);
        bufferedWriter.write("dlfoiadofiaoidsfoidofijoj\nidiof\n3994\n9ojflk");
        bufferedWriter.flush();
        bufferedWriter.close();

        /**
         * FileReader
         */
        FileReader fileReader = new FileReader(FILE);
        BufferedReader bufferedReader = new BufferedReader(fileReader);
        String line;
        while ((line = bufferedReader.readLine()) != null) {
            System.out.println(line);
        }

        /**
         * Process
         */
        Process process = new ProcessBuilder("java -version".split(" ")).start();
        BufferedReader terminalReader = new BufferedReader(new InputStreamReader(process.getInputStream()));
        String s;
        while ((s = terminalReader.readLine()) != null) {
            System.out.println(s);
        }

        BufferedReader terminalErrorReader = new BufferedReader(new InputStreamReader(process.getErrorStream()));
        while ((s = terminalErrorReader.readLine()) != null) {
            System.out.println(s);
        }


        /**
         * NIO
         */
//        FileChannel channel = new FileOutputStream(FILE).getChannel();
//        channel.write(ByteBuffer.wrap("你好还好都i哦放假哦时代佛阿瑟东的积分骄傲的佛i阿三".getBytes("UTF-16")));
//        channel.close();
//        RandomAccessFile randomAccessFile =  new RandomAccessFile("C:\\workspace\\TestProject\\src\\com\\company\\data.txt", "rw");
//        randomAccessFile.seek(randomAccessFile.length());
//        randomAccessFile.writeChars("\nAdd line");
//        channel = new RandomAccessFile("C:\\workspace\\TestProject\\src\\com\\company\\data.txt", "rw").getChannel();
//        channel.position(channel.size());
//        channel.write(ByteBuffer.wrap(("\nNew line".getBytes())));
//        channel.close();
//        Charset charset = Charset.forName("UTF-16BE");//Java.nio.charset.Charset处理了字符转换问题。它通过构造CharsetEncoder和CharsetDecoder将字符序列转换成字节和逆转换。
//
//        FileChannel channel1 = new FileInputStream(FILE).getChannel();
//        ByteBuffer buffer = ByteBuffer.allocate(1024);
//        while (true) {
//            buffer.clear(); // 写之前先clear
//            int count = channel1.read(buffer);
//            if (count < 0) {
//                break;
//            }
//            buffer.flip(); // 读之前先flip
//            System.out.println(charset.decode(buffer));
//            buffer.compact();
//        }
//        channel1.close();
//
//        FileChannel in = new FileInputStream(FILE).getChannel();
//        FileChannel out = new FileOutputStream(FILE_TEMP).getChannel();
//        ByteBuffer buffer1 = ByteBuffer.allocate(1024);
//        while (in.read(buffer1) != -1) {
//            buffer1.flip();
//            out.write(buffer1);
//            buffer1.clear();
//        }
//        in.close();
//        out.close();

//        char[] data = "UsingBuffers".toCharArray();
//        ByteBuffer buffer = ByteBuffer.allocate(data.length*2);
//        CharBuffer charBuffer = buffer.asCharBuffer();
//        charBuffer.put(data);
//        System.out.println(charBuffer.rewind());
//        sysmmetrisScrameble(charBuffer);
//        System.out.println(charBuffer.rewind());
//        sysmmetrisScrameble(charBuffer);
//        System.out.println(charBuffer.rewind());
//        System.out.println(charBuffer);
//        mapFile();
    }

    private static void sysmmetrisScrameble(CharBuffer charBuffer) {
        while (charBuffer.hasRemaining()) {
            charBuffer.mark();
            char ch1 = charBuffer.get();
            char ch2 = charBuffer.get();
            charBuffer.reset();
            charBuffer.put(ch2).put(ch1);
        }
    }


    private static void mapFile() throws IOException {
        int length = 0x8ffffff;
        MappedByteBuffer out = new RandomAccessFile(FILE, "rw").getChannel().map(FileChannel.MapMode.READ_WRITE, 0, length);
        for (int i = 0; i < length; i ++) {
            out.put((byte)'x');
        }
        System.out.println("Finished writing");
        for (int i = length / 2; i < length / 2 + 6; i++) {
            System.out.println((char) out.get(i));
        }


    }
}


```

