Varint 编码是一种使用一个或多个字节序列化整数的方法，会把整数编码成变长字节，32 位整型数据经过 Varint 编码后占用 1～5 个字节，数值越小，占用的字节越少

# Varint 编码原理

varint 编码中使用每个字节的地 7 位存储数据本身，最高 1 位为 msb 位（most significant bit）表示后面的字节是否与该字节表示同一个整数，为 1 表示后面的字节属于当前数据，为 0 表示是当前数据的最后一个字节
varint 编码采用小端字节序，即最小的字节放在最前面

以 300 为例，展示使用 varint 编码和解码：

```
300 = 256+32+8+4
=  0000 0001 0010 1100 (二进制)
->  010 1100  000 0010 (小端存储)
-> 1010 1100 0000 0010 (添加 msb 位)

->  010 1100  000 0010 (去掉 msb 位)
->  000 0010  010 1100 (翻转)
-> 0000 0001 0010 1100 (补齐)
= 300
```

但当使用 varint 编码存储负数时，32 位负数始终占 5 个字节，64 位负数始终占 10 个字节
为了使编码更高效，Varint 采用 ZigZag 编码方式

# ZigZag 编码

ZigZag 首先通过公式将带符号整数转化为无符号整数，再通过 varint 进行压缩编码，这样使绝对值较小的负数仍有较小的 varint 编码值

|原值|ZigZag 编码后的值|
|--|--|
|0|0|
|-1|1|
|1|2|
|-2|3|
|2147483647|4294967294|
|-2147483648|4294967295|

对应的公式为：
`(n << 1) ^ (n >> 31)`

以 -1 为例，进行 ZigZag 编码
```
-1        = 1111 1111 1111 1111 1111 1111 1111 1111
(n << 1)  = 1111 1111 1111 1111 1111 1111 1111 1110
(n >> 31) = 1111 1111 1111 1111 1111 1111 1111 1111
(n << 1) ^ (n >> 31) = 1

1         = 0000 0000 0000 0000 0000 0000 0000 0001
(n << 1)  = 0000 0000 0000 0000 0000 0000 0000 0010
(n >> 31) = 0000 0000 0000 0000 0000 0000 0000 0000
(n << 1) ^ (n >> 31) = 2
```


#  Java 代码实现

```java
public class Varint {  
  
    public static int sizeOfInt(int v) {  
        int bytes = 1;  
        while ((v & 0xFFFFFF80) != 0) {  
            bytes += 1;  
            v >>>= 7;  
        }  
        return bytes;  
    }  
  
    public static void intToVarint(int v, ByteBuffer buffer) {  
        v = intToZigZag(v);  
        while ((v & 0xFFFFFF80) != 0) {  
            byte b = (byte) ((v & 0x7F) | 0x80);  
            buffer.put(b);  
            v >>>= 7;  
        }  
        buffer.put((byte) v);  
    }  
  
    public static int varintToInt(ByteBuffer buffer) {  
        int value = 0;  
        int i = 0;  
        byte b;  
        while (((b = buffer.get()) & 0x80) != 0) {  
            value |= (b & 0x7F) << i;  
            i += 7;  
        }  
        value |= b << i;  
        return intToZigZag(value);  
    }  
  
    private static int intToZigZag(int v) {  
        return (v << 1) ^ (v >> 31);  
    }  
  
    private static int zigZagToInt(int v) {  
        return (v >>> 1) ^ -(v & 1);  
    }  
  
}
```