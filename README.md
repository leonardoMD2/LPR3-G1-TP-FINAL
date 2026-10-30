# TP Final - FastAPI + DB Relacional

---

## Objetivo

El trabajo consiste en desarrollar una API REST con FastAPI sobre una base de datos PostgreSQL,en supabase, para gestionar un catálogo de películas, sus géneros y las reseñas que los usuarios escriben sobre ellas.

Lo que se va a evalúar es la organización del código, el modelado relacional de la información y la calidad de la interfaz HTTP (api), no la cantidad de funcionalidades ni la complejidad del producto. Se espera que la entrega pueda ejecutarse desde cero siguiendo las instrucciones del repositorio, sin pasos implícitos ni dependencias de configuración que no estén documentadas.

El acceso a los datos puede resolverse con el driver psycopg o con el cliente oficial de Supabase, y esa decisión debe quedar aislada en un único punto del código.

## Alcance funcional

La API debe permitir dar de alta, consultar, modificar y eliminar películas, indicando para cada una su título, año de estreno, portada y, opcionalmente, director y sinopsis. Cada película puede pertenecer a uno o varios géneros, y un mismo género agrupa muchas películas.

Los usuarios deben poder publicar reseñas sobre las películas, con un puntaje y un comentario. Una reseña siempre corresponde a un usuario y a una película existentes, y una misma persona no puede reseñar dos veces la misma película. El puntaje admitido es un valor entero dentro de un rango fijo, y esa restricción debe estar garantizada por la base de datos, no solo por la aplicación. La API debe poder informar, para cada película, cuántas reseñas tiene y cuál es su puntaje promedio.

La consulta de reseñas debe poder hacerse por película, y la creación de una reseña debe tomar la película desde la propia dirección del recurso, de modo que no exista ambigüedad entre la ruta y el cuerpo del pedido.

## Organización del código

Se solicita una separación explícita en capas. Los routers atienden el protocolo HTTP, validan la entrada y traducen el resultado a una respuesta; no deben contener consultas a la base ni reglas de negocio. La validación del cuerpo en las operaciones de alta y modificación se apoya en modelos de Pydantic, no en controles manuales dentro del endpoint. Los managers concentran las reglas de negocio y la coordinación entre distintas fuentes de datos, y no deben conocer códigos de estado ni excepciones propias de HTTP. Las interfaces definen los contratos que los managers implementan y que el resto del código consume, de manera que la implementación concreta pueda reemplazarse sin modificarlos.

El proyecto debe incluir además la configuración de la aplicación y los valores sensibles en variables de entorno, con un archivo de ejemplo versionado y el archivo real excluido del repositorio. En esencia un .env propio y un .env.example para github como explicación.

## Base de datos relacional

El esquema debe ser relacional y estar creado en el motor de supabase. Se deberán adjuntar los DER correspondientes en los cuales quede expuestas las relaciones entre las tablas.

Se espera que el modelo incluya claves primarias en todas las tablas, al menos dos claves foráneas que representen relaciones de uno a muchos, y una tabla intermedia que resuelva una relación de muchos a muchos entre dos entidades. La tabla intermedia puede o no tener ID propia (queda a criterio de cada uno).
La integridad de los datos debe quedar garantizada por el motor y no solo por las validaciones de la aplicación. En particular, deben existir restricciones de unicidad donde el negocio lo exija y un comportamiento de borrado coherente: al eliminar una entidad, sus registros dependientes no deben quedar huérfanos (CASCADE).

Se valorará que el esquema tenga los índices necesarios para los filtros y las búsquedas más frecuentes, y que las decisiones de modelado estén justificadas en el README.

## Búsqueda y filtros

El listado de películas debe aceptar filtros opcionales y combinables entre sí: búsqueda por texto sobre el título y el director, filtro por género, rango de años de estreno, puntaje promedio mínimo y criterio de ordenamiento, incluyendo orden ascendente y descendente.

Todos los parámetros deben validarse antes de llegar a la base, y debe rechazarse una combinación incoherente de valores, como un rango de años invertido. Las consultas deben construirse de forma segura: ningún valor provisto por el cliente puede concatenarse en el texto de una consulta, y el criterio de ordenamiento debe resolverse contra un conjunto cerrado de opciones permitidas.

El filtro por género, al atravesar una relación de muchos a muchos, no debe producir películas repetidas en el resultado ni alterar el total informado.

## Rutas dinámicas

Los identificadores de los recursos deben viajar en la ruta y validarse como numéricos y positivos, de modo que un valor inválido se rechace antes de consultar la base. La distinción entre un pedido mal formado y un recurso inexistente debe ser explícita y consistente en toda la API: el primero corresponde a un error de validación y el segundo a un recurso no encontrado.

Debe existir al menos una ruta anidada, en la que el recurso hijo se identifique a partir del recurso padre. También debe resolverse correctamente el orden de declaración de las rutas, de manera que una ruta literal no sea capturada por un parámetro dinámico del mismo segmento.

## Condiciones de entrega

La entrega se realiza como repositorio individual, con historial de commits que refleje el proceso de trabajo. El README debe permitir levantar el proyecto desde cero, indicar las variables de entorno necesarias y describir los endpoints disponibles, los parámetros que aceptan y los códigos de respuesta esperados.

No se aceptarán entregas que dependan de datos cargados manualmente desde la interfaz de administración del motor, ni que incluyan credenciales reales en el repositorio.

Se espera una API deployada en algún servicio a elección (render, vercel, otro).

## Criterios de evaluación

Se evaluará en primer lugar la correcta aplicación de la separación en capas y el uso de interfaces, dado que es el objetivo central del trabajo. En segundo lugar se considerará el modelado relacional: claves foráneas, tabla intermedia, restricciones e integridad ante borrados y los respectivos diagramas de entidad-relación.

Luego se evaluará la calidad de la interfaz HTTP, es decir, la coherencia de las rutas, la validación de parámetros, los códigos de estado y la claridad de los mensajes de error. Se espera que el alta responda con un código de creación y la baja con uno de operación exitosa sin cuerpo, que un identificador inexistente se distinga de un pedido mal formado. También se tendrá en cuenta la implementación de los filtros combinables, de la paginación con total, el correcto uso de managers e interfaces; inyección de dependencias y el resto de conceptos aplicados a lo largo del cuatrimestre. 

Finalmente se realizará una breve defensa oral del proyecto.
