---
layout: post
title: Workflows hijos en Temporal
image: 
description: Workflows que invocan a otros workflows
author: Hernán Moreno
disqus: true
---

El concepto de workflows hijos (child workflows de ahora en adelante), existe como complemento al concepto de workflow en Temporal. Recordemos, que para [Temporal](https://sistecma.github.io/2021/02/04/aplicaciones-invencibles-con-temporal.html) un workflow es simplemente un método o función que maneja estado (es stateful), es durable y tolerante a fallas. El concepto de child workflows nos permite "dividir" lógica de negocio, de tal manera que workflows puedan invocar otros workflows, cuando necesitemos:

* Modularizar una lógica compleja, desacomplándola y dividiéndola en partes.

* Ejecutar código remoto de manera confiable(ejemplo invocar cierto subsistema que se ejecute en el cloud). Recordemos que Temporal garantiza dicha confiabilidad mediante el uso de workflows.

* Invocar múltiples workflows. Un workflow padre (de ahora en adelante parent workflow) puede invocar muchos child workflows.

* Temporal siempre guarda un historial de los eventos que ocurren en sus workflows. Al dividir lógica en sus child workflows, cada child workflow tendrá su historial en su alcance, y el magnitud del historial del padre se reduce. 

* Asociaciones uno-a-uno entre un workflow y un recurso.

Existe la siguiente consideración al momento de considerar usar child workflows:

* Son útiles cuando **no** se requiere una lógica tan acoplada entre el parent workflow y su child workflow. Esto debido a que el estado no es compartido entre ellos. Solo existe comunicación asíncrona vía [signals](https://sistecma.github.io/2021/04/27/workflows-larga-duracion.html) 

El modo de invocación de un child workflow es bastante similar a como se hace con la invocación de una [actividad](https://sistecma.github.io/2021/02/04/aplicaciones-invencibles-con-temporal.html). Como siempre, para ilustrar este nuevo concepto prefiero hacerlo a través de un simple ejemplo, y por simplicidad usaré Golang. En este caso será solamente un parent workflow que invoca a un único child workflow. El child workflow solo devuelve como valor un string indicando que ha recibido el mensaje del parent workflow.

* Definimos los workflows

{% gist 2dfd186b461e9e0d23d28bd13754519f %}

Como notamos, la invocación es bastante parecida a la invocación de una actividad a excepción que tenemos que configurar opciones para child workflow en lugar de opciones para una actividad y que creamos un contexto que considera dicha opción.

{% gist aab53ec50ff688c4e7cd4378698e2759 %}

* Creamos el worker

{% gist 1925fc5e0b62eac26d11960e502f11a0 %}

* Creamos el starter

{% gist 447c4114dbae0fb345ed976499d155fe %}

* Despliegue y Ejecución

Descarga el proyecto desde este [sitio](https://github.com/sistecma/temporalio/tree/main/app/go/child)

Para ejecutar este ejemplo:

1- El servicio de Temporal tiene que estar ejecutandose.

2- Ejecutar el worker

{% gist 5fdb7d50063af44fcec5ec42fff76374 %}

3- Ejecutar el starter

{% gist afdea89fa6312bd56da7c8df13c5750e %}


#### Conclusión

En este artículo, discutimos como llamar workflows desde otro workflow, cuando aplicarlo y sus restricciones.  

Para una comprensión completa revisar el código fuente incluido en este [repositorio](https://github.com/sistecma/temporalio/tree/main/app/go), junto con las instrucciones de ejecución.       

