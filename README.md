# Prueba técnica Avianca – Data, Analytics & Martech

- Página: https://github.com/KANG-27/prueba-gtm-ga4.git
- GTM: GTM-TK824L7T
- GA4: G-K4ZS4G19LN

## Archivos

- `index.html`: la página de prueba. Solo cambié el ID del contenedor de GTM.
- `gtm/html-push-flight_search.html`: el código del tag de HTML personalizado del ejercicio 2, tal como está en GTM. No va en la página, es de referencia.

---

## Antes de empezar

- Creé una cuenta y un contenedor web en GTM (`Kevin Niño – Prueba Avianca` / `Prueba Avianca – Web`).
- Creé una propiedad nueva en GA4 con un flujo web.
- Trabajé en local con Live Server (`http://127.0.0.1:5500`) y al final publiqué la página en GitHub Pages.

**Cosas que me frenaron un poco:**
- El botón "Probar" de GTM no detecta la etiqueta en localhost, porque Google no puede entrar a mi equipo. Para validar usé la Vista previa.
- La sesión de Vista previa se desconecta cuando Live Server recarga la página. Toca volver a conectar.
- Al crear el flujo, GA4 me mostró un fragmento con otro ID `GTM-`. No lo instalé, porque cargaría un segundo contenedor y duplicaría datos. Lo único que necesitaba de ahí era el ID de medición `G-`.

---

## Ejercicio 1 – search_flight a GA4

**Etiqueta de Google**
- `GA4 - Etiqueta de Google`, con el ID `G-K4ZS4G19LN`.
- Se activa con `Initialization - All Pages`, para que GA4 cargue antes que cualquier otro evento.

**Variables**

Son variables de capa de datos (versión 2) que leen dentro de `ecommerce`:
- `DLV - flight_from` → `ecommerce.flight_from`
- `DLV - flight_to` → `ecommerce.flight_to`
- `DLV - trip_type` → `ecommerce.trip_type`
- `DLV - date_departure` → `ecommerce.date_departure`
- `DLV - adult_num` → `ecommerce.adult_num`

Al principio creé una variable de tipo "Elemento DOM", pero los datos del vuelo no están en el botón sino en el dataLayer, así que la cambié a variable de capa de datos.

**Activador**
- `CE - search_flight`: evento personalizado que se dispara solo con `search_flight`.

**Tag de GA4**
- `GA4 - Evento - search_flight`: evento `search_flight` con `flight_from`, `flight_to`, `trip_type`, `date_departure` y `adult_num`.

**Cómo lo probé**
- En la Vista previa de GTM revisé que el tag se disparara en `search_flight` y que las variables tuvieran valor.
- En el DebugView de GA4 revisé que llegara `search_flight` con sus parámetros, en ida y vuelta y en solo ida.
- Ojo: en DebugView el número que sale al lado de cada parámetro es cuántas veces llegó, no el valor. Para ver el valor hay que expandir el parámetro.

---

## Ejercicio 2 – flight_search

**Qué hace mi código** ([ver código](gtm/html-push-flight_search.html))

Es un tag de HTML personalizado (`HTML - push flight_search`) que se dispara con el mismo activador `CE - search_flight`:
1. Suma `adult_num + child_num + infant_num` y lo guarda como número en `total_passengers`.
2. Convierte `date_departure` de `20261118` a `2026-11-18`, cortando el texto por posiciones (año, mes y día).
3. Limpia `ecommerce` con un push de `ecommerce: null`.
4. Hace push de `flight_search` con `flight_from`, `flight_to`, `flight_type` (con el valor que traía `trip_type`), `total_passengers` y `departure_date`.

**Por qué lo hice así**
- Usé las variables de GTM (`{{DLV - ...}}`) dentro del código en vez de leer el dataLayer a mano, porque es más legible y reutilizo las variables del ejercicio 1.
- Mantuve la estructura dentro de `ecommerce` para que se parezca al evento original.
- Limpio `ecommerce` antes del push porque GTM fusiona los objetos. Si no lo hago, `flight_search` hereda campos viejos de `search_flight`, como `trip_type`, y el reemplazo que pide el ejercicio no se cumpliría del todo.

**Si llega un NA o falta un dato**
- Los pasajeros pasan por la función `toNumber`. Si el valor no es un número válido (`"NA"`, vacío, `undefined`), cuenta como 0.
- La fecha solo se formatea si tiene 8 caracteres (por ejemplo `20261118`). Si no, se manda `null` en lugar de inventar una fecha.

**Cómo evité duplicados y bucles**
- El tag escucha `search_flight` y emite `flight_search`. Como el nombre es distinto, el tag no se vuelve a disparar a sí mismo.
- Lo comprobé en Tag Assistant: en el evento `flight_search`, el tag de HTML sale como "No activada".
- El push de limpieza no tiene `event`, así que no dispara ningún activador.

**Lo que creé en GTM**
- Variables nuevas:
  - `DLV - child_num` → `ecommerce.child_num`
  - `DLV - infant_num` → `ecommerce.infant_num`
  - `DLV - total_passengers` → `ecommerce.total_passengers`
  - `DLV - departure_date` → `ecommerce.departure_date`
- Activador `CE - flight_search`.
- Tag `GA4 - Evento - flight_search` con `total_passengers`, `departure_date`, `flight_from` y `flight_to`.

**Problemas que tuve**
- GTM no me dejaba guardar el tag por usar `const`/`let`, porque en los tags de HTML personalizado su compilador solo acepta ES5. Lo cambié a `var` y concatenación con `+`.
- Me salía "variable desconocida" porque las variables tenían nombres mezclados (guion corto, guion largo y sin espacio). Las dejé todas con el mismo formato `DLV - nombre`.
- Al principio puse el tag de HTML con el activador de `flight_search`. Así nunca se disparaba, y además podía generar un bucle. Lo corregí a `CE - search_flight`.
- Al principio pensé que el tag no se disparaba, porque Tag Assistant selecciona el último evento (`flight_search`) y ahí sale "No activada". En el `search_flight` de justo antes sí se activa.
- `date_departure` (la fecha original) y `departure_date` (la fecha convertida) son variables distintas, y es fácil confundirlas.

**Cómo lo probé**
- Visor del dataLayer y consola: por cada clic aparece `search_flight`, luego el `ecommerce: null` y después `flight_search`.

| Búsqueda | total_passengers | departure_date | flight_type |
|---|---|---|---|
| Ida y vuelta | 3 | 2026-11-18 | Round Trip |
| Solo ida | 1 | 2026-11-18 | One Way |

- DebugView: llegan los dos eventos y `flight_search` trae sus parámetros.

---

## Ejercicio 3 – cookies

Está en el documento que mandé por correo.

---

## Supuestos

- `flight_type` en el evento nuevo lleva el valor de `trip_type` (`Round Trip` / `One Way`). El `flight_type: "Domestic"` original se descarta, porque el ejercicio pide reemplazar `trip_type` por `flight_type`.
- Si un número de pasajeros no llega o viene como `"NA"`, cuenta como 0.
- Si la fecha de salida no tiene el formato esperado, se manda `null`.
- `flight_search` no conserva los demás campos de `search_flight` (como `event_category` o `search_term`). Solo lleva lo que pide el ejercicio, para que quede limpio.