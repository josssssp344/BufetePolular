# Gestión de expedientes

Aplicación web de una sola página (`index.html`, sin librerías externas) para abrir, consultar, dar seguimiento, cerrar y **reabrir** expedientes. Los datos se guardan en el `localStorage` del navegador.

## 1. Ficha de análisis

**Problema.** Los asuntos de las personas atendidas se registran de forma dispersa (papel, hojas sueltas, chats). Esto provoca pérdida de información, falta de trazabilidad (quién hizo qué y cuándo) y cierres sin evidencia de trabajo previo. Además, cuando un caso cerrado debe retomarse, se tiende a crear un expediente nuevo y se pierde el vínculo con el historial anterior.

**Usuario.** Personal de un despacho u oficina de atención (responsables de caso y asistentes) que necesita registrar y consultar expedientes de forma rápida, sin instalar software ni contar con conexión a un servidor.

**Prioridades.**
1. Integridad de los datos: campos obligatorios válidos (sin vacíos ni solo espacios).
2. Trazabilidad: historial con fecha y hora automáticas que distingue actividades de cambios de estado, cierres y reaperturas.
3. Control del flujo: Abierto → En seguimiento → Cerrado, con cierre condicionado a una actividad previa y a un motivo, y reapertura solo desde «Cerrado» con motivo.
4. Conservación de la evidencia: ningún cierre ni dato previo se borra o sobrescribe al reabrir.
5. Consulta rápida: búsqueda por identificador o nombre y detalle inmediato.
6. Simplicidad: un solo archivo, interfaz clara y funcionamiento sin conexión.

## 2. Decisiones con uso razonado de la IA

| Decisión o necesidad | Aporte de la IA | Decisión del equipo y razón | Cómo se comprobó |
|---|---|---|---|
| Cómo validar los campos obligatorios y qué mensajes mostrar | Propuso validar con `trim()` y mensajes específicos por campo, en lugar de un único aviso genérico. | Se adoptó un mensaje descriptivo por campo, junto al campo y con borde de error. Así la persona sabe qué corregir sin adivinar. | Se intentó enviar el formulario vacío y con solo espacios en cada campo. Se mostró el mensaje correspondiente y no se creó el expediente. |
| **Reapertura de expedientes cerrados con trazabilidad completa** (retomar un caso sin perder su historia) | Propuso modelar la reapertura como un evento más del historial (solo se agregan registros, nunca se editan ni se borran), con etiquetas propias de «Cierre» y «Reapertura», y centralizar la regla en una función interna (`reabrir`) que valida el estado y el motivo, además de ocultar el formulario en la interfaz. | Se adoptó el historial de solo-agregar: al reabrir se conservan identificador, datos, actividades y todos los cierres previos con su fecha y motivo. El estado pasa a «En seguimiento» y se registra la reapertura con fecha, hora y motivo. La regla se aplica en doble capa (interfaz y lógica interna) para que no pueda evadirse. Un nuevo cierre mantiene las reglas originales y se agrega como evento nuevo, sin reemplazar los anteriores. | Se cerró un expediente y se reabrió con motivo vacío y con solo espacios (rechazado con mensaje). Se invocó `reabrir()` desde la consola sobre un expediente «Abierto» y otro «En seguimiento» (rechazado en ambos). Tras reabrir, se agregó una actividad y se cerró de nuevo: el historial mostró ambos cierres y la reapertura, cada uno con su fecha y motivo. Al recargar la página todo se mantuvo. |
| Cómo generar identificadores y persistir datos sin servidor | Propuso un contador correlativo guardado junto a los expedientes en `localStorage`, con formato `EXP-0001`, y datos de prueba si el almacenamiento está vacío. | Se guardó el contador en el propio objeto de datos (no se calcula por cantidad de elementos), para que los identificadores no se repitan. Se precargan 2 expedientes de demostración. | Se abrieron varios expedientes y se recargó la página: los datos se mantuvieron y los identificadores siguieron la secuencia sin repetirse. |

## 3. Lista de comprobación

| Recorrido | Resultado | Resultado observado |
|---|---|---|
| 1. Apertura | **Cumple** | Con campos vacíos o solo espacios se muestran errores descriptivos. Con datos válidos se crea el expediente con identificador correlativo (`EXP-0003`, `EXP-0004`...), fecha y hora automáticas y estado «Abierto». Aparece de inmediato en la tabla. |
| 2. Consulta | **Cumple** | La tabla muestra Identificador, Persona, Responsable y Estado. Filtrar por ID o nombre funciona; si no hay coincidencias aparece un mensaje claro. Al seleccionar una fila se despliega el detalle completo y el historial. |
| 3. Seguimiento y cierre | **Cumple** | Solo se pueden agregar actividades en expedientes «Abierto» o «En seguimiento», con fecha y hora automáticas. El botón pasa de «Abierto» a «En seguimiento». El cierre se rechaza sin actividades o sin motivo. Un expediente «Cerrado» queda bloqueado para actividades y cambios de estado. El historial distingue visualmente actividades de cambios de estado. |
| 4. Reapertura y trazabilidad | **Cumple** | Solo los expedientes «Cerrado» muestran «Reabrir expediente». Sin motivo (vacío o solo espacios) se muestra un error claro y no cambia el estado. Intentar reabrir uno «Abierto» o «En seguimiento» es rechazado por la interfaz (no aparece la acción) y por la lógica interna (`reabrir()` devuelve error). Al reabrir, el estado pasa a «En seguimiento», se conservan identificador, datos, actividades y cierres previos, y se agrega al historial un evento de reapertura con fecha, hora y motivo. Se pueden registrar nuevas actividades y cerrar de nuevo con las reglas originales; el segundo cierre se suma al primero sin reemplazarlo, y el bloque «Registro de cierres y reaperturas» los lista todos. |
