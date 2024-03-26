---
weight: 1
title: "PicoCTF 2024"
date: 2024-03-16T14:30:00+07:00
lastmod: 2024-03-16T14:30:00+07:00
draft: false
author: "ClownCS"
authorLink: "https://clowncs.github.io"
description: "Solutions for all reverse challenges in PicoCTF"
tags: ["RE", "Pico", "2024", "Weekly"]
categories: ["Writeups"]

lightgallery: true

toc:
  enable: true
---
Solutions for all reverse challenges in PicoCTF

<!--more-->

# RE

Giải này mình khá may mắn vì có thể hoàn thành nó trong 2 tiếng (~~nếu author không ra đề lỗi thì có thể sớm hơn~~)

![image](https://hackmd.io/_uploads/H1YcDT6CT.png)

## packer

Như tên đề bài thì mình nghĩ file đã được packed nên mình dùng ``Detect it easy`` để check thử

![image](https://hackmd.io/_uploads/S1anFppCp.png)

Như bạn thấy thì nó được packed bởi UPX 3.95. Để unpack mình dùng [UPX](https://github.com/upx/upx/releases/tag/v3.96) và lệnh ``.\upx.exe -d out``

![image](https://hackmd.io/_uploads/SJC25pTCT.png)

Mở file bằng ida ta thấy được flag là một chuỗi hex ``7069636f4354467b5539585f556e5034636b314e365f42316e34526933535f65313930633366337d`` decode ta được:

Flag: ***picoCTF{U9X_UnP4ck1N6_B1n4Ri3S_e190c3f3}***

## FactCheck
Bài này là bài author ra đề sai khiến mình bị chậm trễ clear =))). Đây là một bài rev C++ cũng khá dễ đọc nó chỉ thực hiện vài phép so sánh.

```c++
if ( *(char *)std::string::operator[](v24, 0LL) <= 65 )
std::string::operator+=(v22, v34);
if ( *(_BYTE *)std::string::operator[](v35, 0LL) != 65 )
std::string::operator+=(v22, v37);
if ( "Hello" == "World" )
std::string::operator+=(v22, v25);
v19 = *(char *)std::string::operator[](v26, 0LL);
if ( v19 - *(char *)std::string::operator[](v30, 0LL) == 3 )
std::string::operator+=(v22, v26);
std::string::operator+=(v22, v25);
std::string::operator+=(v22, v28);
if ( *(_BYTE *)std::string::operator[](v29, 0LL) == 71 )
std::string::operator+=(v22, v29);
```

Mình đặt breakpoint ở cuối chương trình và lấy flag ra

![image](https://hackmd.io/_uploads/HkaJ2ppAa.png)

Flag: ***picoCTF{wELF_d0N3_mate_97750d5f}***

## WinAntiDbg0x100

Đây là một bài antidebug nên việc tìm ra các chỗ check debug là điều quan trọng. Đọc sơ chương trình mà xem phần ``Imports``, mình nhận ra chỉ có một hàm check duy nhất là ``IsDebuggerPresent``
```C
 if ( IsDebuggerPresent() )
      {
        OutputDebugStringW(L"### Oops! The debugger was detected. Try to bypass this check to get the flag!\n");
      }
```
Để bypass đoạn này thì chỉ cần debug và setip tới nhánh đúng.

![image](https://hackmd.io/_uploads/BktzTppRp.png)

Flag: ***picoCTF{d3bug_f0r_th3_Win_0x100_e6c390e2}***


## Classic Crackme 0x100
Đây là một bài checker cơ bản. Nhận input của người dùng thực hiện biến đổi và compare với chuỗi ``xjagpediegzqlnaudqfwyncpvkqneusycourkguerjpzcbstcc``


```c
strcpy(output, "xjagpediegzqlnaudqfwyncpvkqneusycourkguerjpzcbstcc");
setvbuf(_bss_start, 0LL, 2, 0LL);
printf("Enter the secret password: ");
__isoc99_scanf("%50s", input);
i = 0;
len = strlen(output);
secret1 = 85;
secret2 = 51;
secret3 = 15;
fix = 97;
while ( i <= 2 )
{
for ( i_0 = 0; i_0 < len; ++i_0 )
{
  random1 = (secret1 & (i_0 % 255)) + (secret1 & ((i_0 % 255) >> 1));
  random2 = (random1 & secret2) + (secret2 & (random1 >> 2));
  input[i_0] = ((random2 & secret3) + input[i_0] - fix + (secret3 & (random2 >> 4))) % 26 + fix;
}
++i;
}
if ( !memcmp(input, output, len) )
printf("SUCCESS! Here is your flag: %s\n", "picoCTF{sample_flag}");
else
puts("FAILED!");
```

Đây là script bruteforce của mình:

```c
#include <stdio.h>
#include <string.h>

char key[] ="xjagpediegzqlnaudqfwyncpvkqneusycourkguerjpzcbstcc";
int secret1 = 85;
int secret2 = 51;
int secret3 = 15;
int fix = 97;
int f(int i , char c){
    int i_0;
    char x = c;
    int random1, random2;
    for ( i_0 = 0; i_0 < 3; ++i_0 )
    {
      random1 = (secret1 & (i % 255)) + (secret1 & ((i % 255) >> 1));
      random2 = (random1 & secret2) + (secret2 & (random1 >> 2));
      x = ((random2 & secret3) + x - fix + (secret3 & (random2 >> 4))) % 26 + fix;
    }
  return x == key[i];
}

int main(){
    for (int i = 0 ; i < strlen(key); i++){
        for ( char brute = '!'; brute < '}'; brute++){
        if( f(i,brute)){
            printf("%c",brute);
            break;
        }
      }
    }
}
```

Ta có key là ``xg*am+*`bathfe$iak`nse&dpbhb(igj,ioie*l%lagn&"ge)&`` nc tới instance ta có kết quả

Flag: ***picoCTF{s0lv3_angry_symb0ls_31b29976}***

## weirdSnake
Bài cho mình một đống bytecode python dựa vào document [dis](https://docs.python.org/3/library/dis.html) ta có thể reimplement và biết chính xác nó làm gì. Tuy nhiên ở đây mình làm có phần hơi guess vì mình thấy nó khá ngắn.
Mình thấy chương trình gồm có hai thứ đáng quan tâm thứ nhất là mảng mình sẽ gọi là ``input_list``

```python
input_list = [4, 54, 41, 0, 112, 32, 25, 49, 33, 3, 0, 0, 57, 32, 108, 23, 48, 4, 9, 70, 7, 110, 36, 8, 108, 7, 49, 10, 4, 86, 43, 59, 124, 86, 0, 69, 59, 47, 93, 78]
```

Và mảng thứ hai là một chuỗi ``t_Jo3``, có thể thấy độ dài mảng chia hết cho độ dài chuỗi nên mình mạnh dạn xor luôn =))). 

```python
input_list = [4, 54, 41, 0, 112, 32, 25, 49, 33, 3, 0, 0, 57, 32, 108, 23, 48, 4, 9, 70, 7, 110, 36, 8, 108, 7, 49, 10, 4, 86, 43, 59, 124, 86, 0, 69, 59, 47, 93, 78]


key_str = 't_Jo3'

for i in range(len(input_list)):
    print(chr(input_list[i] ^ ord(key_str[i%5])),end="")
```
Flag: ***picoCTF{N0t_sO_coNfus1ng_sn@ke_d6931de2}***

## WinAntiDbg0x200
Bài này trước khi load vào ida thì bạn nên chạy ida dưới quyền admin ``This executable requires admin privileges``. Lần này hàm check debug có thêm một hàm ``pidcheck`` tuy nhiên vẫn như cách cũ mình setip tới nhánh tiếp theo và lấy flag
```C
if ( pidcheck() || IsDebuggerPresent() )
  {
    OutputDebugStringW(L"### Oops! The debugger was detected. Try to bypass this check to get the flag!\n");
  }
```

![image](https://hackmd.io/_uploads/rky9fCpRT.png)

Flag: ***picoCTF{0x200_debug_f0r_Win_603b1bdf}***

## WinAntiDbg0x300

Bài cuối này sau khi thử bằng ``Detect it easy`` thì mình phát hiện nó được packed bằng ``UPX 4.21``

![image](https://hackmd.io/_uploads/HJWSEAaC6.png)

Sau khi unpacked nó load vào ida và đọc thử. Sau khi đọc sơ thì mình thấy chương trình này có hai đoạn check antidebug.


```C
if ( (unsigned __int8)debugger_check() )
{
    MessageBoxW(hWnd, L"Oops! Debugger Detected. Challenge Aborted.", &Caption, 0x40u);
    sub_4011E0(255);
}

...

while ( CreateProcessA(0, CommandLine, 0, 0, 0, 0, 0, 0, &StartupInfo, &ProcessInformation) )
{
WaitForSingleObject(ProcessInformation.hProcess, 0xFFFFFFFF);
GetExitCodeProcess(ProcessInformation.hProcess, &ExitCode);
switch ( ExitCode )
{
  case 0xFFu:
    MessageBoxW(hWnd, L"Something went wrong. Challenge aborted.", &Caption, 0x10u);
    sub_4011E0(255);
  case 0xFEu:
    MessageBoxW(
      hWnd,
      L"The debugger was detected but our process wasn't able to fight it. Challenge aborted.",
      &Caption,
      0x10u);
    sub_4011E0(255);
  case 0xFDu:
    MessageBoxW(
      hWnd,
      L"Our process detected the debugger and was able to fight it. Don't be surprised if the debugger crashed.",
      &Caption,
      0x10u);
    break;
}
CloseHandle(ProcessInformation.hProcess);
CloseHandle(ProcessInformation.hThread);
Sleep(0x1388u);
```

Cách làm vẫn như cũ mình sẽ debug và setip qua nhánh đúng riêng hàm ``while`` mình sẽ setip để không nhảy vào hàm while đó.

![Untitled](https://hackmd.io/_uploads/Bkw2tATC6.png)

Sau đó flag sẽ nhả ra

![asd](https://hackmd.io/_uploads/r13W9RTCp.png)

Flag: ***picoCTF{Wind0ws_antid3bg_0x300_daad7155}***

> Giải này mình còn làm được vài bài pwn tuy nhiên thì do mình không thực sự hiểu rõ bản chất nên mình không muốn trình bày wu ở đây. Cảm ơn mọi người đã dành thời gian đọc blog <3