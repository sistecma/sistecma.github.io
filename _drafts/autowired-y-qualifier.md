---
layout: post
title: Autowired y Qualifier en Spring
description: Un modo sencillo de resolver e inyector beans colaboradores en tus Spring beans.
image: 
author: Hernán Moreno
---

A partir de Spring 2.5, se introdujo la inyección de dependencia basada en anotaciones, lo cual generó un cambio de estilo basado en java sobre como gestionar y organizar la metadata para trabajar con Spring. Entre estos cambios una de las anotaciones principales esta @Autowired que permite que Spring resuelva e inyecte beans colaboradores en nuestros beans.

Damos inicio a los artículos de wiring en Spring usando anotaciones en Java. Particularmente en este artículo veremos cómo habilitar el wiring automático y las  diversas formas de hacer wiring en los beans. También, hablaremos sobre la resolución de conflictos en los beans usando la anotación @Qualifier, así como sus posibles excepciones.

#### Habilitar @Autowired
Spring permite la inyección automática de dependencias. En otras palabras, al declarar todas las dependencias de los beans en un archivo de configuración, el contenedor de Spring puede conectar automáticamente las relaciones entre los beans que colaboran. Esto se llama wiring automático de Spring Beans.

A continuación veremos el modo más básico para habilitar el wiring automático.

##### Configuración basada en Java
Para la configuración basada en Java en nuestra aplicación, habilitamos la inyección basada en anotaciones:

{% gist e325128f4fd8dbb54f620dae3d957e45 %}

* La anotación @Configuration le indica a Spring que la clase AppConfig servirá como archivo de configuración para los Spring beans.
* La anotación @ComponentScan le indica a Spring que escanee dentro del paquete com.sistecma.proyecto por todas las clases que tengan la anotación @Component para que sean consideradas como _componentes_ dentro del contenedor de Spring. Cabe destacar que la anotación @Component es la anotación más genérica en Spring y una clase decorada con @Component Spring la busca dentro del classpath y la registra como un Spring Bean. 
* La sintaxis de @Component es:

{% gist 84f49b80d41081e8a89b24ff4f74b9b9 %}  

**Nota-** Como alternativa, bajo configuración en xml podemos activar la configuración en java con la etiqueta <context:annotation-config>. 

### Usando el @Autowired
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

{% gist %}