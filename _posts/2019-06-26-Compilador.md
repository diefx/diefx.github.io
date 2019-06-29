---
title:            "Compilador"
date:             2019-06-26
categories:       toolchain
tags:             [tools, gcc]
---

Comencemos con la puesta a punto de nuestro entorno de trabajo y la primera herramienta de desarrollo a instalar sera el compilador para microcontroladores [**ARM-GCC**][4]. El compilador GCC es totalmente libre y lo mejor, no presenta limitación alguna =). En la terminal escribe:

```bash
$ sudo pacman -S arm-none-eabi-gcc arm-none-eabi-newlib
```

Este compilador es la versión GCC para microcontroladores con soporte para los que poseen un Core **ARM** como el que trae nuestra herramienta ( _de echo es un Cortex-M0 pero eso lo veremo mas adelante_ ). Si observas estamos instalando dos paquetes, el primero es el propio compilador GCC, mientras que el segundo es librería estándar de C reducida a una versión para microcontroladores llamada **Newlib**. 

Después de instalar comprobamos que todos este bien escribiendo en la terminal:

```bash
$ arm-none-eabi-gcc -v
Using built-in specs.
COLLECT_GCC=arm-none-eabi-gcc
COLLECT_LTO_WRAPPER=/usr/lib/gcc/arm-none-eabi/9.1.0/lto-wrapper
Target: arm-none-eabi
Configured with: /build/arm-none-eabi-gcc/src/gcc-9.1.0/configure --target=arm-none-eabi --prefix=/usr --with-sysroot=/usr/arm-none-eabi --with-native-system-header-dir=/include --libexecdir=/usr/lib --enable-languages=c,c++ --enable-plugins --disable-decimal-float --disable-libffi --disable-libgomp --disable-libmudflap --disable-libquadmath --disable-libssp --disable-libstdcxx-pch --disable-nls --disable-shared --disable-threads --disable-tls --with-gnu-as --with-gnu-ld --with-system-zlib --with-newlib --with-headers=/usr/arm-none-eabi/include --with-python-dir=share/gcc-arm-none-eabi --with-gmp --with-mpfr --with-mpc --with-isl --with-libelf --enable-gnu-indirect-function --with-host-libstdcxx='-static-libgcc -Wl,-Bstatic,-lstdc++,-Bdynamic -lm' --with-pkgversion='Arch Repository' --with-bugurl=https://bugs.archlinux.org/ --with-multilib-list=rmprofile
Thread model: single
gcc version 9.1.0 (Arch Repository)
```

Con el compilador instalado ya podemos empezar a escribir código en nuestro editor de código favorito (_el favorito sera el de tu eleccion_) ojo una cosa es editor y otro es un ambiente de desarrollo integrado, pero aquí no usaremos de los segundos. Editores hay muchos y muy variados como Atom, Vim, Emacs, VS Code, etc ..


[1]: https://ww.st.com
[2]: https://www.archlinux.org
[3]: https://www.st.com/content/st_com/en/products/evaluation-tools/product-evaluation-tools/mcu-mpu-eval-tools/stm32-mcu-mpu-eval-tools/stm32-nucleo-boards/nucleo-f072rb.html 
[4]: https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-rm

