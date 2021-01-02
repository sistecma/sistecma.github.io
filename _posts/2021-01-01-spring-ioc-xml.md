---
layout: post
title: Spring IoC con Xml
description: Revisión del uso de Spring con Xml.
image: 
author: Hernán Moreno
tags: SpringFramework
---

La Inversión de Control (IoC) en Spring puede realizarse vía Xml o con configuración en Java. Actualmente el estándar es usar la configuración vía Java, sin embargo existen ciertos casos de uso para los cuales usar Xml es lo más apropiado. Por ejemplo:

Cuando tienes que usar una librería  típica empaquetada en un jar ,y simplemente no tienes accesos a los fuentes, o si quieres que tu código se mantenga lo más limpio posible como [POJOs](https://es.wikipedia.org/wiki/Plain_Old_Java_Object)  evitando así que tus desarrollos queden acoplados y dependientes de Spring. 

**Nota-** Como indiqué previamente, la tendencia es usar anotaciones para casi todo, sin embargo por propósitos educativos abordamos brevemente la configuración vía Xml puesto que la considero más simple para iniciar con Spring. En otro artículo revisaremos la configuración vía Java a fondo. Así mismo si tus nociones sobre IoC no estan claras, te recomiendo que revises el post como parte de la Serie de artículos Spring desde Cero porque aquí ya estoy asumiendo que lo tienes claro. Te dejo el enlace respecto a ello [aquí]({% post_url 2020-12-28-welcome-ioc-di %})


#### AbstractApplicationContext

Para usar IoC con Xml recomiendo usar [AbstractApplicationContext](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/support/AbstractApplicationContext.html), y no puramente [ApplicationContext](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/ApplicationContext.html) por una sencilla razón. Con AbstractApplicationContext podemos manejar facilmente en el Xml ciclos de post-construcción (o inicialización adicionales) y destrucción de beans personalizadas(útil para liberar recursos de manera explícita. Por ejemplo cerrar conecciones de base de datos). Mientras que con ApplicationContext solo podemos aplicar post-construcción.

{% gist 5ee456ee8ead01d3f45a97c6eb67cf72 %}

#### Tipos de Inyección
Cuando configuras beans vía Xml, lo común es que inyectes tus dependencias vía método o constructor. 

* En la inyección basada en métodos, proporcionamos las dependencias mediando el paso de valores que son requeridos como parámetros de los setXX de la clase dependiente. Donde XX corresponde a la propiedad o atributo de dicha clase. 
* En la inyección basada en constructor, le proporcionamos las dependencias requeridas al constructor de la clase dependiente.

En la práctica, preferir la inyección vía constructor tiene sus ventajas. Puesto que creamos el objeto llamando a su constructor. Así el constructor espera todas las dependencias requeridas como parámetros, entonces podemos estar 100% seguros de que la clase nunca será instanciada sin sus dependencias inyectadas. El contenedor de IoC se asegura de que todos los argumentos proporcionados en el constructor estén disponibles antes de pasarlos al mismo. Esto ayuda a prevenir el infame NullPointerException por ejemplo. Otro beneficio es que previene las [dependencias circulares](https://es.qaz.wiki/wiki/Circular_dependency)

Finalmente, La inyección por constructor es extremadamente útil ya que no tenemos que escribir la lógica de negocios separada en todas partes para verificar si todas las dependencias requeridas están cargadas, simplificando así la complejidad del código.

#### Organización de la metadata
Los Xml de configuración pueden estar alojados en un solo archivo o divididos en varios archivos, para mejor organización. Al momento de arrancar el contenedor, todas la metadata es cargada como si fuera una solo archivo. Como se pueden cargar varios archivos Xml?. Lo puedes hacer vía el [ClassPathXmlApplicationContext](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/support/ClassPathXmlApplicationContext.html) simplemente pasas como parámetros los archivos Xmls a su constructor, obviamente dichos archivos deben estar visibles en tu classpath.

El otro modo, es vía imports directamente dentro de los Xmls. Así mismo, todos los Xml deben ser visibles en tu classpath.  

#### El mini-proyecto de juguete
Para entender como funciona todo esto, vamos a revisar los conceptos alrededor de un ejemplo básico. Vamos a asumir que tenemos el siguiente proyecto de juguete:
* Queremos simular la comunicación entre una computadora y un servidor.
* La computadora se puede encender y apagar.
* El servidor, asumimos que esta siempre encendido.
* Ambos equipos tienen su propio [hostname](https://es.wikipedia.org/wiki/Hostname) y pueden hacer [ping](https://es.wikipedia.org/wiki/Ping)
* La computadora tiene un monitor
* El monitor solo puede "mostrar" información.
* Existe una red entre la computadora y el servidor que se puede configurar y destruir; asi como puede establer ping para comunicar los equipos.

**Nota-** A fín de mostrar las carácteristicas más importantes de Spring con Xml verás en el código que he sobre-diseñado en algunas partes y esta bien, todo esto es con fines didácticos y demostrativos a fín de resaltar las capacidades de Spring.

#### Diseño
* Vamos a usar Spring con Xml metadata. 
* Tenemos una interfaz que contiene el contrato de un [Equipo](https://github.com/sistecma/spring-desde-cero/blob/1fcdf6dc9bd1319b3a8fbf401c638ad25aad9ea3/app/ioc-di-xml/src/main/java/com/sistecma/springdesdecero/iocdi/Equipo.java#L3)

{% gist 20c28f1701ef8aca69b4da47281e699 %}

* Tenemos una interfaz que contiene el contrato de un [Monitor](https://github.com/sistecma/spring-desde-cero/blob/1fcdf6dc9bd1319b3a8fbf401c638ad25aad9ea3/app/ioc-di-xml/src/main/java/com/sistecma/springdesdecero/iocdi/Monitor.java#L3). Porqué una interfaz y no directamente una clase?. Porque pueden haber diversos tipos de monitores. Adicionalmente quiero hacer evidente uno de los puntos fuertes de Spring que tiene que ver con el desacople y como elegir en configuración la implementación.

{% gist 0b1fa370c7cb3d764733af050d849e1c %}

* Tenemos una implementación de Monitor llamada [MonitorImpl](https://github.com/sistecma/spring-desde-cero/blob/1fcdf6dc9bd1319b3a8fbf401c638ad25aad9ea3/app/ioc-di-xml/src/main/java/com/sistecma/springdesdecero/iocdi/MonitorImpl.java#L4)  

{% gist c2a80788eb40c5349e909230e6b3fce2 %}

* Tenemos una clase [Computadora](https://github.com/sistecma/spring-desde-cero/blob/1fcdf6dc9bd1319b3a8fbf401c638ad25aad9ea3/app/ioc-di-xml/src/main/java/com/sistecma/springdesdecero/iocdi/ComputadoraDI.java#L3) que implementa el contrato de Equipo. Tiene como dependencias a dependencias a Monitor, su hostname, e ip. A esta clase le vamos inyectar estas dependencias vía su constructor. 

{% gist 074047535120f980942095baaa8471f3 %}

* Tenemos una clase [Servidor](https://github.com/sistecma/spring-desde-cero/blob/1fcdf6dc9bd1319b3a8fbf401c638ad25aad9ea3/app/ioc-di-xml/src/main/java/com/sistecma/springdesdecero/iocdi/Servidor.java#L4) que implementa el contrato de Equipo. Tiene como dependencias hostname e ip estos atributos serán inyectados vía setters (usando sus métodos setXX).

{% gist cd8b3675ef88f19fdfc3a2993f9cee6a %}

* Tenemos una clase llamada [Red](https://github.com/sistecma/spring-desde-cero/blob/a4e5948dbda5336f49d5390bb31b7d568d294f65/app/ioc-di-xml/src/main/java/com/sistecma/springdesdecero/iocdi/Red.java#L4) que tiene como dependencia a los equipos Computadora y Servidor. Así mismo la inyección será vía métodos. 

{% gist 52bf7b9f9f4eb0a938811a5a3e741280 %}

* La clase que arranca al contenedor Spring la llamaremos: [XmlConfig.java](https://github.com/sistecma/spring-desde-cero/blob/1fcdf6dc9bd1319b3a8fbf401c638ad25aad9ea3/app/ioc-di-xml/src/main/java/com/sistecma/springdesdecero/iocdi/XmlConfig.java#L6). Esta clase así mismo sostiene una lógica de negocio simple:
- Obtiene los beans de cada equipo (computadora, servidor) e imprime sus hostname en la consola
- Obtiene el bean de la red y establece un ping de comunicación entre los equipos

{% gist 4611b85d1fdaebaf5d89e92ae2d5cc85 %}


#### Wiring
Para hacerlo más divertido dividiremos las definiciones de la metada (osea dividimos en varios Xmls). 

#### contexto-1.xml
Tenemos [contexto-1.xml](https://github.com/sistecma/spring-desde-cero/blob/master/app/ioc-di-xml/src/main/resources/contexto-1.xml) que esta en el classpath y tiene las definiciones de los beans de tipo ComputadoraDI y con id computadoraRef (identificador único entre todos los Xml del proyecto). El bean computadoraRef tiene 3 propiedades entre ellas la referencia del bean monitor (definido en contexto-3.xml) y 2 valores (ip y hostname) dichas dependencias son pasadas vía inyección a su constructor.

{% gist 2fdcccc639275fc52a47f8fe59bfb3d9 %}

El contexto-1.xml también tiene la definición del bean servidorRef que sostiene la instancia de la clase Servidor. Este tiene como propiedades ip y hostname. Así mismo el contenedor es quien ejecuta el método boot (observar el tag init-method) al momento de post-construir el bean computadoraRef. Invoca el método shutdown (observar el tag destroy-method) al cerrarse el contenedor. Cuándo cierras el contenedor? [Cuando se ejecuta este fragmento en XmlConfig.java](https://github.com/sistecma/spring-desde-cero/blob/a4e5948dbda5336f49d5390bb31b7d568d294f65/app/ioc-di-xml/src/main/java/com/sistecma/springdesdecero/iocdi/XmlConfig.java#L33)

{% gist 0e200d641d6b5de76c940b9a2374da65 %}


#### contexto-2.xml
El [contexto-2.xml](https://github.com/sistecma/spring-desde-cero/blob/master/app/ioc-di-xml/src/main/resources/contexto-2.xml) tiene la definición del bean con id red que tiene la instancia de la clase Red y tiene como dependencias las referencias de los beans computadora y servidor (estos son exactamente los nombres de las propiedades en el código Java definido en Red.java). Dichos beans estan referenciados en el archivo contexto-1.xml

{% gist d042fe25b01a9fa6d2fba3280a739e40 %}

#### contexto-3.xml
El [contexto-3.xml](https://github.com/sistecma/spring-desde-cero/blob/master/app/ioc-di-xml/src/main/resources/contexto-3.xml) contiene la definición del Monitor. Dicho bean esta referenciado en contexto-1.xml

{% gist af909cb7c6c9fdd2fe31ec3b75eefd5c %}


#### Conclusiones

En este breve artículo, hemos discutido: 
* Los aspectos básicos sobre como funciona el contenedor de Spring y su uso con Xml. 
* Revisamos como realizar inyección por constructor, e inyección por método. 
* Revisamos ciertos mecanismos sobre como organizar nuestra metadata en diversos Xml y como entrelazarlos e invocarlos en código.

Para una comprensión completa revisar el código fuente incluido en este [repositorio](https://github.com/sistecma/spring-desde-cero), junto con las instrucciones de ejecución.         

