# Writeup: memcpy (pwnable.kr)

![memcpy](./memcpy.png)

## Información General
- Reto: memcpy
- Plataforma: pwnable.kr (Toddler's Bottle)
- Objetivo: Resolver el experimento de rendimiento de la función memcpy completando las pruebas de copia de bloques sin provocar fallos de segmento.

## Explicación del Código y Teoría
El reto presenta un programa en C que compara slow_memcpy (copia byte a byte) y fast_memcpy (copia optimizada por bloques). 

La función fast_memcpy utiliza ensamblador en línea con instrucciones vectoriales de hardware (como movdqa). Estas instrucciones exigen que la memoria asignada mediante malloc esté estrictamente alineada a múltiplos de 16 bytes. Si el tamaño del búfer rompe esta alineación exacta, el programa provoca un Segmentation Fault y se cierra.

El programa ejecuta un bucle de 10 iteraciones pidiendo un tamaño de búfer dentro de un rango dinámico específico en cada paso.

## Metodología de Resolución (Enfoque Empírico)
En lugar de calcular teóricamente las alineaciones de memoria de cada malloc, se aplicó una técnica de depuración empírica directa:

1. Creación de entradas: Se generó un archivo de texto plano (input.txt) con una lista inicial de 10 tamaños basados en potencias de dos para los rangos pedidos.
2. Ejecución y detección: Se lanzó el programa redirigiendo la entrada mediante el archivo. El binario avanzaba hasta que un tamaño incompatible con fast_memcpy provocaba un Segmentation Fault, revelando en qué experimento ocurría.
3. Parcheo dinámico: Usando el comando sed -i, se modificaba al vuelo el valor problemático sumándole un pequeño offset de 8 bytes para desplazar la alineación sin salir del rango permitido.
4. Iteración: Se repitió el proceso hasta que los 10 experimentos finalizaron con éxito, mostrando la bandera al final.

## Comandos Utilizados

cat << EOF > input.txt
8
16
32
64
128
256
512
1024
2048
4096
EOF

./memcpy < input.txt

sed -i 's/128/136/' input.txt
sed -i 's/256/264/' input.txt
sed -i 's/512/520/' input.txt
sed -i 's/1024/1032/' input.txt
sed -i 's/2048/2056/' input.txt

./memcpy < input.txt

Finalmente acabo siendo correcto y obtuvimos la flag:
```flag
b0thers0m3_m3m0ry_4lignment
```
