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
