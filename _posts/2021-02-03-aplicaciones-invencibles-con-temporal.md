---
layout: post
title: Construye aplicaciones invencibles con Temporal. Una nueva forma de orquestar tus microservicios.
image: 
description: Breve introducción a Temporal.
author: Hernán Moreno
---

#### El problema

La tecnología esta en constante evolución. Tenemos continuas innovaciones y cambios de paradigma. No hace mucho nuestros sistemas los concebíamos como monolíticos. Grandes o pequeñas aplicaciones corrían en ambientes autosuficientes como un todo. Luego comenzamos a dividir, aplicaciones por un lado, con balanceo de cargas, redundancia, etc y bases de datos por otro lado, en clusters, etc. Y progresivamente continuamos subdividiendo los sistemas hasta llegar a los microservicios. 

Con la aparición del paradigma de los microservicios. Hubieron mayores ventajas para las aplicaciones, tales como: libertad para que un mismo sistema pueda ser implementado en diversos lenguajes y bases de datos, cada microservicio cumple con una sola responsabilidad orientada al negocio, despliegues independientes, pueden ser desarrollados en paralelo, facilidades para ser probados, etc. 

Sin embargo, los microservicios también introdujeron desventajas, entre ellas: Complejidad aumentada, demoras en las llamadas remotas, incremento en configuraciones, mayor complejidad en mantener la seguridad, mayor dificultad al mantener el estado de las transacciones.

La confiabilidad de las aplicaciones se volvió aún más crítica con los microservicios, ahora los desarrolladores tienen que enfocarse mucho más en establecer mecanismos a fín de que sus aplicaciones continúen siendo confiables. El uso de colas, caches, un mayor uso de bases de datos, y temporizadores se ha vuelto común en la era de los microservicios. Esto a su vez incrementa aún más la complejidad, y los puntos sencibles a fallas.

#### Bienvenidos a Temporal

Temporal es una plataforma open-source, que surgió para terminar gran parte de los problemas actuales relacionados a la confiabilidad de sistemas distribuidos, reduciendo la carga para los desarrolladores puesto que maneja toda la complejidad asociada con los sistemas distribuidos. Es altamente recomendada por gigantes de la industria como: Hashicorp, Box, Uber (con la versión anterior a Temporal, Cadence), Coinbase, Checkr, entre otros. 

Temporal consiste de un framework de programación (SDK, que soporta actualmente Go y Java. Muy pronto soportará Ruby,Python, Node, C#/.NET, Swift, Haskell, Rust, C++ y PHP) y un backend. La abstracción principal en Temporal es un flujo de trabajo ( de ahora en adelante workflow) con manejo del estado a prueba de fallas con una lógica de negocio expresada como código. El estado del workflow es código, incluyendo las variables locales, e hilos que crea. Es inmune inclusive a fallas del mismo servicio y procesos de Temporal. Esto permite escribir código utilizando toda la potencia de un lenguaje de programación, mientras que Temporal se encarga de la durabilidad, disponibilidad y escalabilidad de la aplicación.

Temporal encaja perfectamente para una gran cantidad de casos de uso, en general aquellos que van más allá de una sola solicitud-respuesta, requieren el seguimiento de un estado complejo, responden a eventos asincrónicos y se comunican con dependencias externas no confiables, temporizadores que necesitemos se disparen en segundos, días, semanas, meses o inclusive años.

El servicio de backend no tiene estado y se basa en un almacenamiento persistente. Actualmente, usa como almacenamiento persistente a Cassandra, Postgress o MySQL. Se puede agregar un adaptador a cualquier otra base de datos que proporcione transacciones multi-row con simple sharding. 

#### Topología de una aplicación en Temporal

Los componentes principales de cualquier aplicación en Temporal son: iniciadores (de ahora en adelante llamados starters), workflows, actividades, trabajadores (de ahora en adelante llamados workers) estos son controlados y desarrollados por el cliente. Temporal provee los mecanismos para que dicho código mediante el SDK trabaje con el servicio de backend y el almacenamiento permanente.

Debido a que Temporal es acerca de implementar workflows como código, la lógica de estos es expresada en cualquiera de los lenguajes disponibles. La aplicación cliente de temporal, es decir la lógica de negocio que el desarrollador implemente será expresada en términos de los componentes principales. Cualquier aplicación típica que interactue con el backend de Temporal tendrá: starters, workflows, actividades, y workers expresados ya sea en Java o Go.

* Un starter, inicia el workflow. Es el punto común que arranca toda la lógica de negocio expresada en código. 

* Un workflow, no es otra cosa que cualquier fragmento de código (lógica de negocio) sobre el cual se quiera abstraer y persistir el estado de la aplicación. Aquí es donde ocurre el modo anti-fallas, es el sistema inmune de la aplicación y es controlado por la lógica de programación realizada por el desarrollador con ciertas restricciones. La restricción mas importante es que el código debe ser determinista. Es decir, que si es ejecutado multiples veces, producirá el mismo resultado. El fragmento de código considerado como workflow debe estar aislado del mundo externo, cualquier interacción con sistemas externos al workflow debe hacerse a través de las _actividades_.

* Una actividad, no es otra cosa que cualquier fragmento de código (lógica de negocio), que permite interactuar al workflow con el mundo exterior. La actividad no maneja estado y no tiene ningura restricción de ningún tipo. 

* Un worker, permiter configurar workflows y/o actividades. Es un proceso demonio/stand-alone que sondea el servicio Temporal en busca de tareas, también realiza dichas tareas, y comunica los resultados de la ejecución de estas al servicio de Temporal. Los workers son implementados y operados por clientes de Temporal.  

![Temporal](/assets/images/temporal.png)

#### Instalación de Temporal

Temporal provee todos los mecanismos necesarios para instalar apropiadamente su plataforma. En este artículo nos concentraremos en la instalación vía docker.

* Imagen docker- El modo más sencillo para instalar Temporal (para desarrollo) es mediante una imagen docker (obviamente asumimos que tienes configurado docker y docker-compose). 

1. Clona el repositorio: [https://github.com/temporalio/docker-compose](https://github.com/temporalio/docker-compose)

git clone https://github.com/temporalio/docker-compose.git

2. Cambia al directorio raiz del proyecto.

cd  docker-compose

3. Ejecuta docker-compose up

docker-compose up

4. El servidor de Temporal provee una aplicación web para ciertas opciones administrativas (este componente es otro cliente que interactúa con el backend). Puedes validarlo en tu navegador en: localhost:8088

_Nota:_ Existen configuraciones alternativas. Por ejemplo puedes usar la configuración con mysql, postgress o cockroach (la configuración por defecto es con Cassandra). En caso de ser necesario sigue el README en: [https://github.com/temporalio/docker-compose/blob/main/README.md](https://github.com/temporalio/docker-compose/blob/main/README.md)

* Instalación Nativa- Debido a que Temporal es open-source puedes compilar sus fuentes a binario y ejecutar todo nativamente en esta ruta: [https://github.com/temporalio/temporal](https://github.com/temporalio/temporal)

* Kubernetes- Puedes instalar también Temporal en Kubernetes, para eso sigue esta guía: [https://github.com/temporalio/helm-charts](https://github.com/temporalio/helm-charts)

#### Ejemplo

A fín de iniciar facilmente con Temporal. Vamos a realizar como ejemplo el típico Hola Mundo con el Golang SDK (pronto haremos otro artículo con Java). 

* Definimos el paquete business que contiene las funciones Workflow y Activity dentro del archivo business/hello.go

{% gist 96bc2a9f8ebec215170e7bc7d598bfc3 %}

* Definimos el workfer dentro de worker/main.go

{% gist da2f6e984a2defc1ce410bc4f17ad603 %}

* Definimos el starter dentro de starter/main.go

{% gist ff7a9b863ac14aad61701d46c2a6f893 %}

#### Ejecutar localmente el ejemplo

Una vez instalado y ejecutado Temporal (ver paso previo de este artículo), descarga el ejemplo desde esta [ruta](https://github.com/sistecma/temporalio). Busca la carpeta app/go/hola y esta es la carpeta raiz.

1. Ejecuta: go run worker/main.go

{% gist 862fef234ce1da8dd3d57d05fb65949c %}

2. Abre otra terminal y ejecuta: go run starter/main.go

{% gist 8aea8785d42158932ef2c81b6bca1c84 %}

3. Deberías obtener como impreso en consola: Resultado: Hola Sistecma


#### Conclusión

En este breve artículo solo hemos visto de modo muy breve que es Temporal, tocamos en muy alto nivel como armar una aplicación en Temporal y terminología básica.En otros artículos revisaremos en más detalle la arquitectura, casos de uso, etc.



