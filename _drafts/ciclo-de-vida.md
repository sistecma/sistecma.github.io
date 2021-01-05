---
layout: post
title: Ciclo de vida de Spring Beans
description: null
image: 
author: Hernán Moreno
---

El ciclo de vida de los Spring Beans lo podemos dividir en 2 grandes partes:

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

 
