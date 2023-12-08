# Bootcamp Lemoncode Backend Continuo - [Documental] Módulo 2 - MongoDB
## Caso  básico
# Laboratorio MongoDB

Vamos a trabajar con el set de datos de Mongo Atlas _airbnb_. Lo puedes encontrar en este enlace: https://drive.google.com/drive/folders/1gAtZZdrBKiKioJSZwnShXskaKk6H_gCJ?usp=sharing

Para restaurarlo puede seguir las instrucciones de este videopost:
https://www.lemoncode.tv/curso/docker-y-mongodb/leccion/restaurando-backup-mongodb

> Acuerdate de mirar si en el directorio `/opt/app` del contenedor Mongo hay contenido de backups previos que haya que borrar

Para entregar las soluciones, añade un README.md a tu repositorio del bootcamp incluyendo enunciado y consulta (lo que pone '_Pega aquí tu consulta_').

## Introducción

En este base de datos puedes encontrar un montón de alojamientos y sus reviews, esto está sacado de hacer webscrapping.

**Pregunta**. Si montaras un sitio real, ¿Qué posibles problemas pontenciales les ves a como está almacenada la información?

```md
Veo que por ejemplo si hay que actualizar los tipos de servicios sería un problema ya que están todos metidos en la colección. Yo no metería tantas reviews, sólo pondría las 10 últimas como mucho. El precio, ¿no sabemos en que moneda está? Imagino que dependiendo del país se debería mostrar en una moneda u otra y hacer la conversión.
```

## Obligatorio

Esta es la parte mínima que tendrás que entregar para superar este laboratorio.

### Consultas

- Saca en una consulta cuantos alojamientos hay en España.

```js
db.listingsAndReviews.countDocuments(
    {"address.country" : {$eq: "Spain"}}
)
```

- Lista los 10 primeros:
  - Ordenados por precio de forma ascendente.
  - Sólo muestra: nombre, precio, camas y la localidad (`address.market`).

```js
db.listingsAndReviews.find(
    {"address.country" : {$eq: "Spain"}},
    {_id: 0, name: 1, price: 1, beds: 1, "address.market": 1}
).limit(10).sort(
    {price:1}
)
```

### Filtrando

- Queremos viajar cómodos, somos 4 personas y queremos:
  - 4 camas.
  - Dos cuartos de baño o más.
  - Sólo muestra: nombre, precio, camas y baños.

```js
db.listingsAndReviews.find(
    {
        "address.country" : {$eq: "Spain"},
        "beds" : {$eq: 4 },
        "bathrooms" : {$gte: 2 },
        
    },
    {_id: 0, name: 1, price: 1, beds: 1, bathrooms: 1}
).limit(10).sort(
    {price:1}
)

```

- Aunque estamos de viaje no queremos estar desconectados, así que necesitamos que el alojamiento también tenga conexión wifi. A los requisitos anteriores, hay que añadir que el alojamiento tenga wifi.
  - Sólo muestra: nombre, precio, camas, baños y servicios (`amenities`).

```js
db.listingsAndReviews.find(
    {
        "address.country" : {$eq: "Spain"},
        "beds" : {$eq: 4 },
        "bathrooms" : {$gte: 2 },
        "amenities" : { $regex: ".*Wifi.*" } ,
        
    },
    {_id: 0, name: 1, price: 1, beds: 1, bathrooms: 1, amenities:1}
).limit(10).sort(
    {price:1}
)

```

- Y bueno, un amigo trae a su perro, así que tenemos que buscar alojamientos que permitan mascota (_Pets allowed_).
  - Sólo muestra: nombre, precio, camas, baños y servicios (`amenities`).

```js
db.listingsAndReviews.find(
    {
        "address.country" : {$eq: "Spain"},
        "beds" : {$eq: 4 },
        "bathrooms" : {$gte: 2 },
        "amenities" :  { $all: ["Wifi", "Pets allowed"] },
        
    },
    {_id: 0, name: 1, price: 1, beds: 1, bathrooms: 1, amenities:1}
).limit(10).sort(
    {price:1}
)

```

- Estamos entre ir a Barcelona o a Portugal, los dos destinos nos valen. Pero queremos que el precio nos salga baratito (50 $), y que tenga buen rating de reviews (campo `review_scores.review_scores_rating` igual o superior a 88).
  - Sólo muestra: nombre, precio, camas, baños, rating y localidad.

```js
db.listingsAndReviews.find(
    {
        $or: [ 
            { "address.market": { $eq: "Barcelona" } }, 
            { "address.country": {$eq: "Portugal"} } 
            ], 
        "price" : {$eq: 50 },
        "review_scores.review_scores_rating": {$gte: 88}
    },
    {
        _id: 0, 
        name: 1, 
        price: 1, 
        beds: 1, 
        bathrooms: 1, 
        "review_scores.review_scores_rating":1,
        "address.market":1,
    }
).sort(
    {"review_scores.review_scores_rating":-1}
)

```

- También queremos que el huésped sea un superhost (`host.host_is_superhost`) y que no tengamos que pagar depósito de seguridad (`security_deposit`).
  - Sólo muestra: nombre, precio, camas, baños, rating, si el huésped es superhost, depósito de seguridad y localidad.

```js
db.listingsAndReviews.find(
    {
        $or: [ 
            { "address.market": { $eq: "Barcelona" } }, 
            { "address.country": {$eq: "Portugal"} } 
            ], 
        "price" : {$eq: 50 },
        "review_scores.review_scores_rating": {$gte: 88},
        "host.host_is_superhost": {$eq: true},
        "security_deposit": {$eq:0},
    },
    {
        _id: 0, 
        name: 1, 
        price: 1, 
        beds: 1, 
        bathrooms: 1, 
        "review_scores.review_scores_rating":1,
        "host.host_is_superhost":1,
        "security_deposit": 1,
        "address.market":1,
        
    }
).sort(
    {"review_scores.review_scores_rating":-1}
)
```

### Agregaciones

- Queremos mostrar los alojamientos que hay en España, con los siguientes campos:
  - Nombre.
  - Localidad (no queremos mostrar un objeto, sólo el string con la localidad).
  - Precio

```js
db.listingsAndReviews.aggregate([
    {
        $match:{
            "address.country": "Spain",
        }
    },
    {
        $project:{
            _id: 0,
            name: 1, 
            localidad: "$address.market",
            price: 1
        }
    },
    {
        $count: 
    }
])
```

- Queremos saber cuantos alojamientos hay disponibles por pais.

```js
db.listingsAndReviews.aggregate([
    {
        $group:{
            _id: "$address.country",
            count: {$sum: 1}
        }
    },
    {
        $sort: {_id: 1}
    }

])
```