---
title:      "Primer Programa (Pt II)"
date:       2019-06-30
categories: stm32f0cube
tags:       [stm32f0cube, primer programa]
---

En el anterior posts creamos nuestro primer programa y lo programamos en nuestra tarjeta **Nucleo-f072rb**, un programa muy sencillo que solo hacia parpadear su led verde, este programa lo realizamos manipulando los registros de nuestro micro, habrás notado que en el código usamos las direcciones de los registros y una notación extraña conocida como **cast a punteros**, deja te explico.

Sabemos que el registro para activar el reloj de el **puerto A** se encuentra en la dirección **0x40021014** si nosotros escribimos la siguiente linea en C:

```c
0x40021014 |= 0x00020000:
```

Nos arrojaría un error de compilación ya que no es posible asignar un valor a otro valor constante, lo que tenemos que decirle a C y al compilador cuando procese nuestro código es que el primer numero es en realidad la dirección del registro al que queremos cargar el valor, esto lo realizamos mediante un **cast a puntero de tipo unsigned long**

```c
*((volatile unsinged long*)0x40021014) |= 0x00020000;
```

Ahora le estamos diciendo a nuestro programa que haga una operación **OR** con el valor **0x00020000** y el valor que esta en la dirección **0x40021014** ( _lo anterior coloca en uno el bit 17 del registro RCC-AHBENR_ ). Utilizar esta notación en todo nuestro código seria algo muy poco practico pues es muy difícil recordar direcciones de registros, es mucho mas sencillo recordar sus nombres.

Tomemos nuestro código anterior y escribamos algunas definiciones para nuestros registros


{% highlight c linenos %}
/*Definimos los registros a utilizar*/
#define RCC_AHBENR      *((volatile unsigned long*)0x40021014)
#define GPIOA_MODER     *((volatile unsigned long*)0x48000000)
#define GPIOA_OSPEEDR   *((volatile unsigned long*)0x48000008)
#define GPIOA_ODR       *((volatile unsigned long*)0x48000014)
#define LED_PIN 5

int main ( void )
{
    volatile unsigned long i;

    /* Enable clock for GPIO port A */
    RCC_AHBENR |= 0x00020000;
    /* Configure GPIOA pin 5 as output */
    GPIOA_MODER |= (1 << (LED_PIN << 1));
    /* Configure GPIOA pin 5 in max speed */
    GPIOA_OSPEEDR |= (3 << (LED_PIN << 1));

    for(;;)
    {
        /* Toggle pin 5 from port A */
        GPIOA_ODR ^=(1 << LED_PIN);
        /* simple and practical delay */
        for(i=0;i<100000;i++);
    }
}
{% endhighlight %}

Puedes ver que ahora nuestro código luce mas legible y se entiende mejor que registros estamos manipulando para hacer que nuestro led parpadee. La información sobre los nombres y las direcciones de los registros la encuentras en la pagina 46 del manual de referencia del micro.

Lo anterior se puede mejorar aun mas utilizando estructuras. Si estudias el manual del micro podrás notar que los registros estan agrupados por perifericos ( _como el puerto A, el puerto serial, el adc y etc..._ ), y los registros que componen estos perifericos estan ordenados de manera consecutiva, por ejemplo los registros del puerto A ( _Tabla 24, pagina 163_ )

```bash
0x48000000  MODER
0x48000004  OTYPER
0x48000008  OSPEEDR
0x4800000C  PUPDR
0x48000010  IDR
0x48000014  ODR
0x48000018  BSRR
0x4800001C  LCKR
0x48000020  AFRL
0x48000024  AFRH
0x48000028  BRR
```

Podemos notar que las direcciones avanzan de a cuatro, esto es porque cada registros tiene una longitud de 32 bits o 4 bytes ( _por eso hsimos el cast con un unsigned long_ ) con esto como referencia en nuestro codigo podemos definir una estructura que representara nuestro puerto A.

```c
typedef struct 
{
    volatile unsigned long MODER;
    volatile unsigned long OTYPER;
    volatile unsigned long OSPEEDR;
    volatile unsigned long PUPDR;
    volatile unsigned long IDR;
    volatile unsigned long ODR;
    volatile unsigned long BSRR;
    volatile unsigned long LCKR;
    volatile unsigned long AFRL;
    volatile unsigned long AFRH;
    volatile unsigned long BRR;
}GPIOA_Typedef;
```

Y para poder utilizarlo en nuestro código definimos nuestro puerto A con el nombre de **GPIOA** y a la dirección le realizamos un cast a puntero de esa estructura que definimos con nuestros registros.

```c
#define GPIOA   (GPIOA_Typedef*)0x48000000
```

Listo, cunado quieras usarlo en tu código solo escribes ( _digamos que quieres escribir en el registro ODR_ )

```c
GPIOA->ODR = 0x00000001:
```

Ahora imagínate que escribes las definiciones de las estructuras para **TODOS** los perifericos del micro con cada uno de sus registros, y **TODA** esa información la colocas en un archivo header llamado **stm32f072rb.h** y lo incluyes en tu código, entonces tu aplicación de parpadeo de led quedaría asi

{% highlight c linenos %}
#include "stm32f072rb.h"

#define LED_PIN 5

int main ( void )
{
    volatile unsigned long i;

    /* Enable GPIOA clock */
    RCC->AHBENR |= 0x00020000;
    /* Configure GPIOA pin 5 as output */
    GPIOA->MODER |= (1 << (LED_PIN << 1));
    /* Configure GPIOA pin 5 in max speed */
    GPIOA->OSPEEDR |= (3 << (LED_PIN << 1));

    for(;;)
    {
        /* Toggle pin 5 from port A */
        GPIOA->ODR ^= (1 << LED_PIN);
        /* simple and practical delay */
        for(i=0;i<300000;i++);
    }
}
{% endhighlight %}

Pues déjame decirte que no necesitas escribir dicho archivo con todas esas definiciones, ( _son bastantes_ ), porque alguien mas ya lo hizo por ti ...

Por cierto, esa forma de definir los registros del micro con punteros a estructuras `[Periferico]->[registro]` no es casualidad lo dicta un estandar llamado **CMSIS** ( _despues te dare detalles sobre este estandar_ )

