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
2번째 인자값을 buffer에 복사하고 buffer값을 출력하는 구도  

### strcpy함수의 취약점   
>(srtingcopy)strcpy함수는 strcpy(인자1, 인자2); 일때
>인자1에 인자 2의 값을 복사해주는데 이떄 인자1의 크기에 상관없이
>인자 2의 모든 값을 복사해서 인자1의 영역너머를 우리가 변조할 수 있게됨  

### 인자 전달방법
>파일을 실행할때는 "./파일명"으로 실행  
>여기에 인자를 포함해서 전달하려면 뒤에 인자로 넣을 것 을 추가    
>파이썬으로 전달하려면? + ex) ./gremlin `python -c 'print "A"*100'`  `


## 이제 공략해봅시다.
