# Pwnable.kr - leg

![leg.png](./leg.png)

Writeup y solución para el reto **leg** de la categoría *Toddler's Bottle* en Pwnable.kr.

## Descripción del Reto
El objetivo de este reto es comprender la arquitectura **ARM**, el funcionamiento de los registros especiales como el **Program Counter (PC)** y el **Link Register (LR)**, así como las diferencias de desplazamiento entre los modos de ejecución ARM y Thumb.

Analizando el código fuente (`leg.c`), observamos que el programa solicita un número entero (`key`) y comprueba la siguiente condición para liberar la flag:

```c
if( (key1()+key2()+key3()) == key ) {
    // ¡Muestra la flag!
}
```

Nuestra misión es calcular qué valor devuelve cada una de las tres funciones (`key1`, `key2` y `key3`) y sumar los resultados.

---

## Análisis y Resolución

### 1. Función `key1()`
El código en ensamblador de `key1` copia el valor del registro `pc` en `r3`:
```c
int key1(){
    asm("mov r3, pc\n");
}
```
* **Comportamiento en ARM:** Debido al *pipeline*, el registro `pc` en ARM siempre apunta **8 bytes por delante** de la instrucción actual.
* Dirección de la instrucción: `0x00008cdc`
* Cálculo: `0x00008cdc + 8 = 0x8ce4`
* **Valor decimal:** **`36068`**

---

### 2. Función `key2()`
Esta función cambia del modo estándar ARM (32 bits) al modo **Thumb** (16 bits) utilizando la instrucción `bx r6`.
```c
int key2(){
    // Mezcla código ARM y Thumb
}
```
* **Comportamiento en Thumb:** En el modo Thumb, el desplazamiento del `pc` es de **4 bytes**, y la instrucción posterior añade un offset adicional de 4 bytes.
* Dirección analizada: `0x00008d0c`
* **Valor decimal:** **`36108`**

---

### 3. Función `key3()`
`key3` copia el valor del registro **Link Register (`lr`)** en `r3`:
```c
int key3(){
    asm("mov r3, lr\n");
}
```
* **Comportamiento del LR:** El registro `lr` almacena la dirección de retorno a la que debe volver el programa tras finalizar una función (la instrucción siguiente a la llamada `bl key3` en el `main`), que se encuentra en la dirección `0x00008d80`.
* **Valor decimal:** **`36224`**

---

## Cálculo del Total

Sumamos los valores obtenidos de las tres funciones:

* `Total = key1() + key2() + key3()`
* `Total = 36068 + 36108 + 36224 = 108400`

---

## Explotación (Obtención de la Flag)

1. Conéctate al servidor oficial del reto mediante socat:
   ``` socat STDIO,raw,echo=0,opost,onlcr TCP:pwnable.kr:10007 ```
2. Ejecuta el binario `./leg` e introduce el valor calculado (`108400`) cuando te lo solicite:
   ```bash
   ./leg
   Daddy has very strong arm! : 108400
   Congratz!
   daddy_has_lot_of_ARM_muscl3
   ```
