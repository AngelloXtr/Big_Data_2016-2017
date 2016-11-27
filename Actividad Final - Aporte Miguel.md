# Aportes Miguel D. sobre práctica final #

Recopilando la información que ha recogido Angello en el archivo Requisitos_DB_NOSQL más el aporte se tiene los siguientes apartados.

## Resumen de BD Conocidas ##

### MongoDB ###
 * Orientada a documentos (JSON).
 * No permite transacciones.
 * Auto-Sharding: Sencillez en escalabilidad horizontal (automática).
 * CP: Consistencia frente a disponibilidad.
 * Permite búsquedas en atributos de subdocumentos.

### Cassandra ###
 * Orientada a columnas.
 * Consistencia final en el tiempo.
 * AP: Disponibilidad frente a consistencia.
 * Gestiona gran número de accesos simultáneos.
 * Grandes velocidades de acceso.
 * Fácil mantenimiento.

### Riak ###
 * Clave-valor.
 * AP: Disponibilidad frente a consistencia.
 * Gran velocidad de acceso a un único valor.
 * No puede buscar por atributos dentro del valor.
 * Consistencia final en el tiempo.
 * Grandes velocidades de acceso.
 * Ideal para almacenamiento en diversos formatos.

### Neo4j ###
 * Orientado a grafo (de propiedades).
 * Permite definición de propiedades, nodos y relaciones.
 * Sistema de transacciones ACID.
 * Permite Sharding.
 * Escalabilidad vertical sobre horizontal.
 * Mejora tiempos de respuesta sobre SQL Joins.
 * Consistencia y disponibilidad, pero posible caída de sistema en caso de fallo.

### HBase ###
(Según: https://www.linkedin.com/pulse/real-comparison-nosql-databases-hbase-cassandra-mongodb-sahu).

 * Orientado a columnas.
 * CP: Consistencia frente a disponibilidad.
 * Gran velocidad de acceso y escritura.
 * Sobre HDFS de Hadoop.
 * Escalabilidad horizontal.

## Gestión de suscripciones ##
 * Alto número de usuarios concurrentes.
 * Gestión de información de suscripciones de alta disponibilidad.
 * Información a gestionar:
    * Dado un usuario, saber si tiene la suscripción activa.

### Propuesta: RIAK###
Debido a que la única información a gestionar es conocer si un usuario tiene la suscripción activa o no, se propone un sistema Clave-Valor de manera que ofrezca la máxima disponibilidad.

La clave puede ser un código de usuario, y el valor True/False (si tiene la suscripción activa o no). Como se comenta que el número de usuarios activos se duplica cada mes, también se consigue la escalabilidad horizontal deseada.

Tampoco hace falta almacenar información muy compleja, por lo que no es necesario realizar búsqueda por atributos, únicamente es necesario obtener si esta la suscripción activa o no.

Agregado:

 * Clave: clave de usuario.
 * Valor: True/False

## Catálogo de películas y series ##
 * Garantizar la accesibilidad de todos los usuarios.
 * Integridad de datos de vital importancia.
 * Disponibilidad por regiones.
 * No tiene la necesidad de escalar horizontalmente.
 * El sistema solo tendrá metadatos sobre películas, series y actores. (Películas y series en sí se almacenarán en un sistema de streaming separado).
 * Información a gestionar:
    * Metadatos de películas y series: indicar películas/series similares, categoría, año de estreno y sinopsis.
    * Metadatos de actores: indicar películas/series donde han actuado.

### Propuesta: Neo4j###
Neo4j al utilizar propiedades ACID, puede ofrecer tanto la integridad (consistencia) como la disponiblidad que se necesita.

Además, el catálogo crece lentamente, por lo que la escalabilidad horizontal no es un requisito esencial. Es adecuado para representar las relaciones entre actores y películas y series, y se ahorra tiempo sobre tener que realizar SQL Joins. 

Por ejemplo, si se busca un actor se podrá obtener de forma rápida las películas/series en las que ha actuado, además de sus atributos y otras películas/series relacionadas con éstas.

Modelado del grafo:

 * Película -[similar]-> Película.
 * Película -[similar]-> Serie.
 * Atributos de película: categoría, año de estreno y sinopsis.
 * Actor -[actúa en] -> Película.
 * Actor -[actúa en] -> Serie.

## Análisis de usuarios ##
 * Importante la escalabilidad horizontal.
 * Alta disponiblidad.
 * Información a gestionar:
    * Almacena información de las conexiones de los usuarios: conexión, hora de la conexión, página vista y tiempo que ha permanecido en ella.
    * Adicionalmente: a partir de la clave de usuario, acceder a información sobre todas sus conexiones.


### Propuesta: Cassandra ###
Debido a que es importante la alta escalabilidad horizontal, disponibilidad y tiempo de respuesta, se propone utilizar un sistema orientado a columnas AP como Cassandra.

Gracias a ello se podrá almacenar abundante información de conexiones de usuarios y poder a esa información a través de la clave de usuario (mejor que, por ejemplo, _id de MongoDB).

Debido a su sistema de supercolumnas y claves, se puede conseguir muy rápidos tiempos de respuesta, organizando toda la información necesaria.

Agregado:

 * Clave de usuario (Clave 1).
    * Hora de conexion (Clave 2).
        * Conexión.
        * Página vista.
        * Tiempo que ha permanecido en ella.


## Sistema de recomendación ##
 * Respuesta rápida.
 * El sistema de cálculo de recomendaciones es externo.
 * Información a gestionar:
    * Para cada usuario: información de votaciones, últimas películas y series que ha visto y lista de recomendaciones personalizadas.


### Propuesta HBase ###
Para un sistema de recomendación se considera importante la consistencia de la información, para que ciertamente se le recomiende al usuario las películas adecuadas, además de que vea integridad en las votaciones que ha realizado.

Para conseguir una respuesta rápida de acceso, además de una escritura rápida (cada vez que el usuario vea una nueva película/serie, se recalcula el sistema de recomendación), manteniendo la consistencia, se propone utilizar HBase.

Agregado:

 * Clave de usuario (Clave 1):
    * Lista de votaciones realizadas.
    * Películas que ha visto.
    * Series que ha visto.
    * Recomendaciones personalizadas (calculada externamente).







