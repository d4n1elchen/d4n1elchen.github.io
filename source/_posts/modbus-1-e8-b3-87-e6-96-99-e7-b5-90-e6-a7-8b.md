---
title: Modbus (1) 資料結構
tags:
  - IoT
  - modbus
  - Machine Tools
  - python
id: 483
categories:
  - IoT
date: 2016-09-16 20:12:28
---

因為專題接觸到這東西，他是一種串行通訊協定，在工業電子上應用廣泛

Modbus允許多個 (大約240個) 裝置連線在同一個網路上進行通訊

其他詳細介紹參見[維基百科](https://zh.wikipedia.org/wiki/Modbus)

<!--more-->

這篇記錄Modbus的通訊規格

大部分內容來自這篇[官方文件](http://www.modbus.org/docs/Modbus_over_serial_line_V1_02.pdf)

# Overview

Modbus的指令由Function code和Data組成PDU(Protocol Data Unit)，再加上Address field和Error check成為Modbus serial line PDU，結構如下
```
|========================Serial line=========================|
|--Address field--|--Function code--|--Data--|--Error check--|
                  |============PDU===========|
```

## PDU

Function code: 1byte的指令碼，有效值為1~255，128~255為保留區，為錯誤回應碼

Data: 指令所攜帶的資料內容

Modbus共定義三種PDU結構，分別為

*   MODBUS Request PDU, mb_req_pdu
```
mb_req_pdu = {function_code, request_data}
function_code = [1 byte] MODBUS function code,
request_data = [n bytes] 依據function code給予所需的data
```

*   MODBUS Response PDU, mb_rsp_pdu
```
mb_rsp_pdu = {function_code, response_data}
function_code = [1 byte] MODBUS function code
response_data = [n bytes] 依據function code返回相應的資料
```

*   MODBUS Exception Response PDU, mb_excep_rsp_pdu
```
mb_excep_rsp_pdu = {exception-function_code, request_data}
exception-function_code = [1 byte] MODBUS function code + 0x80  exception_code = [1 byte] MODBUS Exception Code
```
Modbus為big-endian系統，意即當資料長度大於傳輸的最大長度時，從MSB(Most Significant Bit)開始傳，即若有一預傳送之資料`0x12345678`則傳送順序為`0x12 0x34 0x56 0x78`(若為little-endian的話會變成`0x78 0x56 0x34 0x12`)

## Serial Line

Address field: 長度1byte的數值，定義此段指令的傳送對象
- 0               Broadcast address
- 1~247      Slave individual addresses
- 248~255 Reserved

Error check: 使用Redundancy Checking(冗餘校驗)，依據使用的傳輸規格(RTU、ASCII)不同分別使用CRC([循環冗餘校驗](https://zh.wikipedia.org/wiki/%E5%BE%AA%E7%92%B0%E5%86%97%E9%A4%98%E6%A0%A1%E9%A9%97))和LRC([縱向冗餘校驗](https://zh.wikipedia.org/wiki/%E7%BA%B5%E5%90%91%E5%86%97%E4%BD%99%E6%A0%A1%E9%AA%8C))

# Function Code

Function Code可分為由Modbus官方認證指定的Public function code跟硬體廠商自行定義的User define function code

下表為Public function code定義表

![擷取.PNG](https://team6612.files.wordpress.com/2016/09/e693b7e58f964.png)

紅框為此次專題用到的指令，讀取暫存器資料用的，其他的也不知道做什麼用就不介紹了

## 0x03 Read Holding Register

此指令的PDU結構如下

### Request

Function code(1byte): 0x03
Starting Address(2byte): 0x0000 ~ 0xFFFF
Quantity of Register(2byte): 1 ~ 125 (N)

指定起始地址跟要讀取的暫存器數量

### Response

Function code(1byte): 0x03
Byte count(1byte): 2*N
Register value(2byte*N): data value
(N=Quantity of Register)

回應從指定的開始位址開始後N個位址的值

### Error

Error code(1byte): 0x03 + 0x80 = 0x83
Exception code(1byte): 01/02/03/04

回應錯誤碼
01 ILLEGAL FUNCTION
02 ILLEGAL DATA ADDRESS
03 ILLEGAL DATA VALUE
04 SERVER DEVICE FAILURE

下一篇將介紹Modbus的兩種傳輸模式，RTU跟ASCII
