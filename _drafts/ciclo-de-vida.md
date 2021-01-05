---
layout: post
title: Ciclo de vida de Spring Beans
description: Cúal es el ciclo de vida de un Spring Bean y como aprovecharlo
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
6- Pre-inicialización (Post-procesadores del bean)
7- Llamar a InitializingBeans.afterPropertiesSet
8- Método Init personalizado
9- Post-inicialización (Post-procesadores del bean)

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

