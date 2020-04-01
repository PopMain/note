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

public class Mian5 {
    public final static String FILE = "C:\\workspace\\TestProject\\src\\com\\company\\data.txt";
    public final static String FILE_TEMP = "C:\\workspace\\TestProject\\src\\com\\company\\temp.txt";
    public static void main(String[] args) throws IOException {
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
//        /**
//         * FileWriter
//         */
//        FileWriter fileWriter = new FileWriter(FILE);
//        BufferedWriter bufferedWriter = new BufferedWriter(fileWriter);
//        bufferedWriter.write("dlfoiadofiaoidsfoidofijoj\nidiof\n3994\n9ojflk");
//        bufferedWriter.flush();
//        bufferedWriter.close();
//        /**
//         * FileReader
//         */
//        FileReader fileReader = new FileReader(FILE);
//        BufferedReader bufferedReader = new BufferedReader(fileReader);
//        String line;
//        while ((line = bufferedReader.readLine()) != null) {
//            System.out.println(line);
//        }



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

