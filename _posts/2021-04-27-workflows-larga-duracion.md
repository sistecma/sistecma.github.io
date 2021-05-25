---
layout: post
title: Workflows de larga duración
image: 
description: Transacciones que van más allá de la típica solicitud/respuesta (request/reply).
author: Hernán Moreno
---

Autor: Hernán Moreno

Existen muchos casos de uso en los que muchos actores están involucrados y cuyas transacciones son de larga duración, pudiendo durar minutos, horas o incluso semanas. Por ejemplo el proceso de activación de tu plan celular. Típicamente no es instantaneo dependendiendo del país, regulaciones y del operador dicha activación demorará minutos u horas inclusive de un día para otro en muchos casos. Así mismo, la activación como tal debe pasar por ciertos estados: creación del cliente la plataforma de negocios, activación de los servicios en la red celular, etc.

Los workflows de larga duración permiten implementar dichos casos de uso de una manera casi trivial. [Temporal](https://temporal.io/) es nuestra plataforma favorita para implementar sistemas distribuidos, altamente escalables y resilientes. Temporal esta super dotado para gestionar este tipo de casos de uso. 

En Temporal los workflows pueden correr por microsegundos, segundos, minutos, horas, días, meses o incluso años. Son insensibles a caidas de servicios de red o del mismo servicio de Temporal puesto que el estado es guardado consistentemente. Adicionalmente si un workflow esta en espera (por algún tipo de evento), Temporal puede gestionar su óptimo uso de recursos. 

Cuando un workflow tiene estas caracteristicas, Temporal provee 2 mecanismos para interactuar con el workflow, que son las señales(signals) y consultas (queries).

#### Signals

Proveen un mecanismo para enviar información asíncrona directamente a un workflow que se está ejecutando. Recordemos por posts anteriores que Temporal regularmente puede pasar información al inicio de ejecución de un workflow puesto que un workflow es simplemente una función o método implementado en cualquiera de los SDKs disponibles, ya sea Golang, Java, o PHP. Normalmente cada workflow contiene cero o muchas actividades(recuerda que una actividad es un concepto de Temporal, puedes aprender de ello en posts anteriores) para implementar cierta lógica de negocio. Por diseño en este tipo de procedimiento Temporal usa "polling" para ejecución de las actividades y esto significa que la data asociada Temporal la almacena en una tercera parte (base de datos) hasta que sea ejecutada por la actividad.

Cuando un signal es recibido por un workflow que se esta ejecutando, Temporal persiste inmediatamente el evento en la historia del workflow. El workflow entonces puede procesar el signal sin riesgo a perder la información. Una contraparte a clarificar es que las signals no retornan valores solo permiten enviar data al workflow. 

#### Query

Usamos queries para obtener información del workflow. La dupla signal(actualizar workflow) y query(obtener info del workflow) nos permiten realizar transacciones seguras.
Query permite exponer el estado interno del workflow hacia el mundo en un modo síncrono. Los queries son read-only (leer-solamente), no pueden afectar el estado interno del workflow.


Como siempre explicamos los conceptos con un ejemplo, en este caso hecho en Java. 

* Workflow para Billetera básica

Asumamos un ejemplo simple. Una billetera electrónica que tiene las operaciones: crear balance inicial, poner valores , quitar valores, consultar balance.

En Java tenemos que definir el workflow en una interfaz:

{% gist f5668728f9bc51344924bae9580bfa52 %}

En Java con Temporal, solo puedes tener un @WorkflowMethod por @WorkflowInterface, sin embargo en la misma definición puedes tener varios @SignalMethod y @QueryMethod.
La implementación del workflow es la siguiente:

{% gist 955a6182b62244d33e10e17e16e537c4 %}

Recuerda, que necesitamos de un mecanismo que ejecute o inicie el workflow, en términos de Temporal corresponde a un starter. Así mismo y por brevedad pondremos en el mismo starter invocaciones a signals y queries para incrementar el balance inicial y luego decrementarlo. Al final para cerrar el workflow usamos el signal exit.

{% gist 9a60f2ec044e1a547d52f0bcfb231fa9 %}

Para la ejecución:

* Si no sabes todavía qué es Temporal y como usarlo, revisa este [post](https://sistecma.github.io/2021/02/04/aplicaciones-invencibles-con-temporal.html) y descargalo.

* Sube el servicio de Temporal, en mi caso lo uso facilmente con docker (ver el punto anterior)

* Descarga el ejemplo de esta ruta [https://github.com/sistecma/temporalio](https://github.com/sistecma/temporalio)

* Busca el directorio app/java/wallet desde el directorio principal donde se descargó

* Y luego, podemos ejecutar el ejemplo facilmente vía maven.

{% gist 3409d6da32035df52d4a49b4f61bbd40 %}

#### Conclusión

En este artículo, discutimos la implementación de mecanismos de actualización y consulta de workflows de larga duración y que están ejecutándose. Observamos la simplicidad del proceso.  

Para una comprensión completa revisar el código fuente incluido en este [repositorio](https://github.com/sistecma/temporalio/tree/main/app/java), junto con las instrucciones de ejecución.       

**Anterior post:** [¿Cómo realizar tests de tus workflows en Temporal?](https://sistecma.github.io/2021/03/23/como-realizar-tests-de-workflows-en-temporal.html), **Siguiente post:** [Child workflows](https://sistecma.github.io/2021/05/24/child-workflows.html)
