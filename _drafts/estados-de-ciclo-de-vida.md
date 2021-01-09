---
layout: post
title: Estados asociados al ciclo de vida de Spring Beans
description: Estados asociados a creación y destrucción de beans y como aprovecharlo
image: 
author: Hernán Moreno
---

Spring ofrece varias formas de aprovechar el ciclo de vida de los beans. En muy alto nivel tenemos creación y destrucción del bean, sin embargo entre cada etapa existen pasos intermedios que nos pueden ser de utilidad. Por ejemplo en ocasiones que necesitamos hacer algún tipo de inicialización al momento de creación del bean o limpieza de recursos al momento de destruirse el bean. 

#### Creación
La etapa de Creación abarca desde la instanciación del bean hasta que esta listo para uso. Durante esta etapa el Contenedor de Spring sigue a través de las siguientes sub-etapas o pasos en estricta secuencia:

1- Instanciación
2- Llenado de propiedades
3- Llamar a BeanNameAware.setBeanName
4- Llamar a BeanFactoryAware.setBeanFactory
5- Llamar a ApplicationContextAware.setApplicationContext
6- Pre-inicialización (BeanPostProcessor)
7- Llamar a InitializingBeans.afterPropertiesSet
8- Método Init personalizado
9- Post-inicialización (BeanPostProcessor)

#### Eliminación
Es la etapa que ocurre una vez que el contenedor se apaga. Los pasos son los siguientes:

1- Apagado del contenedor
2- Llamar a destroy de los beans desechables.
3- Llamar a método personalizado destroy

#### Interfaces XXAware
Spring proporciona varias interfaces XXAware. Rara vez son necesarias por los programadores puesto acoplan nuestro código a Spring. Pero si la implementamos, con ellas podemos acceder a la infraestructura de Spring Framework. Son útiles para casos de uso como adquirir el nombre de los bean para propósitos de logging o wiring.  

* BeanNameAware: El callback setBeanName() de esta interfaz provee el nombre del bean.
* BeanFactoryAware: Provee setBeanFactory(), un callback que provee la fábrica propietaria de la instancia del bean. 
* ApplicationContextAware: El método setApplicationContext() de esta interfaz provee el ApplicationContext de este bean.

#### BeanPostProcessor
Spring proporciona la interfaz BeanPostProcessor que nos permite para aprovechar el ciclo de vida del contexto de Spring e interactuar con los beans a medida que estos se procesan. 

* postProcessBeforeInitialization: Spring llama a este método después de llamar a los métodos de las interfaces aware y antes de cualquier callback de inicialización de bean, como el afterPropertiesSet de InitializingBean o un método init personalizado.

* postProcessAfterInitialization: Spring llama a este método después de cualquier callback de inicialización de bean.

#### Interfaces de callback InitializingBean y DisposableBean
Spring proporciona las siguientes dos interfaces de callback:

* InitializingBean: Declara el método afterPropertiesSet que se puede utilizar para escribir la lógica de inicialización. El contenedor llama al método después de que se establecen las propiedades.
* DisposableBean: declara el método destroy que se puede usar para escribir cualquier código de limpieza. El contenedor llama a este método durante la destrucción del bean en el apagado del contenedor.

#### Métodos Init and Destroy personalizados
Al declarar bean en la configuración XML, puede especificar los atributos init-method y destroy-method en la etiqueta. Ambos atributos especifican métodos personalizados en la clase de bean. El método declarado en el atributo init-method se llama después de que Spring inicialice las propiedades del bean a través del setter o los argumentos del constructor. Podemos utilizar este método para validar las propiedades inyectadas o realizar cualquier otra tarea.

Spring llama al método declarado en el atributo destroy-method justo antes de que se destruya el bean.



