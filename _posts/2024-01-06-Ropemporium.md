---
title: Solución ropemporium ret2win
date: 2024-01-06
categories: [rop, 64bits, bof]
tags: [64bits, bof, rop] #minúsculas-
---

# Ret2win
En esta entrada, abordaré la resolución del desafío **ropemporium** "ret2win" para sistemas de **64 bits** **x86_64**.

**Protecciones del Binario:**

Antes de comenzar, descargamos el binario y analizamos sus protecciones. Se destaca la activación de **NX** (Data Execution Prevention) y la desactivación del **ASLR** mediante el siguiente comando:
```bash
echo 0 | sudo tee /proc/sys/kernel/randomize_va_space
```

```javascript
checksec --file=ret2win
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH	Symbols		FORTIFY	Fortified	Fortifiable	FILE
Partial RELRO   No canary found   NX enabled    No PIE          No RPATH   No RUNPATH   69 Symbols	 No	0		3		ret2win

```
Si ejecutamos el binario, observamos un posible desbordamiento de búfer.
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
## Herramientas Utilizadas:
### **pwndbg**:
Vamos a utilizar pwndbg, un plugin de gdb, para depurar el binario y obtener información crucial.

![](/assets/img/rop/pwndbg2.png)

**gdb** nos mostrará los registros a continuación:

![](/assets/img/rop/pwndbg.png)

¿Qué podemos ver aquí? **RBP** sobrescrito con **A** en hexadecimal **41** y también **RSP**.

¿Por qué no hemos sobrescrito también **RIP?** Bueno, en realidad, sí hemos sobrescrito **RIP**, el **RIP** no veremos el valor, ya que contiene ret. Si vemos el valor de ret, contiene nuestras A. Esta es una particularidad a tener en cuenta al explotar binarios en 64 bits.
## **Análisis de Offset**:
Vamos a crear un pattern para saber el offset. Para eso, podemos usar **cyclic** de **pwndbg** de la siguiente forma:
```bash
pwndbg> cyclic 200
aaaaaaaabaaaaaaacaaaaaaadaaaaaaaeaaaaaaafaaaaaaagaaaaaaahaaaaaaaiaaaaaaajaaaaaaakaaaaaaalaaaaaaamaaaaaaanaaaaaaaoaaaaaaapaaaaaaaqaaaaaaaraaaaaaasaaaaaaataaaaaaauaaaaaaavaaaaaaawaaaaaaaxaaaaaaayaaaaaaa
```
Luego copiamos el valor de **ret**, que en este caso es:


```bash
 ► 0x400755 <pwnme+109>    ret    <0x6161616161616166>
 ```
Y hacemos esto en **pwndbg** para saber el **offset**:
```bash
pwndbg> cyclic -l 0x6161616161616166
Finding cyclic pattern of 8 bytes: b'faaaaaaa' (hex: 0x6661616161616161)
Found at offset 40
```
El **Offset** es de **40 bytes**. A continuación, se debe provocar el desbordamiento del *búfer* para poder saltar otra vez al inicio del programa y así poder leer la *función* que necesitamos.

## Identificación de Gadget **"ret"**:
Para saber la dirección de **ret**, podemos usar **ropgadget** que está dentro de pwndbg. Aquí podemos ver que nos retorna **95 gadgets**, entre ellos **ret**.
![](/assets/img/rop/ropgadget.png)

Así que **ret** sabemos que es el siguiente:

![](/assets/img/rop/ret.png)

## Localización de la Función **"ret2win"**:

Ahora nos falta saltar a la función que queremos para leer la *flag*. Para saber eso, necesitamos saber primero qué función es, y para eso usaremos **radare2**.

![](/assets/img/rop/radare.png)

En la imagen anterior, podemos ver con **afl** las funciones que tiene el binario. Allí dentro vemos una que es **sym.ret2win**, que si la investigamos con **pdf**, podemos ver que la función hace un cat a flag.txt. De esta forma, obtenemos la dirección de nuestra función.

# Exploit final

Con todos los elementos anteriores, construimos nuestro **exploit** final. La imagen a continuación muestra la secuencia de pasos y la explotación exitosa.

![](/assets/img/rop/exploit.png)