# Bootcamp Lemoncode Backend Continuo - [Documental] Módulo 2 - MongoDB
## Caso  opcional
# Laboratorio MongoDB

Vamos a trabajar con el set de datos de Mongo Atlas _airbnb_. Lo puedes encontrar en este enlace: https://drive.google.com/drive/folders/1gAtZZdrBKiKioJSZwnShXskaKk6H_gCJ?usp=sharing

Para restaurarlo puede seguir las instrucciones de este videopost:
https://www.lemoncode.tv/curso/docker-y-mongodb/leccion/restaurando-backup-mongodb

> Acuerdate de mirar si en el directorio `/opt/app` del contenedor Mongo hay contenido de backups previos que haya que borrar

Para entregar las soluciones, añade un README.md a tu repositorio del bootcamp incluyendo enunciado y consulta (lo que pone '_Pega aquí tu consulta_').

## Introducción

En este base de datos puedes encontrar un montón de alojamientos y sus reviews, esto está sacado de hacer webscrapping.
## Opcional

- Queremos saber el precio medio de alquiler de airbnb en España.

```js
db.listingsAndReviews.aggregate([
    {
        $match:{
            "address.country": "Spain",
        }
    },
    {
        $group: {
            _id: null,
            precioMedio: { $avg: '$price' }
      },
    }
])
```

- ¿Y si quisieramos hacer como el anterior, pero sacarlo por paises?

```js
db.listingsAndReviews.aggregate([
    {
    $group: {
        _id:  "$address.country",
        precioMedio: { $avg: '$price' }
      }
    }
])
```

- Repite los mismos pasos pero agrupando también por numero de habitaciones.

```js
db.listingsAndReviews.aggregate([
    {
    $group: {
        _id:  {
            Pais: "$address.country",
            Habitaciones: "$bedrooms"
        },
        precioMedio: { $avg: '$price' }
      }
    },
    {
        $sort:{
            _id:-1
        }
    }
])
```