# Prompts

Aquí van **todos los prompts que lanzaste** para hacer el ejercicio, en el orden en que los
lanzaste, con el modelo y la herramienta de cada uno.

Esto no es papeleo. Lo que se revisa es **cómo pediste las cosas**, no solo lo que salió: un
resultado flojo con un prompt bueno y un resultado flojo con un prompt vago necesitan feedback
distinto, y sin este archivo no se distinguen.

## Cómo rellenarlo

- Un apartado `## Prompt N` por cada prompt.
- **Pega el prompt tal cual lo lanzaste**, dentro del bloque de código, aunque ocupe diez líneas
  y aunque tenga faltas. No lo reescribas para que quede bien: el que arreglaste mentalmente
  después no es el que lanzaste.
- Incluye también los que **no funcionaron**. Suelen ser los más útiles de leer.
- `Modelo` y `Herramienta` en todos. Si cambiaste de una a otra a mitad, se nota aquí.

Borra el ejemplo de abajo cuando escribas el primero.

---

## Prompt 1

**Modelo:** Sonnet 5.5 xHigh
**Herramienta:** Claude Code

```
Existen dos ambientes, produccion que corr en PostgresSQL 17 y dev que esta en SQLite.

Migra este backend a Postgress, que corra en Docker,con BDs, una en desarrollo y otra de pruebas.

-Usa la imagen oficial de PostgresSQL 17, pgvector/pgvector:pg1

-Abre y amarra los puertos 54410 para desarrollo y 5441 para pruebas. No uses el default 5432: Asi evitamos que quien haya levantado PostgresSQL conflictue y el error que no se mostraria en Docker.

- La BD de pruebas existe solo en memoria ('tmpfs'), sin volumen fisico, es efimera, para que al hacer pruebas siempre se hagan sin los cambios de las pruebas anteriores.

- Los dos servicios llevan healthcheck, y al arranque debene4 star sanos. La propia imagen avisa que, la primera vez, crea la BD, y no acepta conexciones mientras tanto, ya que eso rompe a quien levanta varios contenedores a la vez.

- En el archivo Compose, no metas la llave `version:` ya qye es obsoleta en Docker.

- Se apunta a la DB de pruebas mediante `.env.test`, que AdonisJS carga solo cuando NODE_ENV=`test`

- Crea shortcuts en el Makefile para iniciar las DB, pararlas, migrar las dos y correr las pruebas.


De ninguna manera toques migraciones existentes. Si fuese necesario cambiar algunam, para y avisame antes. Y si encuentras algo que el motor rompe fuera de la configuracion, avisame y no lo arregles.

```

**Qué salió:** Los test en prueba salieron verde, el schema.ts no cambio, el analisis encontro que las tareas ahora no se marcan como vencidas. Creo el config/database.ts, apunto la configuracion al nu evo motor de PstgreSQL 17. Actualizo la documentacion hacia Postgress 17. y el procesdo tomo 16 minutos.



## Prompt 2

**Modelo:** Sonnet 5.5 xHigh
**Herramienta:** Claude Code

```
Corre una comprobacion de tipos (cd backend && npm run typecheck)), indica si esta en verde o rojo. 

Muestra un git diff de la rama en la que estas motor-vmrr que fue creada de s10/start,  y revisa en especifico 'backend/database/schema.ts'. Explica paso por paso , detalle por detalle que declaraciones han cambiado de lo que el proyecto conoce de su esquema. Ninguna migracion ha cambiado. Por cada declaracion indica:

1. El delta del schema.ts, si existe ya indicaste en el prompt anterior que no cambio nada pero VUELVE A REVISAR.
2. Explica que la definia como era antes y donde estaba definido.
3. Si el codigo del proyecto depende del esquema viejo. Ya encontraste que las tareas vencidas dejan de marcarse con el prompt anterior, revisa particularmente como se definia si una tarea estaba vencida. 

```

**Qué salió:** No hay cambios en backend/database/schema.ts, diff vacio pero indica que lo que cambio son los valores que llegan en ejecucion, que rompe dos cosaS:
Declaracion de  vencido: status: 'pending' | 'in_progress' | 'done' , pero NO HAY CHECK EN LA DB, en ninguno de los motores y lo hace el validador.

DueDate la declaracion cambia de un String en SQLite a un date en Postgress 17, y esto rompe el codigo, y ahora la fecha que llega no se convierte a UTC y se guarda como TZ=Europe/Madrid.

Y ahora se compara Date vs String y Date < String en todos los casos en Javascript, que convierte Date en milisegundos y cuando compara vs String se convierte en NaN y siempre da false, y como resultado ninguna tarea sale como vencida tenga le fecha que tenga.


Como consecuencia rompe el contrato de la APIdueDate sale como "2026-01-15T00:00:00.000Z" y en el frontend, y OpenAPI esperan AAAA-MM-DD, tirando un RangeError: Invalid time value y muy probablemente tumbaria la pagina de cualquier tarea con fecha.

"Por qué no lo detecta nada: schema_rules.ts fija el tipo declarado, así que schema.ts no cambia y la comprobación de tipos sigue en verde. Y ningún test toca dueDate ni isOverdue, así que los 23 siguen en verde."

El otro fallo que no sale es la longitud de los varchar(n), SQLite no aplica restriccion de longitud de varchar(n), claude metio 500 caracteres en el varchar(200) declarado.Postgress si la aplica y salta con: value too long for type character varying(255)

Pero los validadores cubren title y email pero no fullName que no tiene maxLength, un registro de mas de 255 caracteres que antes se guardaba ahora fallara.

No hubo arreglos, no hubo cambios. El dueData se arregla configurando eld river o el modelo. y el fullName en el validador, por tanto no hay commits.
