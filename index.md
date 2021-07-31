---
layout: home
title: Home
landing-title: 'Hello, Welcome to my Blog.'
description: null
image: null
author: null
show_tile: false
---

**Sistecma** es un sitio donde puedes aprender sobre innovación tecnológica, lenguajes de programación, sistemas distribuidos, microservicios, desarrollo web, sistemas altamente escalables. Nos enfocamos en brindar las mejores tecnologías que encajen para los casos de uso y retos modernos de tecnología de información y comunicación más comunes desde las pymes hasta las multinacionales.


El repositorio de Sistecma en [GitHub](https://github.com/sistecma): representa el index donde están todos los repositorios asociados con Sistecma. Cada repositorio representa un aprendizaje individual o modular.

### Series Spring desde Cero
Demuestra las capacidades de Spring Framework desde un punto de vista básico (desde cero) a fín de facilitar el aprendizaje y eliminar las barreras para los nuevos desarrolladores que deseen trabajar con Spring Framework. 

Pre-requisitos:
* Conocimientos de Java básico.
* Conocimientos de patrones de diseño.
* Conocimientos del uso de git
* Conocimientos de maven

A continuación el índice de artículos al momento:

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>

Bienvenido.
