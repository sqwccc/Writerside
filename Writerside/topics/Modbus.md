# Modbus

Modbus 是一种在工业自动化系统中广泛使用的通信协议，用于在电子设备之间传输数据。它最初由 Modicon（现为施耐德电气的一部分）在1979年开发，旨在实现可编程逻辑控制器（PLC）之间的通信。由于其简单性和可靠性，Modbus 已成为工业通信的标准之一。

<note>
    <p>
        本文中讨论的是Modbus RTU（Remote Terminal Unit），不涉及Modbus ASCII、Modbus TCP/IP
    </p>
</note>

## 主要特点

* 开放标准： Modbus 是一个开放协议，任何厂商都可以免费实现，无需支付授权费用。
* 简单易用： 协议结构简单，易于理解和实现，适合各种规模的应用。 
* 多种传输方式： 支持多种物理层传输方式，包括串行通信（如 RS-232、RS-485）和以太网（Modbus TCP）。
* 广泛兼容： 兼容性强，支持多种设备和系统，方便不同厂商设备之间的互联互通。

## Modbus 的工作原理

Modbus 通信基于主从（Master-Slave）架构：

* 主设备（Master）：通常是一个控制器或计算机，负责发起请求并控制通信流程。
* 从设备（Slave）：如传感器、执行器或其他控制设备，响应主设备的请求。

## 报文结构
无论是主请求还是从应答的报文格式都为下图
![image_2.png](image_2.png)

上位机发：01030001001b5401

设备回复：01033600FA0000000000000000002F0001005300010000
00000000000000000000000000000000000000000000000000000000000000060000550C

### 字节和进制的概念

* 字节（Byte）：计算机存储数据的基本单位之一，通常由 8 位（bits）也就是8位二进制组成，每位可以是 0 或 1，
* 一个字节可以表示从 00000000（0）到 11111111（255）的数值范围。
* 八位二进制转换为十六进制即为两位十六进制数值

<note>
    因为一字节Byte代表了8位二进制数字，字节数组是文件读写、网络通信最常用的数据类型
</note>


### 地址域（Address Field）
- 长度：1 字节（8 位）
- 作用：标识目标从设备的地址。在主从（Master-Slave）架构中，主设备发送报文时需要指定目标从设备的地址，以便从设备识别并响应。
- 取值范围：1 至 247（0x01 至 0xF7）
0x00：广播地址（所有从设备接收，但不响应）
0xF8 至 0xFF：保留地址
- 示例：
地址为 1 的从设备：0x01
广播地址：0x00

### 功能码（Function Code）
- 长度：1 字节
- 作用：指示主设备请求的操作类型（如读取数据、写入数据等）。从设备根据功能码执行相应操作并返回结果。

<table>
<tr><td>功能码</td><td>描述</td></tr>
<tr><td>0x01</td><td>读取线圈状态（Read Coils）</td></tr>
<tr><td>0x02</td><td>读取离散输入状态（Read Discrete Inputs）</td></tr>
<tr><td>0x03</td><td>读取保持寄存器（Read Holding Registers）</td></tr>
<tr><td>0x04</td><td>读取输入寄存器（Read Input Registers）</td></tr>
<tr><td>0x05</td><td>写单个线圈（Write Single Coil）</td></tr>
<tr><td>0x06</td><td>写单个保持寄存器（Write Single Register）</td></tr>
<tr><td>0x0F</td><td>写多个线圈（Write Multiple Coils）</td></tr>
<tr><td>0x11</td><td>读设备识别（Read Device Identification）</td></tr>
</table>



- 示例：
读取保持寄存器：0x03
写单个线圈：0x05

### 数据域（Data Field）
- 长度：可变（N 字节）
- 作用：包含与功能码相关的具体数据，如读取或写入的起始地址、寄存器数量、写入的值等。数据域的结构根据具体的功能码而定。
   常见数据结构示例：

<tabs>
    <tab id="a" title="功能码 0x03">
      <p>读取保持寄存器请求</p>
       <p> 起始地址 (2 bytes) + 寄存器数量 (2 bytes)</p>
        示例:
         <list>
         <li>起始地址：0x000A（地址 10）
         </li>
         <li>寄存器数量：0x0002（读取 2 个寄存器）
         </li>
         </list>
        <p>报文数据域：00 0A 00 02</p>
    </tab>
    <tab id="b" title="功能码 0x06">
         <p>写单个保持寄存器请求</p>
       <p> 寄存器地址 (2 bytes) + 写入值 (2 bytes) </p>
        示例:
         <list>
         <li>寄存器地址：0x0005（地址 5）（地址 10）
         </li>
         <li>写入值：0x00FF（255）（读取 2 个寄存器）
         </li>
         </list>
        <p>报文数据域：00 05 00 FF</p>
    </tab>
    <tab id="c" title="功能码 0x10">
         <p>写多个保持寄存器请求</p>
       <p> 起始地址(2 bytes) + 寄存器数量N (2 bytes) + 字节计数2XN (1 byte) + 寄存器值（2xN bytes） </p>
        <p>回应：起始地址(2 bytes) + 寄存器数量N (2 bytes)</p>
    </tab>
</tabs>


### CRC 校验域（CRC Check Field）
- 长度：2 字节（16 位）
- 作用：用于校验报文的完整性，确保数据在传输过程中未被篡改或损坏。发送方计算整个报文的 CRC 校验码并附加在报文末尾；接收方同样计算接收到的报文的 CRC 并与附加的 CRC 进行比较，以验证数据的正确性。

```Java
    /**
     * 计算 Modbus CRC16 校验码（低位在前）
     * @param hexString 输入的十六进制字符串
     * @return 计算后的 CRC 校验码字符串（低位在前）
     */
    public static String calculateModbusCRC(String hexString) {
        // 将十六进制字符串转换为字节数组
        byte[] data = hexStringToByteArray(hexString);

        int crc = 0xFFFF; // 初始化 CRC 寄存器

        for (byte b : data) {
            crc ^= (b & 0xFF); // 将当前字节与 CRC 寄存器的低位异或
            for (int i = 0; i < 8; i++) {
                if ((crc & 0x0001) != 0) {
                    crc >>= 1;
                    crc ^= 0xA001; // 将多项式 0xA001 异或到 CRC 寄存器中
                } else {
                    crc >>= 1;
                }
            }
        }

        // 将 CRC 分为低位和高位
        int crcLow = crc & 0xFF; // CRC 低字节
        int crcHigh = (crc >> 8) & 0xFF; // CRC 高字节

        // 格式化为十六进制字符串并返回（低位在前）
        return String.format("%02X%02X", crcLow, crcHigh);
    }
```

## 示例解析

**上位机发：01 03 0001 001b 5401**

- 03功能码
- 0001代表启示地址
- 001b代表寄存器数量

**设备回复**：01 03 36 00FA0000000000000000002F0001005300010000
00000000000000000000000000000000000000000000000000000000000000060000 550C

- 01 03 对应上位机发送的消息
- 0x36代表字节数
- 00FA0000000000000000002F00010053000100000000000000000000000000000000000
0000000000000000000000000000000060000 108位载荷

<note>一个寄存器为四位十六进制，即为2Byte</note>

**要求设置地址1，寄存器地址为0x014b的单个寄存器数值为1，写出发送报文：**

数据域格式为：2Byte寄存器地址+2Byte设置的值

01 06 014b 0001 四位crc

**解析：0310012c00356a0000fffefffefffefffefffefffefffefffefffefffefffefffe
fffefffefffefffefffefffefffefffefffefffefffefffefffe
fffefffefffefffefffe
0001fffefffefffefffefffefffefffefffefffefffefffefffe0000fffefffefffefffefffefffefffefffed77e**

<note>0xfffe为特殊定义，表示当前寄存器数值不变</note>

<p>03</p>
<p>10</p>
<p>012c</p>
<p>0035</p>
<p>6a</p>




