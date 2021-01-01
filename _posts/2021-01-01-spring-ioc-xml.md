---
layout: post
title: Spring IoC con Xml
description: Revisión del uso de Spring con Xml.
image: 
author: Hernán Moreno
---

La Inversión de Control (IoC) en Spring puede realizar vía Xml o con plana y simple configuración en Java. Actualmente es estándar usar la configuración vía Java sin embargo existen ciertos casos de uso para los cuales usar Xml es lo más apropiado. Por ejemplo:

Cuando tienes que usar una librería  típica empaquetada en un jar ,simplemente no tienes accesos a los fuentes, o quieres que tu código se mantenga lo más limpio posible como POJO  evitando así quedar acoplados y dependientes de Spring. 

**Nota-** Como indiqué previamente la tendencia es usar anotaciones para casi todo, sin embargo por propósitos educativos abordamos brevemente la configuración vía xml, en otro artículo revisamos la configuración vía Java. 


#### AbstractApplicationContext

Para usar IoC con Xml recomiendo usar [AbstractApplicationContext](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/support/AbstractApplicationContext.html), y no puramente [ApplicationContext](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/ApplicationContext.html) por una sencilla razón. Con AbstractApplicationContext podemos manejar facilmente en el Xml ciclos de post-construcción (o inicialización adicionales) y destrucción de beans personalizadas(útil para liberar recursos de manera explícita. Por ejemplo cerrar conecciones de base de datos). Mientras que con ApplicationContext solo podemos aplicar post-construcción.

{% gist 5ee456ee8ead01d3f45a97c6eb67cf72 %}

#### El mini-proyecto de juguete

Para entender como funciona, vamos a revisar los conceptos alrededor de un ejemplo básico. Vamos a asumir que tenemos el siguiente proyecto de juguete:
* Queremos simular la comunicación entre una computadora y un servidor.
* La computadora se puede encender y apagar.
* El servidor asumimos que esta siempre encendido.
* Ambos equipos tienen su propio [hostname](https://es.wikipedia.org/wiki/Hostname) y pueden hacer [ping](https://es.wikipedia.org/wiki/Ping)
* La computadora tiene un monitor
* El monitor solo puede "mostrar" información.
* Existe una red entre la computadora y el servidor que se puede configurar y destruir; asi como puede establer ping para comunicar los equipos.

**Nota-** A fín de mostrar las carácteristicas más importantes de Spring con Xml verás en el código que he sobre-diseñado en algunas partes. Todo esto con fines didácticos y demostrativos de las capacidades de Spring.

#### Diseño
* Vamos a usar Spring con Xml metadata. 
* La clase que arranca al contenedor Spring la llamaremos: [XmlConfig.java](https://github.com/sistecma/spring-desde-cero/blob/1fcdf6dc9bd1319b3a8fbf401c638ad25aad9ea3/app/ioc-di-xml/src/main/java/com/sistecma/springdesdecero/iocdi/XmlConfig.java#L6)




#### Conclusiones

En este breve artículo, hemos discutido los aspectos básicos sobre como funciona el contenedor de Spring y su ApplicationContext. 

Para una comprensión completa revisar el código fuente incluido en este [repositorio](https://github.com/sistecma/spring-desde-cero).         

