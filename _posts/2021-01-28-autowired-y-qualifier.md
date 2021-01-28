---
layout: post
title: Autowired y Qualifier en Spring
description: Un modo sencillo de resolver e inyectar colaboradores en tus Spring beans.
image: 
author: Hernán Moreno
---

A partir de Spring 2.5, se introdujo la inyección de dependencia basada en anotaciones, lo cual generó un cambio de estilo (basado en Java) sobre como gestionar y organizar la metadata para trabajar con Spring. Una de las anotaciones principales es @Autowired que permite que Spring resuelva e inyecte beans colaboradores en nuestros beans.

Damos inicio a los artículos de wiring en Spring usando anotaciones en Java. Particularmente en este artículo veremos cómo habilitar el wiring automático y las  diversas formas de hacer wiring con los beans. También, hablaremos sobre la resolución de conflictos en los beans usando la anotación @Qualifier, así como sus posibles excepciones.

#### Habilitar @Autowired
Spring permite la inyección automática de dependencias. En otras palabras, al declarar todas las dependencias de los beans en Java como metadata, el contenedor de Spring puede conectar automáticamente las relaciones entre los beans que colaboran. Esto se llama wiring automático de Spring Beans.

A continuación veremos el modo más básico para habilitar el wiring automático.

#### Configuración basada en Java
Para la configuración basada en Java, en nuestra aplicación habilitamos la inyección basada en anotaciones:

{% gist e325128f4fd8dbb54f620dae3d957e45 %}

* La anotación @Configuration le indica a Spring que la clase AppConfig servirá como archivo de configuración para los Spring beans.
* La anotación @ComponentScan le indica a Spring que escanee dentro del paquete com.sistecma.proyecto por todas las clases que tengan la anotación @Component para que sean consideradas como _componentes_ dentro del contenedor de Spring. Cabe destacar que la anotación @Component es la anotación más genérica en Spring y una clase decorada con @Component, Spring la busca dentro del classpath y la registra como un Spring Bean. 
* La sintaxis de @Component es:

{% gist 84f49b80d41081e8a89b24ff4f74b9b9 %}  

**Nota-** Como alternativa, bajo configuración en xml podemos activar la configuración en java con la etiqueta:

{% gist b9c0ac108b35acdf69b477855fd875af %}

#### Usando el @Autowired
Después de habilitar la inyección por anotaciones, podemos usar el wiring automático en propiedades, setters(métodos) y constructores.

#### @Autowired en propiedades
Veamos cómo podemos anotar una propiedad usando @Autowired. Esto elimina la necesidad de getters y setters.

Primero, definamos un bean miPrinter:

{% gist fb24b8e735762909c448a91749f32f20 %}

A continuación, vamos a inyectar este bean en el PrintService bean mediante @Autowired en la definición de la propiedad:

{% gist 8f8ed2764595a08ec25e0f92e9c8b897 %}

Como resultado, Spring inyecta miPrinter cuando se crea PrintService.

#### @Autowired en Setters
Ahora agreguemos la anotación @Autowired en un setter(método).

En el siguiente ejemplo, se llama al método setter con la instancia de ConsolaPrinter cuando se crea PrintService:

{% gist 38949474a935c33cebe167b4278d0f8c %}

#### @Autowired en constructores
Finalmente, usemos @Autowired en un constructor.

Veremos que Spring inyecta una instancia de ConsolaPrinter como argumento al constructor PrintService :

{% gist e62572757b5709333fa1d078412f8e98 %}

#### Excepciones
Cuando se construye un bean, las dependencias @Autowired deberían estar disponibles. De lo contrario, si Spring no puede resolver un bean para el momento del wiring, lanzará una excepción. Esto en consecuencia, evita que el contenedor Spring se inicie con éxito. Veamos por ejemplo como se vería una excepción de este tipo:

{% gist 70bbe0926d41801e4dc541044b1e82a2 %}

Para solucionar esto, y en caso que querramos que el @Autowired no sea requerido para ese específico bean tenemos que cambiar así:

#### Resolución de beans vía @Component
Notaron que para el caso de ConsolaPrinter usamos la anotación @Component("miPrinter") y para PrintService solo usamos @Component. Lo que ocurre es que para el caso de ConsolaPrinter simplemente le estamos asignado un valor/indicador lógico como nombre del componente. En este caso le asigné el nombre de "miPrinter". Para el caso de PrintService este valor lógico no lo estamos considerando. 

Para estos casos Spring buscará realizar el wiring basados por tipo y nombre automácticamente siempre que sea posible. Hay casos en los que resulta ambiguo para el contenedor resolver dichas dependencias. Para estos casos es recomendable usar qualificadores.

* Un qualificador en Spring es un mecanismo en Spring que permite diferenciar (por nombre) beans con la misma implementación.

#### @Qualifier
Para que quede más claro el uso de @Qualifier. Vamos a realizar las siguientes modificaciones a nuestro ejemplo anterior:

{% gist 105f2ecfe269a3d6cecb5fba07782639 %}

Y asumamos que PrintService requiere como colaborador a Printer y no a ConsolaPrinter.

{% gist 18a8d07001c75ffe376e4cde402c0edf %}

Lo que obtenemos es la siguiente excepción (consideramos al nuevo bean miPrinter2):

{% gist 8183bbc5859c06f91b0b15aef7e7715d %}

Ahora aplicando @Qualifier

{% gist 336420a5d2905d27542454380b60f5ca %}

#### Conclusión
En este artículo, discutimos el wiring automático y las diferentes formas de usarlo. También examinamos formas de resolver excepciones comunes de wiring automático causadas por la falta de un bean o por una inyección de bean ambigua.

**Anterior post:** [Estados de Ciclo de Vida](https://sistecma.github.io/2021/01/25/estados-de-ciclo-de-vida.html)

