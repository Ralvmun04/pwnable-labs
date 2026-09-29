# Writeup: cmd1 (Pwnable.kr)

![cmd1.png](./cmd1.png)

## 📌 Descripción del Reto
El reto **`cmd1`** de la categoría *Toddler's Bottle* nos introduce a cómo las restricciones de entorno (como la variable `PATH`) y los filtros lógicos de cadenas en C pueden afectar a la ejecución de comandos del sistema a través de funciones peligrosas como `system()`.

---

## 🔍 Análisis del Código Fuente
El código fuente proporcionado en el servidor (`cmd1.c`) es el siguiente:

```c
#include <stdio.h>
#include <string.h>

int filter(char* cmd){
    int r=0;
    r += strstr(cmd, "flag")!= 0;
    r += strstr(cmd, "sh")!= 0;
    r += strstr(cmd, "tmp")!= 0;
    return r;
}

int main(int argc, char* argv[], char** envp){
    putenv("PATH=/thankyouverymuch");
    if(filter(argv[1])) return 0;
    setregid(getegid(), getegid());
    system( argv[1] );
    return 0;
}
```

### Puntos Clave del Análisis:
1. **Filtro Estricto (`filter`)**: Utiliza la función `strstr()` para buscar las subcadenas `"flag"`, `"sh"` y `"tmp"` dentro de nuestro argumento (`argv[1]`). Si encuentra cualquiera de ellas, la variable acumuladora `r` aumenta, haciendo que la función devuelva un valor distinto de cero y provocando un `return 0` inmediato que aborta el programa.
2. **Modificación del Entorno (`PATH`)**: La instrucción `putenv("PATH=/thankyouverymuch");` altera la variable de entorno `PATH`. Esto significa que el sistema operativo ya no sabrá dónde encontrar los binarios comunes (como `cat`) si los invocamos únicamente por su nombre.
3. **Ejecución de Comandos (`system`)**: Si el filtro se supera con éxito, el argumento se pasa directamente a la función `system()` con los privilegios adecuados gracias a `setregid`.

---

## 🛠️ Metodología de Explotación

Para resolver el reto y superar los obstáculos, tuvimos que aplicar dos técnicas fundamentales:

1. **Uso de rutas absolutas**: Como el `PATH` está alterado y apunta a un directorio no válido (`/thankyouverymuch`), es obligatorio invocar los comandos escribiendo su ruta absoluta completa en el sistema de ficheros (por ejemplo, `/bin/cat` en lugar de solo `cat`).
2. **Evasión de filtros mediante comodines**: Dado que la palabra `flag` está estrictamente bloqueada por el filtro, podemos hacer uso de los comodines de la shell de Linux (como el asterisco `*`) para referirnos al archivo sin necesidad de escribir su nombre completo (por ejemplo, empleando `fl*`).

---

## 🚩 Explotación y Obtención de la Flag

Combinando las soluciones anteriores, ejecutamos el binario pasándole el comando preparado como argumento:

```bash
./cmd1 "/bin/cat fl*"
```

Esto permitió sortear el filtro de C, evitar el bloqueo del `PATH` y conseguir volcar el contenido del archivo deseado con éxito.

Finalmente la flag que nos da es la siguiente:

```flag
PATH_environment?_Now_I_really_g3t_it,_mommy!
```
