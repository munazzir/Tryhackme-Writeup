Reversing ELF
=============


In this binary reversing will be using these tools,

+ strings

+ ltrace 

+ r2 


## crackme1

Nothing much to do it give flag wheb executed

![crackme1](/Images/reversing-elf/crackme1.png)



## crackme2

when executed it ask for password

````
./crackme2
Usage: ./crackme2 password
````

![crackme2](/Images/reversing-elf/crackme2.png)



## crackme3

this is also similer to earlier one so we can find it with `strings` command but it encoded with `Base64`

![crackme3](/Images/reversing-elf/crackme3.png)

then decode base64 and enter the answer.

```
echo "ZjByX3kwdXJfNWVjMG5kX2xlNTVvbl91bmJhc2U2NF80bGxfN2gzXzdoMW5nNQ==" | base64 -d

f0r_y0ur_5ec0nd_le55on_unbase64_4ll_7h3_7h1ng5
```


## crackme4

when executed we can see this message.

```
└─# ./crackme4          
Usage : ./crackme4 password
This time the string is hidden and we used strcmp
```

looks like we can't find the answer with `strings` command but we can try `ltrace`.

![crackme4](/Images/reversing-elf/crackme4.png)


## crackme5

This is similer to earlier one, we can use `ltrace` to find the correct **input**

![crackme5](/Images/reversing-elf/crackme5.png)


## crackme6

This is different from others we can't get answer from `strings` or `ltrace` so now need to use `r2`

to start,

```
r2 -d ./crackme6

> aaa

> afl
```

![crackme6-1](/Images/reversing-elf/crackme6-1.png)

there is `main` function we can find the answer in there.

```
│       │   0x00400735      b800000000     mov eax, 0
│       │   0x0040073a      e821fdffff     call sym.imp.printf         ; int printf(const char *format)
│      ┌──< 0x0040073f      eb13           jmp 0x400754
│      │└─> 0x00400741      488b45f0       mov rax, qword [var_10h]
│      │    0x00400745      4883c008       add rax, 8
│      │    0x00400749      488b00         mov rax, qword [rax]
│      │    0x0040074c      4889c7         mov rdi, rax
│      │    0x0040074f      e87dffffff     call sym.compare_pwd
│      │    ; CODE XREF from main @ 0x40073f
│      └──> 0x00400754      b800000000     mov eax, 0
│           0x00400759      c9             leave

```



look at `sys.compare_pwd` it comparing password if we look furthr we can find answer.

`pdf @sym.compare_pwd` 


![crackme6-2](/Images/reversing-elf/crackme6-2.png)

In there `sys.my_secure_test` look interesting.

`pdf @sym.my_secure_test`


```
[0x7f9f00410090]> pdf @sym.my_secure_test
            ; CALL XREF from sym.compare_pwd @ 0x4006e4
┌ 340: sym.my_secure_test (int64_t arg1);
│           ; var int64_t var_8h @ rbp-0x8
│           ; arg int64_t arg1 @ rdi
│           0x0040057d      55             push rbp
│           0x0040057e      4889e5         mov rbp, rsp
│           0x00400581      48897df8       mov qword [var_8h], rdi     ; arg1
│           0x00400585      488b45f8       mov rax, qword [var_8h]
│           0x00400589      0fb600         movzx eax, byte [rax]
│           0x0040058c      84c0           test al, al
│       ┌─< 0x0040058e      740b           je 0x40059b
│       │   0x00400590      488b45f8       mov rax, qword [var_8h]
│       │   0x00400594      0fb600         movzx eax, byte [rax]
│       │   0x00400597      3c31           cmp al, 0x31                ; 49
│      ┌──< 0x00400599      740a           je 0x4005a5
│      │└─> 0x0040059b      b8ffffffff     mov eax, 0xffffffff         ; -1
│      │┌─< 0x004005a0      e92a010000     jmp 0x4006cf
│      └──> 0x004005a5      488b45f8       mov rax, qword [var_8h]
│       │   0x004005a9      4883c001       add rax, 1
│       │   0x004005ad      0fb600         movzx eax, byte [rax]
│       │   0x004005b0      84c0           test al, al
│      ┌──< 0x004005b2      740f           je 0x4005c3
│      ││   0x004005b4      488b45f8       mov rax, qword [var_8h]
│      ││   0x004005b8      4883c001       add rax, 1
│      ││   0x004005bc      0fb600         movzx eax, byte [rax]
│      ││   0x004005bf      3c33           cmp al, 0x33                ; 51
│     ┌───< 0x004005c1      740a           je 0x4005cd
│     │└──> 0x004005c3      b8ffffffff     mov eax, 0xffffffff         ; -1
│     │┌──< 0x004005c8      e902010000     jmp 0x4006cf
│     └───> 0x004005cd      488b45f8       mov rax, qword [var_8h]
│      ││   0x004005d1      4883c002       add rax, 2
│      ││   0x004005d5      0fb600         movzx eax, byte [rax]
│      ││   0x004005d8      84c0           test al, al
│     ┌───< 0x004005da      740f           je 0x4005eb
│     │││   0x004005dc      488b45f8       mov rax, qword [var_8h]
│     │││   0x004005e0      4883c002       add rax, 2
│     │││   0x004005e4      0fb600         movzx eax, byte [rax]
│     │││   0x004005e7      3c33           cmp al, 0x33                ; 51
│    ┌────< 0x004005e9      740a           je 0x4005f5
│    │└───> 0x004005eb      b8ffffffff     mov eax, 0xffffffff         ; -1
│    │┌───< 0x004005f0      e9da000000     jmp 0x4006cf
│    └────> 0x004005f5      488b45f8       mov rax, qword [var_8h]
│     │││   0x004005f9      4883c003       add rax, 3
│     │││   0x004005fd      0fb600         movzx eax, byte [rax]
│     │││   0x00400600      84c0           test al, al
│    ┌────< 0x00400602      740f           je 0x400613
│    ││││   0x00400604      488b45f8       mov rax, qword [var_8h]
│    ││││   0x00400608      4883c003       add rax, 3
│    ││││   0x0040060c      0fb600         movzx eax, byte [rax]
│    ││││   0x0040060f      3c37           cmp al, 0x37                ; 55
│   ┌─────< 0x00400611      740a           je 0x40061d
│   │└────> 0x00400613      b8ffffffff     mov eax, 0xffffffff         ; -1
│   │┌────< 0x00400618      e9b2000000     jmp 0x4006cf
│   └─────> 0x0040061d      488b45f8       mov rax, qword [var_8h]
│    ││││   0x00400621      4883c004       add rax, 4
│    ││││   0x00400625      0fb600         movzx eax, byte [rax]
│    ││││   0x00400628      84c0           test al, al
│   ┌─────< 0x0040062a      740f           je 0x40063b
│   │││││   0x0040062c      488b45f8       mov rax, qword [var_8h]
│   │││││   0x00400630      4883c004       add rax, 4
│   │││││   0x00400634      0fb600         movzx eax, byte [rax]
│   │││││   0x00400637      3c5f           cmp al, 0x5f                ; 95
│  ┌──────< 0x00400639      740a           je 0x400645
│  │└─────> 0x0040063b      b8ffffffff     mov eax, 0xffffffff         ; -1
│  │┌─────< 0x00400640      e98a000000     jmp 0x4006cf
│  └──────> 0x00400645      488b45f8       mov rax, qword [var_8h]
│   │││││   0x00400649      4883c005       add rax, 5
│   │││││   0x0040064d      0fb600         movzx eax, byte [rax]
│   │││││   0x00400650      84c0           test al, al
│  ┌──────< 0x00400652      740f           je 0x400663
│  ││││││   0x00400654      488b45f8       mov rax, qword [var_8h]
│  ││││││   0x00400658      4883c005       add rax, 5
│  ││││││   0x0040065c      0fb600         movzx eax, byte [rax]
│  ││││││   0x0040065f      3c70           cmp al, 0x70                ; 112
│ ┌───────< 0x00400661      7407           je 0x40066a
│ │└──────> 0x00400663      b8ffffffff     mov eax, 0xffffffff         ; -1
│ │┌──────< 0x00400668      eb65           jmp 0x4006cf
│ └───────> 0x0040066a      488b45f8       mov rax, qword [var_8h]
│  ││││││   0x0040066e      4883c006       add rax, 6
│  ││││││   0x00400672      0fb600         movzx eax, byte [rax]
│  ││││││   0x00400675      84c0           test al, al
│ ┌───────< 0x00400677      740f           je 0x400688
│ │││││││   0x00400679      488b45f8       mov rax, qword [var_8h]
│ │││││││   0x0040067d      4883c006       add rax, 6
│ │││││││   0x00400681      0fb600         movzx eax, byte [rax]
│ │││││││   0x00400684      3c77           cmp al, 0x77                ; 119
│ ────────< 0x00400686      7407           je 0x40068f
│ └───────> 0x00400688      b8ffffffff     mov eax, 0xffffffff         ; -1
│ ┌───────< 0x0040068d      eb40           jmp 0x4006cf
│ ────────> 0x0040068f      488b45f8       mov rax, qword [var_8h]
│ │││││││   0x00400693      4883c007       add rax, 7
│ │││││││   0x00400697      0fb600         movzx eax, byte [rax]
│ │││││││   0x0040069a      84c0           test al, al
│ ────────< 0x0040069c      740f           je 0x4006ad
│ │││││││   0x0040069e      488b45f8       mov rax, qword [var_8h]
│ │││││││   0x004006a2      4883c007       add rax, 7
│ │││││││   0x004006a6      0fb600         movzx eax, byte [rax]
│ │││││││   0x004006a9      3c64           cmp al, 0x64                ; 100
│ ────────< 0x004006ab      7407           je 0x4006b4
│ ────────> 0x004006ad      b8ffffffff     mov eax, 0xffffffff         ; -1
│ ────────< 0x004006b2      eb1b           jmp 0x4006cf
│ ────────> 0x004006b4      488b45f8       mov rax, qword [var_8h]
│ │││││││   0x004006b8      4883c008       add rax, 8
│ │││││││   0x004006bc      0fb600         movzx eax, byte [rax]
│ │││││││   0x004006bf      84c0           test al, al
│ ────────< 0x004006c1      7407           je 0x4006ca
│ │││││││   0x004006c3      b8ffffffff     mov eax, 0xffffffff         ; -1
│ ────────< 0x004006c8      eb05           jmp 0x4006cf
│ ────────> 0x004006ca      b800000000     mov eax, 0
│ │││││││   ; XREFS: CODE 0x004005a0  CODE 0x004005c8  CODE 0x004005f0  CODE 0x00400618  CODE 0x00400640  CODE 0x00400668  
│ │││││││   ; XREFS: CODE 0x0040068d  CODE 0x004006b2  CODE 0x004006c8  
│ └└└└└└└─> 0x004006cf      5d             pop rbp
└           0x004006d0      c3             ret

```


pay attention to `cmp` it compare the hex value, if we get each hex value and convert we get the answer.

you can use [CyberChef](https://gchq.github.io/CyberChef)

```
cmp al, 0x31
cmp al, 0x33
cmp al, 0x33
cmp al, 0x37
cmp al, 0x5f
cmp al, 0x70
cmp al, 0x77
cmp al, 0x64
```

![crackme6-3](/Images/reversing-elf/crackme6-3.png)


## crackme7

This is same as before need to use `r2`

**Basic Command**

1. r2 -d ./crackme7

2. aaa

3. afl 

4. pdf @main

![crackme7-1](/Images/reversing-elf/crackme7-1.png)

when we execute binary didn't see any output as `push str.Wow_such_h4x0r_`.

we can see compare funtion on main with hex value, lets try to conver it.

no luck **"zi"** not correct answer, lets try to conver it to decimal numbers.

use this to conver from `hex` to `decimal` [rapidtables.com](https://www.rapidtables.com/convert/number/hex-to-decimal.html)

![crackme7-2](/Images/reversing-elf/crackme7-2.png)

![crackme7-3](/Images/reversing-elf/crackme7-3.png)


## crackme8

Same as eariler use `r2`

**Basic Command**

1. r2 -d ./crackme8

2. aaa

3. afl 

4. pdf @main


When we print(pdf) main we can see same `cmp` also something new `sym.imp.atoi`.

![crackme8-1](/Images/reversing-elf/crackme8-1.png)

Little google search and found out about this library,

![crackme8-2](/Images/reversing-elf/crackme8-2.png)

So we know it expecting number, copy the hex from compare `cmp eax, 0xcafef00d`  and convert it.

![crackme8-3](/Images/reversing-elf/crackme8-3.png)

```
┌──(root💀kali)
└─# ./crackme8 3405705229
Access denied.
  
┌──(root💀kali)
└─# ./crackme8 -889262067
Access granted.
flag{at_least_this_cafe_wont_leak_your_credit_card_numbers}
```

