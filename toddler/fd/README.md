# 🛡️ Pwnable.kr - fd (Toddler's Bottle)

![fd.png](./fd.png)

## 📌 Descripción del Reto
El reto **fd** introduce conceptos fundamentales de programación en C y el manejo de descriptores de archivos (*File Descriptors*) en sistemas operativos Linux. El objetivo es manipular la entrada del programa para cumplir con una validación lógica y obtener la flag.

---

Al comenzar nos encontramos con lo que parece ser 3 archivos principales, uno llamado `fd`, otro `fd.c` y otro que es la flag.

No tenemos permisos para ejecutar ni para hacer `cat` a la flag, por lo que hacemos `cat` a `fd.c`.

## 🔍 Análisis del Código Fuente

El código fuente proporcionado en el binario es el siguiente:

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
char buf[32];
int main(int argc, char* argv[], char* envp[]){
	if(argc<2){
		printf("pass argv[1] a number\n");
		return 0;
	}
	int fd = atoi( argv[1] ) - 0x1234;
	int len = 0;
	len = read(fd, buf, 32);
	if(!strcmp("LETMEWIN\n", buf)){
		printf("good job :)\n");
		setregid(getegid(), getegid());
		system("/bin/cat flag");
		exit(0);
	}
	printf("learn about Linux file IO\n");
	return 0;
}
