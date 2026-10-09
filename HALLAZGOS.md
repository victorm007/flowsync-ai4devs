# Hallazgos

Aquí van **las tres líneas** del ejercicio, una por cada punto de abajo. Es lo único que hay que
traer hecho: un cambio de motor a medias con estas tres líneas escritas vale más que lo contrario,
porque lo que se discute en el directo es dónde te chocaste.

Escribe **una sola línea por punto**, con tus palabras y con lo que mediste, no con lo que suponías.

## 1. Las filas que cambian y la rama

Git diff no encontro ningun cambio en el schema (backend/database/schema.ts), el agente no hizo ningun cambio ni en datos ni esquema, no arreglo nada de los problema que encontro.

-

## 2. Lo que la batería de pruebas no podía ver

Con el prompt Claude arrojo en su analisis unas declaraciones de lo que pasa al migrar el schema de SQLite a Postgress 17:

1. Ninguna tarea sale vencida. Es la definición única de «vencida» en todo el sistema, así que no hay otra regla que lo compense.
2. Se rompe el contrato de la API. dueDate sale como "2026-01-15T00:00:00.000Z", mientras que el frontend (lib/types.ts:94) y OpenAPI esperan AAAA-MM-DD, lo que hara que falle el frontend y se rompe aqui:
   `app/models/task.ts:52-56:`
```
isOverdueOn(referenceDay: string): boolean {
    if (this.dueDate === null) return false
    return this.dueDate < referenceDay
}
```
3. La página de la tarea en el frontend probablemente se rompe. formatDay() (task-page.tsx:27) hace split('-') y le llega "15T00:00:00.000Z". He ejecutado esa función en Node con el valor nuevo y lanza RangeError: Invalid time value. No lo he probado en el navegador, pero ese error al renderizar tumbaría la página de cualquier tarea con fecha. Además, el <input type="date"> recibe un valor que no puede mostrar.
. 
-

## 3. Tu duda

No pude comprobar el comportamiento en vivo de la aplicacion, aunque las explicaciones del agente me parecen muy plausibles y claras. Tengo mas dudas de que en unas pruebas a detalle que mas podria romperse.

-
