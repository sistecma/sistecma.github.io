---
layout: post
title: Alcance de Spring Beans
description: Breve introducción al uso de scope/alcance en Spring Beans.
image: 
author: Hernán Moreno
---

Spring Framework provee un mecanismo llamado scope o alcance de su ciclo de vida (en español), para todos los beans definidos en su contenedor. Posee 6 tipos de alcance pre-concebidos. 

Nota- En este artículo abordaré solo configuración vía Xml, configuración vía anotaciones la abordaré en otro artículo como parte de anotaciones en Java con Spring.

#### Para cualquier tipo de aplicación
* Singleton (Este es el alcance por defecto): Spring crea un solo bean por contenedor.
* Prototipo: Es lo opuesto a Singleton, Spring crea un bean cada vez que este es solicitado.

#### Para web
* Aplicación: Es una sola instancia durante todo el ciclo de vida del ServletContext.
* Request: Es una sola instancia durante todo el request de HTTP.
* Sesión: Es una sola instancia durante la sesión de HTTP.
* Websocket: Es una sola instancia durante el ciclo de vida del websocket.

Nota- Asegúrate de haber hecho una configuración apropiada a tu aplicación web basada en Spring/Spring MVC antes de usarlos. En próximos tutoriales explicaré a fondo el uso de estos beans como parte de Spring Web/MVC y Boot.

#### Singleton
Singleton es el alcance por defecto en el contenedor de Spring. Le dice al contenedor crear y manejar solamente una instancia del bean, por contenedor. Esta única instancia es almacenada en el cache de los beans tipo singletons. Todos los subsecuentes invocaciones y referencias para ese específico nombre de bean retornarán la instancia en cache. En general es útil para beans que no manejan estado (stateless beans)

{% gist 14c89a28c5ea830feefdaf065a2528a4 %}

#### Prototipo
Con prototipo, por cada petición un nuevo bean es creado por código de la aplicación. En general es útil para beans que manejan estado (stateful beans).

{% gist f48f01d0988af7528d51e0d14db37267 %}

#### Request
Con request, Spring crea una nueva instancia para cada HTTP request. Por ejemplo: Imagina que el servidor esta manejando 100 requests de HTTP, esto implicaría que en el contenedor tiene al menos 100 instancias del bean con alcance request. Cada cambio en el estado de una instancia, aplicará solo a dicha instancia, las demás no se verán afectadas. Todas estas instancias son destruídas tan pronto el request sea completado.

{% gist 9c9382bf0906d8bc52492e32dcf461d8 %}

#### Sesión
Con sesión, Spring crea una nueva instancia para cada HTTP session. Por ejemplo: Imagina que el servidor esta manejando 50 sesiones activas, esto implicaría que tenemos al menos 50 instancias del bean de tipo sesión.  Cada cambio en el estado de una instancia, aplicará solo a dicha instancia, las demás no se verán afectadas. Todas estas instancias son destruídas tan pronto la sesión haya terminado.

{% gist 986821cf44f96f9c6c1c7578b8ded17a %}

#### Aplicación
Con el alcance x aplicación, Spring crea una instancia por runtime/ejecución de aplicación web. Similar al singleton pero asociado al ServletContext. 

{% gist 59feb89d340001ada16aa41dcaace6e5 %}

#### Websocket
El protocolo de Websocket permite una comunicación de 2 vías entre cliente y host. A continuación la configuración:

{% gist d7a5cf84ce1ad56df5355ac0caeee152 %}

#### Alcance personalizado: Thread
Spring también provee un alcance que no esta disponible por defecto y esta asociado a alcance x thread. Osea el contenedor crea una sola instancia por thread. A continuación la configuración:

{% gist ea79c18c2c4c5b8e7c51d3880f6ef674 %}

#### Conclusiones
Spring nos dá 6 tipos de alcance de beans con diferentes ciclos de vida x cada alcance, que pueden ser configurados de manera sencilla. Como siempre te invito a que revises nuestro [repositorio](https://github.com/sistecma/spring-desde-cero) para un mejor comprensión con ejemplos. 

**Anterior post:** [Spring IoC con Xml](https://sistecma.github.io/2021/01/01/spring-io-xml.html)