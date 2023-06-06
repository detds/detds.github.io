---
title: "Backend shopping system"
classes: wide
excerpt: "A simple implementation of a backend shopping system created with Java and Spring Boot, using Spring Data JPA/Hibernate to interact with PostgreSQL database."
header:
  overlay_color: "#003641"
  # overlay_image: assets/images/portfolio/project-spring-java-17/cart2.jpg
  # overlay_filter: rgba(0, 83, 91, 0.5)
  teaser: assets/images/portfolio/project-spring-java-17/cart-th-2.jpg
sidebar:
  - title: "Application Demo"
    text: "[https://javaspring.fly.dev](https://javaspring.fly.dev)"
    image: assets/images/portfolio/project-spring-java-17/cart-th-2.jpg
    image_alt: "Cart image"
  - title: "Repository"
    text: "Github: [https://github.com/detds/project-spring-java-17](https://github.com/detds/project-spring-java-17)"
  - title: "Technologies"
    text: "Java 17, Spring Boot 2 (with Spring Web MVC, Spring Data JPA), Maven, H2 Database, PostgreSQL Database, Postman, ~~Heroku~~ Fly.io"
gallery:
  - url: /assets/images/unsplash-gallery-image-1.jpg
    image_path: assets/images/unsplash-gallery-image-1-th.jpg
    alt: "placeholder image 1"
  - url: /assets/images/unsplash-gallery-image-2.jpg
    image_path: assets/images/unsplash-gallery-image-2-th.jpg
    alt: "placeholder image 2"
  - url: /assets/images/unsplash-gallery-image-3.jpg
    image_path: assets/images/unsplash-gallery-image-3-th.jpg
    alt: "placeholder image 3"
---

# A Web services project with Spring Boot e JPA/Hibernate

[![Website](https://img.shields.io/website?down_message=offline&label=Application%20Status&up_message=online&url=https%3A%2F%2Fjavaspring.fly.dev%2Fhealth)](https://javaspring.fly.dev/health)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellowgreen.svg)](https://github.com/detds/project-spring-java-17/blob/main/LICENSE)

A simple implementation of a **backend** shopping system created with **Spring Boot** framework and **Java**, using 
**Spring Data JPA/Hibernate** to interact with **PostgreSQL database**.

This project was deployed to Fly.io and can be accessed at this link: [https://javaspring.fly.dev](https://javaspring.fly.dev).

In this app, we can **create**, **read**, **update** or **delete** users (CRUD). We can also read products, orders and
categories.

This project also contains: **exception handling** (findById, delete and update methods) and **one-to-one**, 
**one-to-many** and **many-to-many associations**.

## Technologies used

- Java 17
- Spring Boot 2 (with Spring Web MVC, Spring Data JPA)
- Maven
- H2 Database
- PostgreSQL Database
- Postman
- Deploy app to ~~Heroku~~ Fly.io

## Domain Model

![Domain Model image](https://raw.githubusercontent.com/detds/project-spring-java-17/main/assets/DomainModel.png)

## Summary of HTTP methods and description of its actions

| Methods | Urls            | Actions                |
|---------|-----------------|------------------------|
| POST    | /users          | create new User        |
| GET     | /users          | find all Users         |
| GET     | /users/:id      | find a User by :id     |
| PUT     | /users/:id      | update a User by :id   |
| DELETE  | /users/:id      | delete a User by :id   |
| GET     | /products       | find all Products      |
| GET     | /products/:id   | find a Product by :id  |
| GET     | /orders         | find all Orders        |
| GET     | /orders/:id     | find a Order by :id    |
| GET     | /categories     | find all Categories    |
| GET     | /categories/:id | find a Category by :id |

## Running

The Spring profile is preconfigured for testing that uses the H2-database to save data in memory.

> To run this project using a local **PostgreSQL database**, you need to configure *application-dev.properties* file properly
> and set Spring profile to "dev" in *application.properties*. You can find these files in this
> link: [https://github.com/detds/project-spring-java-17/tree/main/src/main/resources)](https://github.com/detds/project-spring-java-17/tree/main/src/main/resources)

To run this project follow these steps:

1. Clone the repository to your workspace folder:

    ```shell
    $ git clone https://github.com/detds/project-spring-java-17
    ```

2. Navigate to the repository folder:

    ```shell
    $ cd project-spring-java-17
    ```

3. Run app:

    ```shell
    $ ./mvnw spring-boot:run
    ```

4. Open browser at http://localhost:8080/ or http://localhost:8080/h2-console to use H2-console.