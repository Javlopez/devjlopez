---
title: "Herencia en Go: Composición sobre Herencia Tradicional"
date: 2025-01-13T10:57:25-06:00
draft: false
tags: ["golang", "inheritance", "composition"]
image: 'ai-generated-8113881_1280.jpg'
description: "Aprende sobre composición en Go usando embedding y agregación. Descubre cómo construir tipos complejos de forma flexible y eficiente, evitando problemas de herencia tradicional."
---

Go, a diferencia de lenguajes como Java o C++, no implementa la herencia de clases de la forma tradicional. En lugar de jerarquías de clases, Go promueve la composición como mecanismo principal para la reutilización de código y la creación de tipos más complejos. Esta aproximación ofrece mayor flexibilidad y evita algunos de los problemas asociados con la herencia profunda.

¿Qué es la Composición en Go?

La composición es un principio fundamental en el diseño de software que prioriza la construcción de objetos complejos mediante la combinación de objetos más simples. En lugar de heredar características de una jerarquía de clases (como en la herencia tradicional), la composición permite crear tipos nuevos componiéndolos a partir de otros tipos existentes. Esto fomenta la reutilización de código, la flexibilidad y un menor acoplamiento entre las partes del sistema.

En Go, la composición se implementa principalmente a través de dos mecanismos:

 - Embedding (Incrustación): Un tipo se incluye directamente dentro de la definición de otro tipo, sin darle un nombre de campo.
 - Agregación: Un tipo contiene un campo que es una instancia de otro tipo.

#### Tabla Comparativa: Embedding vs. Agregación


| Característica        | Embedding (Incrustación)                                   | Agregación                                               |
| --------------------- | -----------------------------------------------------------|----------------------------------------------------------|
| Relación              | "Es-un" (en cierto sentido)                                | "Tiene-un"                                               |
| Acceso a miembros    | Directo (promoción) o con nombre del tipo embebido (si hay colisión) | Explícito a través del nombre del campo                  |
| Espacio de nombres    | Implícito                                                 | Explícito (nombre del campo)                             |
| Sobrescritura         | Oculta el método del tipo embebido, pero se puede acceder usando el nombre del tipo embebido | No aplica (acceso explícito)                             |
| Flexibilidad          | Menor control directo, pero más conveniente para acceso rápido | Mayor control sobre cómo se utiliza el tipo agregado     |

### Embedding (Incrustación)

El embedding se realiza declarando un tipo dentro de otro sin especificar un nombre de campo. Esto crea una relación de "tipo-de" o "es-un" (en cierto sentido, aunque no es herencia en el sentido clásico). Los campos y métodos del tipo embebido se promocionan al tipo contenedor.

- **Promoción**: La promoción significa que los campos y métodos del tipo embebido se pueden acceder directamente como si fueran parte del tipo contenedor. No es necesario usar un selector (como objeto.campo).
- **Espacio de nombres implícito**: Aunque no hay un nombre de campo explícito, el tipo embebido crea un espacio de nombres implícito. Si hay colisiones de nombres entre los campos del tipo contenedor y los del tipo embebido, se debe usar el nombre del tipo embebido para acceder al campo o método específico (ej. instancia.TipoEmbebido.Campo).
- **Sobrescritura de métodos**: Si el tipo contenedor define un método con la misma firma (nombre y parámetros) que un método del tipo embebido, el método del tipo contenedor oculta al método del tipo embebido para las instancias del tipo contenedor. Sin embargo, se puede acceder al método del tipo embebido usando el espacio de nombres implícito (ej. instancia.TipoEmbebido.Metodo()).

Imaginemos un sistema que gestiona usuarios. Cada usuario tiene una dirección. Podemos usar embedding para representar esta relación:
Go
```go
package main

import "fmt"

type Address struct {
    Street  string
    City    string
    ZipCode string
}

func (a Address) FullAddress() string {
    return fmt.Sprintf("%s, %s, %s", a.Street, a.City, a.ZipCode)
}

type User struct {
    Name    string
    Email   string
    Address // Embedded Address
}

func main() {
    user := User{
        Name:  "Juan Pérez",
        Email: "juan.perez@ejemplo.com",
        Address: Address{
            Street:  "Calle Principal 123",
            City:    "Ciudad de México",
            ZipCode: "01000",
        },
    }

    fmt.Println(user.Name)
    fmt.Println(user.Address.FullAddress()) // Accediendo al método del tipo embebido
    fmt.Println(user.Email)

}
```
### Agregación - Aggregation

La agregación se realiza declarando un campo en un tipo que es una instancia de otro tipo. Esto crea una relación de "tiene-un". A diferencia del embedding, los campos y métodos del tipo agregado no se promocionan. Se debe acceder a ellos utilizando el nombre del campo.

- **Acceso explícito**: Para acceder a los miembros del tipo agregado, se utiliza el nombre del campo como selector (ej. instancia.campo.miembro).
- **Sin promoción**: No hay promoción de campos ni métodos.
- **Mayor control**: La agregación ofrece un mayor control sobre cómo se accede y se utiliza el tipo agregado.

```go
package main

import "fmt"

// (Address y User del ejemplo anterior)

type Company struct {
    Name    string
    Users   []User
    Address Address
}

func main() {
    company := Company{
        Name: "Empresa Ejemplo S.A.",
        Users: []User{
            {
                Name: "Usuario 1", 
                Email: "usuario1@ejemplo.com", 
                Address: Address{
                    Street: "Calle 1", 
                    City: "Monterrey", 
                    ZipCode: "64000",
                },
            },
            {
                Name: "Usuario 2", 
                Email: "usuario2@ejemplo.com", 
                Address: Address{
                    Street: "Calle 2", 
                    City: "Puebla", 
                    ZipCode: "72000",
                },
            },
        },
        Address: Address{Street: "Oficinas Centrales", City: "CDMX", ZipCode: "06000"},
    }

    fmt.Println(company.Name)
    fmt.Println("Dirección de la empresa:", company.Address.FullAddress())
    fmt.Println("Usuarios de la empresa:")
    for _, user := range company.Users {
        fmt.Println("-", user.Name, user.Email)
    }
}
```
Aquí, Company tiene una colección de Users. Accedemos a los usuarios a través del campo Users.

Estos ejemplos prácticos demuestran cómo la composición en Go, a través de embedding y agregación, permite modelar relaciones entre entidades de una manera más clara y eficiente que la herencia tradicional. Estos ejemplos son más realistas y te ayudarán a comprender mejor cómo aplicar la composición en tus proyectos.