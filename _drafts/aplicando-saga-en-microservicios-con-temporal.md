---
layout: post
title: Aplicando el patrón SAGA en tus microservicios construidos con Temporal.
image: 
description: Transacciones distribuidas con SAGA y Temporal.
author: Hernán Moreno
---

Uno de los mayores retos al construir microservicios es la gestión de transacciones distribuidas. Por el hecho de ser independientes y de implementar partes de la aplicación por microservicio como resultado introducimos más puntos de falla que no ocurrirían si la misma aplicación fuera monolítica. Problemas típicos de los microservicios tales como latencia de la red, fallas en cualquiera de los hosts donde reciden o sobrecarga en cualquier de los microservicios que componen una transacción distribuida pueden afectar la consistencia y rendimiento de una aplicación o sistema.

Debido a lo anteriormente mencionado, los microservicios no solo deben implementar la lógica de negocio si no, todo el ecosistema debe estar configurado de tal modo que maneje aspectos como:

Reintentos- Mecanismos que permiten repetir las llamadas a microservicios, cuando existen fallas de red o en el microservicio llamado.

Contrapresión- Mecanismos que limitan la carga en los microservicios.

Persistencia- Gestión del estado de los procesos (workflows).

Descubrimiento de servicios- Que permitan detectar los servicios componentes, en lo posible abstrayendo detalles de ips y puertos que dificulten el mantenimiento del sistema.

Encolamiento- Que permite desacoplar los servicios y hacerlos más resilientes.

Balanceo de carga- A fín de que el sistema se ejecute con rendimiento óptimo.

La plataforma Temporal, abstrae todos estos aspectos y permite al desarrollador enforcarse en la lógica de negocio reflejada en Workflows o flujos estructurados de interacción entre entidades que pueden ser microservicios. Temporal garantiza la consistencia y resiliencia del estado de los workflows implementados en ella (ya sean en python, java, golang,php, rust, etc) y los microservicios con su lógica son implementados facilmente y reflejados a través de funciones o métodos (ya sea en python, java, golang, php, rust, etc). Si todavía no has leído previamente sobre Temporal te recomiendo revisar siguiente link: [https://sistecma.github.io/2021/02/04/aplicaciones-invencibles-con-temporal.html](https://sistecma.github.io/2021/02/04/aplicaciones-invencibles-con-temporal.html)

SAGA es un patrón que nos ayuda a mantener la integridad/consistencia de datos en servicios distribuidos. Una transacción en SAGA se considera exitosa cuando todos los pasos (transacciones individuales) de sus componentes son exitosos. Si alguno de los pasos falla el sistema puede reversar el estado de la aplicación a su estado original. 

Temporal es un sólido candidato para la implementación de SAGA con cualquiera de sus SDKs (java, golang, etc). A continuación ejemplificamos un caso típico, de implementación de SAGA usando Temporal.


#### Aplicación para gestión de reservaciones de hotel, vuelo, y vehículo para los viajeros.  

La implementación la realizaremos con Golang y Temporal. 

Asumiremos 4 microservicios: 

* Hotel: Microservicio encargado de implementar la lógica de negocio referente al agendamiento de hotel (desde el punto de vista de Temporal lo representaremos con actividades)

* Vuelo: Microservicio encargado de implementar la lógica de negocio referente al agendamiento de vuelos (desde el punto de vista de Temporal lo representaremos con actividades)

* Vehículo: Microservicio encargado de implementar la lógica de negocio referente al agendamiento de transporte terrestre desde el aeropuerto al hotel (desde el punto de vista de Temporal lo representaremos con actividades)

* Viaje: Microservicio orquestador (desde el punto de vista de Temporal lo representaremos con un workflow)

![Arquitectura del sistema de viaje](/assets/images/viaje.png)

En nuestro ejemplo, podemos considerar como exitoso un agendamiento de viaje cuando: el agendamiento de hotel sea exitoso, el agendamiento del vuelo sea exitoso, el agendamiento de vehiculo sea exitoso. Caso contrario no se puede realizar el viaje y la transacción debe ser reversada para aquellos pasos intermedios que fueron exitosos.  

Temporal no impone un modo o estilo para codificar nuestros microservicios sin embargo, una buena práctica es considerar la equivalencia entre servicios (punto de vista de arquitectura de microservicios) con actividades (punto de vista de Temporal) y el proceso servidor (punto de vista de arquitectura de microservicios) con el worker (punto de vista de Temporal). Así, para el caso del microservicio Hotel, lo codificamos dentro de un simple archivo en go en donde las funciones reservar y cancelar son las actividades, mientras que la función main implementa el worker y registra las actividades que serán expuestas vía una cola con un nombre específico. El mismo patrón aplica para los microservicios Vehículo y Vuelo.

{% 7edc344a3991449b03ec81c327eefc2a %}

Dependiendo de la complejidad de las aplicaciones pueden existir uno o muchos workflows. Para nuestro ejemplo solo estamos considerando un solo workflow que funciona de tipo petición-respuesta e implementa la SAGA. Siguiendo el mismo patrón que los anteriores microservicios, función reservar implementa el workflow mientras que main implementa el worker todo esto encapsulado dentro de un simple archivo en go.

{% 7fd9b0a835e3e384eb93a9f0abd9fe30 %}

Una tema a destacar es que cada microservicio tiene una cola única que esta asociada a su respectivo worker y actividades o worfklow. Y esto nos sirve para hacer los enrutamientos. Notar que hemos usado simples strings, pero pueden ser enrutados con interfaces.

{% db259b4405d7d5a9e3f612870797138a %}

Finalmente, usamos un starter para invocar e iniciar el workflow. Notar que pudiera ser implementado con un http server sin embargo por simplicidad lo dejamos en un código simple que invoque dicho workflow.

{%4d395839727ace21ddad188873e1ebd8 %}

Temporal hace que esta aplicación tan simple tenga todos los beneficios de los microservicios, sea escalable, extremadamente resiliente y practicamente elimina los inconvenientes que se dan al adoptar una arquitectura de microservicios.

Si ejecutamos (para detalles de ejecución hacer [click aquí](https://github.com/sistecma/temporalio/blob/main/app/go/saga/README.md)) el ejemplo veremos algo como lo mostrado a continuación:

* Para el microservicio de Hotel

![Workflow completado. Microservicio Hotel](/assets/images/saga/ok/hotel.png)

* Para el microservicio de Vehiculo

![Workflow completado. Microservicio Vehiculo](/assets/images/saga/ok/vehiculo.png)

* Para el microservicio de Vuelo

![Workflow completado. Microservicio Vuelo](/assets/images/saga/ok/vuelo.png)

* Para el microservicio de Viaje (el que tiene la implementación de SAGA y orquestador)

![Workflow completado. Microservicio Viaje](/assets/images/saga/ok/viaje.png)

* Starter. Como podemos observar muestra como resultado: [5577006791947779410 5577006791947779410 5577006791947779410] que son los ids de transacción exitosa de hotel, vehículo, vuelo respectivamente.

![Workflow completado. Starter](/assets/images/saga/ok/starter.png)

Temporal tiene un componente Web UI que permite ciertas opciones administrativas respecto a los workflows ejecutandose en el ambiente de Temporal. Por defecto corre (para la versión docker) en: http://localhost:8088. A continuación podemos ver el resultado de nuestra ejecutación:

![Workflow completado. Web UI General](/assets/images/saga/ok/web-general.png)

Si queremos con un poco más detalle, podemos hacer click sobre el id del workflow:

![Workflow completado. Web UI Detalle](/assets/images/saga/ok/web-detalle.png)

Podemos también comprobar en nuestro demo que se cumple la SAGA cuando bajamos cualquiera de los microservicios. Por ejemplo si bajamos el microservicio vuelo, después del timeout preconfigurado veremos:

* Para el microservicio de Hotel

![Workflow fallido. Microservicio Hotel](/assets/images/saga/error/hotel.png)

* Para el microservicio de Vehiculo

![Workflow fallido. Microservicio Vehiculo](/assets/images/saga/error/vehiculo.png)

* Para el microservicio de Viaje (el que tiene la implementación de SAGA y orquestador)

![Workflow fallido. Microservicio Viaje](/assets/images/saga/error/viaje.png)

* Starter. Como podemos observar muestra como resultado: No es posible obtener resultado del workflow workflow execution error (type: viaje.reservar, workflowID: viaje_workflowID, runID: 928df882-25d2-44a9-89a5-ac95010791dd): activity error (type: vuelo.reservar, scheduledEventID: 17, startedEventID: 0, identity: ): activity timeout (type: ScheduleToStart)

![Workflow fallido. Starter](/assets/images/saga/error/starter.png)

Si observamos de nuevo en el Web UI, notamos:

![Workflow fallido. Web UI General](/assets/images/saga/error/web-general.png)

En el detalle, notamos:

![Workflow fallido. Web UI Detalle](/assets/images/saga/error/web-detalle.png)

Si hacemos click en History (menú principal del Web UI):

![Workflow fallido. Web UI History](/assets/images/saga/error/web-history.png)

#### Conclusión
En este artículo, discutimos la implementación del patrón SAGA con Temporal. Observamos la simplicidad para la construcción de microservicios, los detalles de las configuraciones, la validación de los resultados de ejecución del demo reconfirmando que SAGA funciona correctamente, y la visualización del estatus de los workflows mediante la Web UI.  

Para una comprensión completa revisar el código fuente incluido en este [repositorio](https://github.com/sistecma/temporalio/tree/main/app/go/saga), junto con las instrucciones de ejecución.       

**Anterior post:** [Construye aplicaciones invencibles con Temporal](https://sistecma.github.io/2021/02/04/aplicaciones-invencibles-con-temporal.html)

