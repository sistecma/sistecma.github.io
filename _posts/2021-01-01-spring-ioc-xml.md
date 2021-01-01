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

#### Tipos de Inyección
Cuando configuras beans vía Xml lo común es que inyectes tus dependencias vía método y constructor. 

* En la inyección basada en métodos, proporcionamos las dependencias requeridas como parámetros de campo/atributos/propiedades para la clase y los valores se establecen utilizando los métodos de setXX de los mismos.
* En la inyección basada en constructor, proporcionamos las dependencias requeridas al constructor de la clase dependiente.

En la práctica, preferir la inyección vía constructor tiene sus ventajas. Puesto que creamos el objeto llamando a su constructor. Si el constructor espera todas las dependencias requeridas como parámetros, entonces podemos estar 100% seguros de que la clase nunca será instanciada sin sus dependencias inyectadas. El contenedor de IoC se asegura de que todos los argumentos proporcionados en el constructor estén disponibles antes de pasarlos al constructor. Esto ayuda a prevenir la infame NullPointerException.

Finalmente, La inyección por constructor es extremadamente útil ya que no tenemos que escribir la lógica de negocios separada en todas partes para verificar si todas las dependencias requeridas están cargadas, simplificando así la complejidad del código.

#### Organización de la metadata
Los Xml de configuración pueden estar alojados en un solo archivo o divididos en varios archivos, para mejor organización. Finalmente todo es cargado en el contenedor de Spring como si fuera una solo archivo. Como se unen? lo puedes hacer vía el [ClassPathXmlApplicationContext](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/support/ClassPathXmlApplicationContext.html) simplemente pasas como parámetros los archivos Xmls a su constructor, obviamente dichos archivos deben estar visibles en tu classpath.

El otro modo recomendado es vía imports directamente dentro de los Xmls. Así mismo recomiendo todos los Xml visibles en tu classpath.  

#### El mini-proyecto de juguete
Para entender como funciona todo esto, vamos a revisar los conceptos alrededor de un ejemplo básico. Vamos a asumir que tenemos el siguiente proyecto de juguete:
* Queremos simular la comunicación entre una computadora y un servidor.
* La computadora se puede encender y apagar.
* El servidor asumimos que esta siempre encendido.
* Ambos equipos tienen su propio [hostname](https://es.wikipedia.org/wiki/Hostname) y pueden hacer [ping](https://es.wikipedia.org/wiki/Ping)
* La computadora tiene un monitor
* El monitor solo puede "mostrar" información.
* Existe una red entre la computadora y el servidor que se puede configurar y destruir; asi como puede establer ping para comunicar los equipos.

**Nota-** A fín de mostrar las carácteristicas más importantes de Spring con Xml verás en el código que he sobre-diseñado en algunas partes y esta bien, todo esto es con fines didácticos y demostrativos de las capacidades de Spring.

#### Diseño
* Vamos a usar Spring con Xml metadata. 
* Tenemos una interfaz que contiene el contrato de un [Equipo](https://github.com/sistecma/spring-desde-cero/blob/1fcdf6dc9bd1319b3a8fbf401c638ad25aad9ea3/app/ioc-di-xml/src/main/java/com/sistecma/springdesdecero/iocdi/Equipo.java#L3)

{% gist 20c28f1701ef8aca69b4da47281e699 %}

* Tenemos una interfaz que contiene el contrato de un [Monitor](https://github.com/sistecma/spring-desde-cero/blob/1fcdf6dc9bd1319b3a8fbf401c638ad25aad9ea3/app/ioc-di-xml/src/main/java/com/sistecma/springdesdecero/iocdi/Monitor.java#L3). Porqué una interfaz y no directamente una clase?. Pueden haber diversos tipos de monitores. Adicionalmente quiero hacer evidente uno de los puntos fuertes de Spring que tiene que ver con el desacople y como elegir en configuración la implementación.

{% gist 0b1fa370c7cb3d764733af050d849e1c %}

* Tenemos una implementación de Monitor llamada [MonitorImpl](https://github.com/sistecma/spring-desde-cero/blob/1fcdf6dc9bd1319b3a8fbf401c638ad25aad9ea3/app/ioc-di-xml/src/main/java/com/sistecma/springdesdecero/iocdi/MonitorImpl.java#L4)  

{% gist c2a80788eb40c5349e909230e6b3fce2 %}

* Tenemos una clase [Computadora](https://github.com/sistecma/spring-desde-cero/blob/1fcdf6dc9bd1319b3a8fbf401c638ad25aad9ea3/app/ioc-di-xml/src/main/java/com/sistecma/springdesdecero/iocdi/ComputadoraDI.java#L3) que implementa el contrato de Equipo. Tiene como dependencias a dependencias a Monitor, su hostname, e ip. A esta clase le vamos inyectar estas dependencias vía su constructor. 

{% gist 074047535120f980942095baaa8471f3 %}

* Tenemos una clase [Servidor](https://github.com/sistecma/spring-desde-cero/blob/1fcdf6dc9bd1319b3a8fbf401c638ad25aad9ea3/app/ioc-di-xml/src/main/java/com/sistecma/springdesdecero/iocdi/Servidor.java#L4) que implementa el contrato de Equipo. Tiene como dependencias hostname e ip estos atributos serán inyectados vía setters (usando sus métodos setXX).

{% gist cd8b3675ef88f19fdfc3a2993f9cee6a %}

* Tenemos una clase llamada [Red](https://github.com/sistecma/spring-desde-cero/blob/a4e5948dbda5336f49d5390bb31b7d568d294f65/app/ioc-di-xml/src/main/java/com/sistecma/springdesdecero/iocdi/Red.java#L4) que tiene como dependencia a los equipos Computadora y Servidor. Así mismo la inyección será vía métodos. 

{% gist 52bf7b9f9f4eb0a938811a5a3e741280 %}

* La clase que arranca al contenedor Spring la llamaremos: [XmlConfig.java](https://github.com/sistecma/spring-desde-cero/blob/1fcdf6dc9bd1319b3a8fbf401c638ad25aad9ea3/app/ioc-di-xml/src/main/java/com/sistecma/springdesdecero/iocdi/XmlConfig.java#L6)

{% gist 4611b85d1fdaebaf5d89e92ae2d5cc85 %}


#### Wiring
Para hacerlo más divertido dividiremos las definiciones de la metada (recuerdas que es con Xml?) en varias partes. 

#### contexto-1.xml
Tenemos [contexto-1.xml](https://github.com/sistecma/spring-desde-cero/blob/master/app/ioc-di-xml/src/main/resources/contexto-1.xml) que esta en el classpath y tiene las definiciones de los beans de tipo ComputadoraDI y con id computadoraRef (identificador único entre todos los Xml del proyecto). El bean computadoraRef tiene 3 propiedades entre ellas la referencia del bean monitor y 2 valores (ip y hostname) dichas dependencias son pasadas vía inyección a su constructor.

El contexto-1.xml también tiene la definición del bean servidorRef que sostiene la instancia de la clase Servidor. Este tiene como propiedades ip y hostname.

{% gist 2fdcccc639275fc52a47f8fe59bfb3d9 %}

#### contexto-2.xml
El [contexto-2.xml](https://github.com/sistecma/spring-desde-cero/blob/master/app/ioc-di-xml/src/main/resources/contexto-2.xml) tiene la definición del bean con id red que tiene la instancia de la clase Red y tiene como dependencias las referencias de los beans computadora y servidor (estos exactamente los nombres de las propiedades en el código Java definido en Red.java). Dichos beans estan referenciados en el archivo contexto-1.xml

{% gist d042fe25b01a9fa6d2fba3280a739e40 %}

#### contexto-3.xml
El [contexto-3.xml](https://github.com/sistecma/spring-desde-cero/blob/master/app/ioc-di-xml/src/main/resources/contexto-3.xml) contiene la definición del Monitor. 

{% gist af909cb7c6c9fdd2fe31ec3b75eefd5c %}


#### Conclusiones

En este breve artículo, hemos discutido los aspectos básicos sobre como funciona el contenedor de Spring y su uso con Xml. También revisamos como realizar inyección por constructor, e inyección por método. Finalmente revisamos ciertos mecanismos sobre como organizar nuestra metadata.

Para una comprensión completa revisar el código fuente incluido en este [repositorio](https://github.com/sistecma/spring-desde-cero).         

