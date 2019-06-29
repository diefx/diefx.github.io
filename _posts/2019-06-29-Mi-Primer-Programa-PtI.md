---
title: "Primer Programa (Pt I)"
date: 2019-06-29
category: stm32f0cube
tags: [stm32f0cube, primer programa]
---

Llego la hora de crear nuestro primer programa **hola mundo**, este programa lo realizaremos sin el uso de ninguna librería de apoyo, pero si necesitaremos un par de archivos ya definidos los cuales obtendremos de la librería oficial [**STM32F0Cube**][1] de ST. Descarga el archivo y descomprimelo en tu lugar favorito de la computadora =). **NOTA:** tendrás que registrarte en la pagina de **ST**

Crea una carpeta donde estará tu flamante y nuevo programa de prueba

```bash
$ mkdir ~/test_bare
$ cd ~/test_bare
```

Copia a la carpeta de tu proyecto el archivo **linker**, el cual permitirá a tu programa saber como esta ordenada la memoria del micro. El archivo se encuentra en uno de los ejemplos que están en la librería de **ST** en la siguiente ruta:

```bash
STM32Cube_FW_F0_V1.10.0/Projects/STM32F072RB-Nucleo/Templates/SW4STM32/STM32F072RB-Nucleo/STM32F072RBTx_FLASH.ld
```

Copia a la carpeta de tu proyecto el archivo **startup** el cual permitirá a tu programa entre otras cosas iniciar desde el vector de reset, setear el valor del stack pointer y acomodar la tabla de vectores de interrupciones del micro. El archivo lo puedes encontrar en uno de los ejemplos que se encuentran en la librería de **ST** en la siguiente ruta:

```bash
STM32Cube_FW_F0_V1.10.0/Projects/STM32F072RB-Nucleo/Templates/SW4STM32/startup_stm32f072xb.s
```

Habrá que hacer una pequeña modificación al archivo **startup_stm32f072xb.s** para que no nos de problema con una función que manda llamar ( _y que por lo pronto no tenemos_ ). Abre con tu editor favorito el archivo y comenta la linea numero **99**. _Tal vez tengas que quitar la proteccion contra escritura para realizar lo anterior._

```c
98  /* Call the clock system intitialization function.*/
99    //bl  SystemInit
100 /* Call static constructors */
```

Crea un nuevo archivo al que llamaremos **main.c** y que contendrá nuestro código.

```bash
$ touch ~/test_bare
```

Escribiendo el codigo
---------------------

Abre el archivo con tu editor de texto favorito y escribe el siguiente codigo:

{% highlight c linenos %}
#define LED_PIN 5

int main ( void )
{
    volatile unsigned long i;

    /* Enable clock for GPIO port A */
    *((volatile unsigned long*)0x40021014) |= 0x00020000;
    /* Configure GPIOA pin 5 as output */
    *((volatile unsigned long*)0x48000000) |= (1 << (LED_PIN << 1));
    /* Configure GPIOA pin 5 in max speed */
    *((volatile unsigned long*)0x48000008) |= (3 << (LED_PIN << 1));

    for(;;)
    {
        /* Toggle pin 5 from port A */
        *((volatile unsigned long*)0x48000014) ^=(1 << LED_PIN);
        /* simple and practical delay */
        for(i=0;i<100000;i++);
    }
}
{% endhighlight %}

El anterior codigo solo hará parpadear el led conectado al **puerto A pin 5**, el cual esta presente en la tarjeta. Y te estaras preguntando que son todos esos números??, pues son las direcciones en las que se encuentran los registros que deberemos manipular para interactuar con el **pin A5**. Esta información la obtienes del manual de referencia del micro.

Hora de compilar nuestro programa. La compilación la realizaremos usando un archivo **makefile** el cual concentrara las ordenes de compilación que le pasaremos a nuestro compilador.

Crea un nuevo archivo llamado makefile en la carpeta de tu proyecto

```bash
$ touch ~/test_bare/makefile
```

Abre este nuevo archivo con tu editor de texto favorito y escribe lo siguiente

{% highlight makefile linenos %}
# Nombre que desees para tu proyecto
PROJECT = test
# Archivos fuente a compilar
OBJ = main.o startup_stm32f072xb.o
# linker script a usar
LINKERFILE = STM32F072RBTx_FLASH.ld
# Definiciones de control
SYMBOLS =
# directorio con los archivos a compilar .c y .s
VAPTH = .
# directorios con los archivos headers .h
INCLUDES = -I .

# Apartir de aqui no modifiques nada a menos que sepas lo que hases. ;)
CPU = -mcpu=cortex-m0 -mthumb -mlittle-endian

AS = arm-none-eabi-as
CC = arm-none-eabi-gcc
LD = arm-none-eabi-gcc
OD = arm-none-eabi-objdump
OC = arm-none-eabi-objcopy
SZ = arm-none-eabi-size

CCFLAGS = $(CPU) -Wall -fno-common -O0 -fomit-frame-pointer -Wstrict-prototypes -fverbose-asm
ASFLAGS = $(CPU)
LDFLAGS = $(CPU) -Wl,--gc-sections
LDFLAGS += -Wl,-Map=Output/$(PROJECT).map
OCFLAGS = -Oihex
ODFLAGS = -S

all : $(PROJECT)

$(PROJECT) : $(addprefix Output/,$(PROJECT).elf)
    $(OC) $(OCFLAGS) $< Output/$(PROJECT).hex
    $(OD) $(ODFLAGS) $< > Output/$(PROJECT).lst
    $(SZ) --format=berkeley $<

# Link files and generate elf file
Output/$(PROJECT).elf : $(addprefix Output/obj/,$(OBJ))
    $(LD) $(LDFLAGS) -T$(LINKERFILE) -o $@ $^

# Compile C source files
Output/obj/%.o : %.c
    mkdir -p Output/obj
    $(CC) $(CCFLAGS) $(INCLUDES) $(SYMBOLS) -c -o $@ $^

# Compile ASM files
Output/obj/%.o : %.s
    $(AS) $(ASFLAGS) -o $@ $^

clean :
    rm -r Output
{% endhighlight %}

No te apures si no sabes nada de **makefiles**, mas adelante explicaremos a detalle como crear uno desde cero. Otra cosa muy importante cuando escribes lo anterior usa **TABS** en la identacion y no espacios, o tendras errores.

Uff!!. Hora de compilar, solo escribe **make** en tu terminal, recuerda estar en la carpeta de tu proyecto

```bash
$ cd ~/test_bare
$ make
arm-none-eabi-gcc -mcpu=cortex-m0 -mthumb -mlittle-endian -Wl,--gc-sections -Wl,-Map=Output/test.map -TSTM32F072RBTx_FLASH.ld -o Output/test.elf Output/obj/main.o Output/obj/startup_stm32f072xb.o
arm-none-eabi-objcopy -Oihex Output/test.elf Output/test.hex
arm-none-eabi-objdump -S Output/test.elf > Output/test.lst
arm-none-eabi-size --format=berkeley Output/test.elf
   text    data     bss     dec     hex filename
    756    1076    1568    3400     d48 Output/test.elf
```

Si la terminal no te arrojo ningún error de compilación deberás tener un archivo **test.hex** en tu folder **Output/**, anda ve y revisa, porque hay que programar la tarjeta

Ahora, a programar
-----------------

Conecta al puerto USB tu tarjeta y en la terminal ejecutamos **OpenOCD**

```bash
$ cd ~/test_bare  #Recuerda siempre estar en el directorio de tu proyecto
$ sudo openocd -f interface/stlink-v2-1.cfg -f target/stm32f0x_stlink.cfg
```

En una nueva terminal mandaremos nuestro programa compilado a nuestra tarjeta usando el programa **telnet**, conectate al puerto **4444** de la siguiente manera

```bash
$ cd ~/test_bare #Recomendacion, siempre estar en el directorio de tu proyecto
$ telnet localhost 4444
Trying ::1...
Connection failed: Connection refused
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
Open On-Chip Debugger
>
```

Si te acepta la conexión, solo restara mandar el archivo **.hex**. Escribe los siguientes comandos en orden

```bash
> reset halt
Unable to match requested speed 1000 kHz, using 950 kHz
Unable to match requested speed 1000 kHz, using 950 kHz
adapter speed: 950 kHz
target halted due to debug-request, current mode: Thread
xPSR: 0xc1000000 pc: 0x08000bc4 msp: 0x20004000
> flash write_image erase Output/test.hex
auto erase enabled
device id = 0x20016448
flash size = 128kbytes
target halted due to breakpoint, current mode: Thread
xPSR: 0x61000000 pc: 0x2000003a msp: 0x20004000
wrote 2048 bytes from file Output/test.hex in 0.172037s (11.625 KiB/s)
> reset run
Unable to match requested speed 1000 kHz, using 950 kHz
Unable to match requested speed 1000 kHz, using 950 kHz
adapter speed: 950 kHz
>
```

El primer comando `reset halt` resetea y detiene al micro, el segundo `flash write_image erase Output/test.hex` manda el programa y lo escribe en la memoria y el ultimo `reset run` lo resetea y pone a correr el programa, asi que ya podras ver un feliz led parpadeando.

Para salir de telnet presiona `Ctrl+]` y despues escribe `quit`, y para desconectarte de OpenOCD solo presiona `Ctrl+c`

Para terminar te dejamos la estructura del directorio de tu proyecto, que debe lucir mas o menos asi

```bash
.
├── main.c
├── makefile
├── Output
│   ├── obj
│   ├── test.elf
│   ├── test.hex
│   ├── test.lst
│   └── test.map
├── startup_stm32f072xb.s
└── STM32F072RBTx_FLASH.ld
```


[1]: https://www.st.com/content/st_com/en/products/embedded-software/mcu-mpu-embedded-software/stm32-embedded-software/stm32cube-mcu-mpu-packages/stm32cubef0.html#overview

