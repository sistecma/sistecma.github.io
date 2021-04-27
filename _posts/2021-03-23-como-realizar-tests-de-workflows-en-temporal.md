---
layout: post
title: ¿Cómo realizar tests de tus workflows en Temporal?
image: 
description: Unit Tests con Temporal.
author: Hernán Moreno
---

Autor: Hernán Moreno

Una de las grandes ventajas de hacer workflows en código es que puedes realizar tests de estos. Lo cual es muy complicado de hacer para otros orquestadores como Airflow por ejemplo. Con el poder de tener workflows como código tus unit tests pueden tener la complejidad que desees con las mismas herramientas estándar que conoces como desarrollador. 

Temporal, íncorpora para cada uno de sus SDKs un test suite que nos permite ponernos manos a la obra con nuestros tests. Es de nuestro estilo explicarlo con ejemplos para esto tomemos el típico hola mundo que realizamos con Temporal tiempo atrás. 

Recordando, el ejemplo tiene solo una actividad expresada a través de la función Activity y el workflow expresado a través de la función Workflow. El workflow lo único que hace es invokar a la actividad y pasar los parámetros que necesita. 

Si deseas revisar el paso a paso sobre como construir este ejemplo, te lo dejo en este [link](https://sistecma.github.io/2021/02/04/aplicaciones-invencibles-con-temporal.html)

{% gist 96bc2a9f8ebec215170e7bc7d598bfc3 %}

El unit test es muy simple:

{% gist f8b61db0984678f9bfd468dca9e4d214 %}

Y el resultado:

{% gist f697ade2f68cab6cd10e7a4b21779d64 %}

Como siempre, el código lo puedes descargar en nuestro [repo](https://github.com/sistecma/temporalio/tree/main/app/go/hola)

#### Conclusión

En este artículo aprendimos como realizar unit tests para probar nuestros workflows con Temporal.

**Anterior post:** [Aplicando SAGA en Microservicios con Temporal](https://sistecma.github.io/2021/03/04/aplicando-saga-en-microservicios-con-temporal.html), **Siguiente post:** [Workflows de larga duración](https://sistecma.github.io/2021/04/27/workflows-larga-duracion.html)
