---
title: Heap Overflow
date: 2024-01-24
categories: [heap, overflow, bof]
tags: [heap, overflow, bof]
---
# Heap Overflow
¿Qué es el (**Heap**)?

El **heap** es una región de memoria asignada a cada programa. A diferencia de la pila (**stack**), la memoria del heap puede asignarse dinámicamente. Esto significa que el programa puede ``"solicitar"`` y ``"liberar"`` memoria del segmento del **heap** siempre que lo necesite. Además, esta memoria es global, es decir, se puede acceder y modificar desde cualquier lugar dentro de un programa y no está localizada en la función donde se asigna. Esto se logra utilizando ``"punteros"`` para hacer referencia a la memoria asignada dinámicamente, lo que a su vez conlleva a una pequeña degradación en el rendimiento en comparación con el uso de variables locales (en la **stack**).


Normalmente, usamos funciones como **malloc** y **free**, las cuales están incluidas en la biblioteca **stdlib.h**.

Definamos ahora **malloc** y **free**

``malloc``: Es una función en el lenguaje de programación C que se utiliza para asignar dinámicamente un bloque de memoria en el **heap**. Toma como argumento el tamaño en **bytes** que se desea reservar y devuelve un **puntero** al inicio de la memoria asignada. Es importante liberar esta memoria cuando ya no sea necesaria para evitar fugas de memoria.

``free``: Es una función en C que se utiliza para liberar la memoria previamente asignada con **malloc** u otras funciones de asignación dinámica de memoria. Al liberar la memoria, se indica al sistema operativo que puede reutilizar esa porción de memoria para otros fines. Es crucial utilizar **free** para evitar problemas de pérdida de memoria **memory leaks**.

# Heap Exploitation

Tipos de ataques 

|  Vulnerabilidad              | Descripción      | Técnica de Explotación |
| :--------------------------- | :--------------- | ----------------------:|
| Double Free          | Making ``malloc`` return an already allocated fastchunk     | Disrupting fastbin link structure           |
| Forging chunks                | Making ``malloc`` return a nearly arbitrary pointer   | Disrupt the fastbin by freeing a chunk twice|
| Unlink Exploit | Getting (nearly) arbitrary write access |  Freeing a corrupted chunk and exploiting unlink |
| Shrinking Free Chunks |  Making ``malloc`` return a chunk overlapping with an already allocated chunk | Corrupting a free chunk by decreasing its size 
| House of Spirit | Making malloc return a nearly arbitrary pointer | Forcing freeing of a crafted fake chunk
| House of Lore | Making ``malloc`` return a nearly arbitrary pointer | Disrupting smallbin link structure
| House of Force | Making ``malloc`` return a nearly arbitrary pointer | Overflowing into top chunk's header 
| House of Einherjar | Making ``malloc`` return a nearly arbitrary pointer | Overflowing a single byte into the next chunk

faltan mas
## Casos Practicos ejemplos


# Como detectar fallos en el Heap 
