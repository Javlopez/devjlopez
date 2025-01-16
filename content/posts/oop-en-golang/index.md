---
title: "OOP en Golang"
date: 2025-01-15T15:55:25-06:00
draft: false
tags: ["Go", "OOP", "interfaces", "herencia"]
image: 'pexels-divinetechygirl-1181675.jpg'
description: "Go no es un lenguaje orientado a objetos en el sentido clásico, pero nos da las herramientas para lograr los mismos objetivos de una manera más simple y directa. Con structs, métodos, composición e interfaces, podemos escribir código organizado, reutilizable y flexible. ¡Así que no te preocupes si no ves clases y herencia tradicional"
---
¡Hola, Go-pers! 
Hoy vamos a platicar sobre un tema que a veces genera debate: 

La Programación Orientada a Objetos (OOP en inglés) en Go. 
Algunos dicen que Go no es orientado a objetos, otros que sí, pero a su manera. 

¿Quién tiene razón?, yo creo que ambos, si bien Go no sigue el modelo clásico de POO como Java o C++,
nos da herramientas para lograr cosas similares,
con un enfoque más práctico y flexible.

¿Qué onda con la OOP?

Antes de meternos a ver como funciona la OOP en Go, 
recordemos rapidamente qué es la OOP. Básicamente, 
se trata de organizar nuestro código en "objetos" 
que tienen propiedades (datos) y métodos (acciones). 
Esto nos ayuda a encapsular, reutilizar y organizar mejor nuestro código. 

## Las principales caracteristicas de la OOP son:

- Encapsulamiento: Ocultar los detalles internos de un objeto y exponer solo lo necesario. 
- Herencia: Crear nuevas clases a partir de clases existentes, heredando sus propiedades y métodos.
- Polimorfismo: La capacidad de un objeto de tomar muchas formas.

### Go y la POO: Un enfoque diferente

Go no tiene clases en el sentido tradicional. 
En lugar de eso, usa structs para definir tipos de datos personalizados, 
que vendrían a ser como nuestros "objetos". 
Y para los métodos, Go nos permite asociarlos directamente a estos structs.

Ejemplo 1: Structs y métodos

Imaginemos que queremos representar un usuario:

```go
package main

import "fmt"

type User struct {
    Name string
    LastName   string
    Age  int
}

func (u User) displayData() {
    fmt.Printf("Hola soy %s%s y tengo %d años", u.Name, u.LastName, u.Age)
}

func main() {
    user := User{Nombre: "Javier", LastName: "LL", Edad: 123}
    user.displayData() // Imprime: Hola soy JavierLL y tengo 123 años
}
```

Aquí, User es nuestro struct (como una clase), y displayData() es un método asociado a él y con eso ¡Ya tenemos encapsulamiento!

¿Y la herencia? Recordemos que Go prefiere la composición leer más sobre composición en el post anterior https://devjlopez.com/posts/herencia-en-go-composicion/

En lugar de herencia, Go nos anima a usar la composición. Esto significa que podemos incluir un struct dentro de otro para reutilizar su funcionalidad.

### Polimorfismo con interfaces

Go logra el polimorfismo con interfaces. Una interfaz define un conjunto de métodos que un tipo debe implementar.

Ejemplo 3: Interfaces

```Go
package main

import "fmt"

// User define el comportamiento de un usuario autenticado.
type User interface {
    IsAuthenticated() bool
    GetRole() string // Añadimos un método para obtener el rol
}

// BasicUser representa un usuario básico.
type BasicUser struct {
    Username string
}

func (u BasicUser) IsAuthenticated() bool {
    return true // Un usuario básico siempre está autenticado (para este ejemplo)
}

func (u BasicUser) GetRole() string {
    return "basic"
}

// Admin representa un usuario administrador.
type Admin struct {
    Username string
}

func (a Admin) IsAuthenticated() bool {
    return true
}

func (a Admin) GetRole() string {
    return "admin"
}

// Staff representa un miembro del staff.
type Staff struct {
    Username string
    Permissions []string
}

func (s Staff) IsAuthenticated() bool {
    return true
}

func (s Staff) GetRole() string {
    return "staff"
}

func main() {
    users := []User{
        BasicUser{Username: "usuario1"},
        Admin{Username: "admin1"},
        Staff{Username: "staff1", Permissions: []string{"read", "write"}},
    }

    for _, user := range users {
        // divided for better reading
        fmt.Printf("Rol de %s: %s, Autenticado: %t\n", 
            user.(interface{ GetRole() string }).GetRole(),
            user.GetRole(),
            user.IsAuthenticated()
        )
        switch user.(type){
        case BasicUser:
            fmt.Println("Es un usuario basico")
        case Admin:
            fmt.Println("Es un Admin")
        case Staff:
            fmt.Println("Es un miembro del Staff")
        }

    }

    // Ejemplo de uso polimórfico: función que requiere un User
    authorize(users[1]) // Autoriza al admin
    authorize(users[0]) // Autoriza al usuario basico

}

func authorize(u User) {
    if u.IsAuthenticated() {
        fmt.Println("Usuario autorizado.")
    } else {
        fmt.Println("Usuario no autorizado.")
    }
}
```
El polimorfismo se ve claramente en el bucle `for...range` y en la función `authorize()`. 
Tanto el bucle como la función trabajan con la interfaz `User`.
No les importa si el usuario es un `BasicUser`, un `Admin` o un `Staff`. 
Solo les importa que implementen el método `IsAuthenticated()`.

Cuando se llama a `user.IsAuthenticated()`, 
Go determina dinámicamente el tipo concreto de user y 
ejecuta la implementación correspondiente. 
Esto es el polimorfismo en acción: un mismo método `IsAuthenticated()` se comporta de forma diferente según el tipo del objeto.

Conclusión:

Go no es un lenguaje orientado a objetos en el sentido clásico, pero nos da las herramientas para lograr los mismos objetivos de una manera más simple y directa. Con structs, métodos, composición e interfaces, podemos escribir código organizado, reutilizable y flexible. ¡Así que no te preocupes si no ves clases y herencia tradicional, Go tiene su propia onda!

