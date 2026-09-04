# TutorBot 🤖 - Sistema de Asesorías Académicas Automatizado

Bienvenidos al repositorio oficial de **TutorBot**, una solución integral de automatización para la gestión y asignación inteligente de tutorías académicas . Este proyecto ha sido desarrollado en la plataforma **n8n** utilizando un diseño arquitectónico distribuido, robustas reglas de negocio basadas en JavaScript, y un Agente de Inteligencia Artificial (Gemini) que actúa como asistente conversacional interactivo .

---

## 🎯 1. Definición del Problema y Estrategia

### El Problema 🛑
Tradicionalmente, la coordinación de asesorías académicas en la institución ha sido un proceso caótico, ineficiente y completamente manual . Los estudiantes dependen de correos electrónicos o mensajes informales para localizar a un tutor, mientras que los docentes carecen de una agenda centralizada para gestionar sus horarios disponibles . Esto genera:
*   Cruces y traslapes constantes de horarios .
*   Tutorías que no se concretan o se quedan sin atender .
*   Falta absoluta de trazabilidad y estadísticas para la dirección de la institución .

### La Estrategia 🚀
**TutorBot** ataca este problema mediante un flujo inteligente y conversacional guiado tipo **Wizard** (asistente paso a paso) que automatiza por completo el registro, agendamiento, consulta y cancelación de tutorías . 
*   **Orquestación en n8n:** n8n funciona como el coordinador y cerebro lógico que procesa el estado de sesión de cada usuario, gestiona el emparejamiento tutor-estudiante y despacha las notificaciones automáticas .
*   **Base de datos centralizada:** Se utiliza Google Sheets como base de datos organizada en pestañas estructuradas para el almacenamiento seguro y persistente de los datos .
*   **Control Matemático de Conflictos:** Un motor en JavaScript valida colisiones de agenda en tiempo real para eliminar por completo la duplicidad de reservas .

---

## 📂 2. Estructura de la Base de Datos (`TutorBot_DB`)

El modelo relacional se implementa sobre una hoja de cálculo nativa de Google Sheets . El archivo está compuesto por cinco pestañas estructuradas de la siguiente manera :

### Pestaña: `tutores` 👨‍🏫
Almacena el listado y estado de los docentes oficiales :
*   `id_tutor`: Identificador único numérico (ej. `1`).
*   `nombre`: Nombre completo del profesor (ej. `Javier Cuadros`).
*   `materia`: Asignatura que imparte (`Software`, `Inglés`, `Ser`).
*   `estado`: Disponibilidad actual del tutor (`Disponible`).

### Pestaña: `estudiantes` 🎓
Guarda el registro de los alumnos inscritos formalmente mediante el formulario :
*   `id_estudiante`: ID de chat único de Telegram del alumno (ej. `7635568207`).
*   `nombre_estudiante`: Nombres del estudiante.
*   `apellido_estudiante`: Apellidos del estudiante.

### Pestaña: `disponibilidad` ⏰
Define los bloques horarios de atención de cada tutor para el próximo lunes :
*   `id_disponibilidad`: Identificador del bloque (ej. `DIS001`).
*   `id_tutor`: ID del tutor asociado.
*   `dia_semana`: Día asignado (únicamente `LUNES`).
*   `hora_inicio`: Hora de inicio del bloque (ej. `6:00`).
*   `hora_fin`: Hora de finalización del bloque (ej. `11:00`).
*   `estado`: Estado del bloque (`LIBRE` / `OCUPADO`).

### Pestaña: `tutorias` 📋
Registra el historial completo de citas académicas agendadas en el sistema :
*   `id_tutoria`: Código único de referencia de la tutoría (ej. `TUT-9861`).
*   `id_estudiante`: ID de Telegram del estudiante que reservó.
*   `id_tutor`: ID del tutor asignado.
*   `materia`: Nombre de la asignatura agendada.
*   `fecha`: Fecha de la tutoría (formato limpio `YYYY-MM-DD`).
*   `hora_inicio`: Hora de inicio de la sesión.
*   `hora_fin`: Hora de fin de la sesión.
*   `estado`: Estado de la tutoría (`Asignada`, `Cancelada`).
*   `url_de_conexion`: Enlace dinámico al aula virtual (`link_here`).

### Pestaña: `secciones` (Control de Sesión) 💾
Fundamental para el funcionamiento del Enrutador de Estados (Traffic Cop) :
*   `usuario_telegram`: ID de chat único del usuario de Telegram.
*   `pantalla_actual`: Fase conversacional en la que se encuentra el estudiante (ej. `Menu`, `reinicio`).
*   `paso_actual`: Estado lógico para saber qué subflujo procesar (ej. `solicitando tutoria`, `Cancelando tutoria`, `eligiendo opciones`).
*   `datos_parciales`: JSON dinámico que almacena las elecciones temporales del estudiante en flujos multi-paso.

---

## ⚙️ 3. Arquitectura del Enrutador de Estados ("Traffic Cop")

Para evitar flujos gigantescos, desordenados y difíciles de depurar, se implementó el patrón arquitectónico **Traffic Cop** . El flujo principal (`validacion_usuario`) actúa como un policía de tránsito de datos que intercepta el mensaje entrante, consulta la base de datos de sesiones y decide qué subflujo ejecutar :

```
                     [ Estudiante escribe en Telegram ]
                                     │
                        [ validacion_usuario ] (Trigger)
                                     │
                       [ consulta_existencia en DB ]
                                    / \
                                  /     \
                       [ No Existe ]   [ Sí Existe ]
                             │                 │
               [ Envía Link Registro ]   [ consulta_seccion en DB ]
                             │                 │
               [ Formulario Registro ]   [ Switch: pantalla_actual ]
                                                │
               ┌───────────────────────┬────────┴──────────────┐
               ▼                       ▼                       ▼
      [ solicitando tutoria ] [ Cancelando tutoria ]          [ Menu ]
               │                       │                       │
     [ eligiendo_materia ]    [ procesar_anulacion ]      [ Switch1 (Opciones) ]
                                                        ┌──────┼──────┐
                                                        ▼      ▼      ▼
                                                       (1)    (2)    (3)
                                                        │      │      │
                                                     [Sol.] [Cons.] [Canc.]
```

### El proceso lógico es el siguiente :
1.  **Filtro de Entrada:** Cada mensaje de Telegram es capturado y consulta la pestaña `estudiantes` para validar si el ID existe . Si no existe, se desvía el flujo para enviarle el link al formulario de registro en la nube .
2.  **Evaluación de Sesión:** Si el estudiante está registrado, se consulta su `pantalla_actual` y `paso_actual` en la pestaña `secciones` .
3.  **Enrutamiento Inteligente:** Dependiendo de ese estado:
    *   Si el paso es `solicitando tutoria`: Se redirige el texto al subflujo de agendamiento (`eligiendo_materia`) .
    *   Si el paso es `Cancelando tutoria`: Se redirige al subflujo de anulación (`cancelar_tutoria` en modo de confirmación) .
    *   Si el paso está vacío o es `Menu`: Se asume que el usuario está en el menú principal y se evalúa el mensaje . Si digita `1`, `2` o `3`, se ejecutan las respectivas opciones lógicas; si escribe texto libre, el flujo desvía la conversación al **Agente de IA** .

---

## 🛠️ 4. Explicación Detallada de Cada Flujo

El sistema completo se encuentra distribuido de forma profesional en **11 flujos autónomos** que se interconectan dinámicamente para garantizar modularidad, facilidad de auditoría y máxima robustez:

### 4.1. Flujo Principal: `validacion_usuario` 🔌
*   **Función:** Orquestar el flujo inicial del bot, verificar la autenticación del alumno, consultar las sesiones activas en la base de datos y enrutar el mensaje hacia el subflujo correcto.
*   **Nodos Clave:**
    *   `usuario_escribe` (Telegram Trigger): Despierta el flujo al recibir cualquier interacción del estudiante.
    *   `consulta_existencia` (Google Sheets): Busca el Chat ID en la pestaña `estudiantes` de la base de datos.
    *   `If` / `formulario_registro`: Si el alumno no existe, le envía el formulario web único de n8n para registrarse.
    *   `consulta_seccion` (Google Sheets): Obtiene el estado conversacional del alumno desde la pestaña `secciones`.
    *   `Switch` (Enrutador de Estados): Redirige el flujo lógicamente a los flujos secundarios mediante el nodo `Execute Workflow`.

### 4.2. Flujo de Soporte: `formulario_registro` 📝
*   **Función:** Recibir los datos del formulario web de registro de n8n, agregarlos a la base de datos de estudiantes y dar inicio a su sesión de forma automática.
*   **Nodos Clave:**
    *   `On form submission` (Form Trigger): Formulario nativo de n8n que solicita Nombres y Apellidos del alumno, capturando de fondo su Chat ID de Telegram mediante un campo oculto (`idestudiante`).
    *   `Append row in sheet` (Google Sheets): Añade al alumno a la pestaña `estudiantes`.
    *   `Append row in sheet1` (Google Sheets): Crea el registro inicial en la pestaña `secciones` con estado de pantalla `Menu` para desbloquear el bot.
    *   *Mejora de UX (Redirección):* Configurada la opción nativa de redireccionamiento automático (`Redirect URL`) para enviar al estudiante de regreso al chat de Telegram con el bot tras presionar enviar.

### 4.3. Flujo de Soporte: `menu_usuario` 📱
*   **Función:** Registrar la sesión en estado `Menu` dentro de Google Sheets y enviar el mensaje de bienvenida y autogestión de manera estática.
*   **Nodos Clave:**
    *   `Append or update row in sheet` (Google Sheets): Setea la pantalla en `Menu` y el paso en `eligiendo opciones`.
    *   `Send a text message` (Telegram): Envía las opciones interactivas para que el alumno elija (1. Solicitar, 2. Consultar, 3. Cancelar).

### 4.4. Flujo de Soporte: `solicitar_tutoria` 🗓️
*   **Función:** Consultar el catálogo de materias configuradas en la base de datos de tutores y construir un menú dinámico y estético de selección en Telegram.
*   **Nodos Clave:**
    *   `Get row(s) in sheet` (Google Sheets): Lee todos los registros de la pestaña `tutores`.
    *   `Code in JavaScript` (Code): Recorre las materias configuradas y arma una lista numerada usando emojis (1️⃣ Software, 2️⃣ Inglés, etc.) de forma dinámica.
    *   `Send a text message` (Telegram): Envía el listado de selección al chat de Telegram del estudiante.

### 4.5. Flujo de Soporte: `eligiendo_materia` ⚠️ (Con Validación de Duplicidad)
*   **Función:** Procesar la materia que el alumno seleccionó, extraer la disponibilidad asignada para el tutor de dicha materia, validar matemáticamente que no existan cruces de horarios y registrar la tutoría.
*   **Nodos Clave:**
    *   `leer_tutores` (Google Sheets): Consulta los profesores de la base de datos.
    *   `Code in JavaScript` (Code): Asocia la opción numérica escrita por el alumno con el tutor y la asignatura seleccionada.
    *   `Get row(s) in sheet` (Google Sheets): Lee la disponibilidad horaria asociada a ese tutor en la pestaña `disponibilidad`.
    *   `Leer todas las Tutorías` (Google Sheets): Carga todo el historial de la pestaña `tutorias` con estado "Asignada".
    *   `Validar Cruce de Agenda` (Code in JavaScript): Script de lógica matemática avanzada de colisiones. Evalúa si el próximo lunes, en el horario del bloque elegido, el tutor o el estudiante ya tienen otra cita programada.
    *   `If` (Decisión):
        *   **Rama True (Existe Cruce):** Ejecuta `Resetear Estado por Duplicidad` en la sesión, cancela el agendamiento y envía un mensaje detallado a Telegram explicando que el tutor o el estudiante ya están ocupados en ese horario.
        *   **Rama False (Cita Libre):** Ejecuta `Append row in sheet` para guardar la cita en estado `Asignada` con un código de referencia dinámico generado de forma aleatoria (`TUT-XXXX`), asigna el enlace del aula virtual, resetea la sesión en `secciones` a `reinicio` y envía la tarjeta de confirmación al estudiante.

### 4.6. Flujo de Soporte: `consultar_tutoria` 🔎
*   **Función:** Consultar las tutorías activas agendadas en la base de datos de Google Sheets que le corresponden al estudiante que hace la petición, cruzando los nombres reales de los tutores asignados de manera exacta.
*   **Nodos Clave:**
    *   `Get row(s) in sheet` (Google Sheets): Lee las citas de la pestaña `tutorias` asociadas al Chat ID del usuario.
    *   `Get row(s) in sheet1` (Google Sheets): Trae el catálogo de tutores oficiales.
    *   `Code in JavaScript` (Code): Realiza un cruce lógico indexando los tutores por su ID y filtra de forma segura las tutorías activas (estado "Asignada"). Construye un reporte estructurado y elegante en Markdown con emojis.
    *   `Send a text message` (Telegram): Despacha el reporte consolidado directo al Telegram del estudiante.

### 4.7. Flujo de Soporte: `cancelar_tutoria` ❌ (Con Excepción de Cero Citas y Guardián Antiboqueo)
*   **Función:** Mostrar las tutorías activas del alumno para que digite el código de referencia (`TUT-XXXX`), validar el código antes de procesarlo, ofrecer una ruta de escape presionando `0` para evitar bloqueos y actualizar el estado de la cita a "Cancelada".
*   **Nodos Clave:**
    *   `Leer Tutorias Cancelar` (Google Sheets): Lee las tutorías asignadas del estudiante.
    *   `Code in JavaScript` (Code): Filtra las citas en estado "Asignada" y cuenta el total de registros activos.
    *   `If` (Verificación de Cero Tutorías):
        *   **Rama True (Tiene 0 citas):** Si el alumno no tiene tutorías agendadas para dar de baja, actualiza de inmediato su sesión a `Menu` y le notifica amistosamente en Telegram para evitar que su estado de conversación se bloquee.
        *   **Rama False (Tiene citas activas):** Envía el mensaje con la lista de tutorías y sus códigos `TUT-XXXX` solicitando la confirmación de la baja.
    *   `¿Es Cero?` (If): Nodo guardián que evalúa si el usuario escribió `0`. Si es así, activa el nodo `Resetear Sesión a Menú` y envía `Notificar Cancelación Anulada` de Telegram.
    *   `Leer Tutorías Confirmar` / `Validar Código Ingresado` (Code): Si no es cero, lee la DB y valida que el código sea real y pertenezca al usuario para evitar falsas bajas.
    *   `Update row in sheet` (Google Sheets): Busca la fila que coincida de forma exacta con el código escrito (`TUT-XXXX`) y cambia su estado a `cancelada` para liberar la agenda.

### 4.8. Flujo Programado: `reporte_diario_coordinacion` 🔔 (Recordatorios Diarios para Estudiantes)
*   **Función:** Ejecutarse de manera automática todos los días para enviar un recordatorio masivo e individualizado con su respectivo enlace de conexión a cada estudiante que tenga una clase asignada para el día de hoy.
*   **Nodos Clave:**
    *   `Schedule Trigger`: Disparador automático configurado de forma diaria a una hora específica de la mañana.
    *   `Leer Estudiantes` / `Leer Tutorias` / `Leer Tutores` (Google Sheets): Consulta y extrae las tres hojas de la base de datos.
    *   `Consolidar Reporte` (Code in JavaScript): Calcula la fecha actual en la zona horaria local, filtra las tutorías activas de hoy y asocia el nombre de los tutores. Retorna una lista con la información individualizada de cada estudiante.
    *   `Send a text message` (Telegram): Gracias a la iteración nativa por ítems de n8n, envía de forma secuencial y paralela un mensaje personalizado a cada estudiante a su Chat ID de Telegram correspondiente, sin necesidad de bucles complejos de programación.

### 4.9. Flujo Programado: `reporte_semanal_coordinacion` 📊 (Métricas de Gestión)
*   **Función:** Recopilar estadísticas acumuladas de actividad en el sistema y enviarlas semanalmente de forma consolidada al administrador o coordinación académica.
*   **Nodos Clave:**
    *   `Schedule Trigger`: Disparador automático programado para ejecutarse todos los lunes a las 08:00 AM.
    *   `Leer Tutorias` (Google Sheets): Extrae todo el historial de la pestaña `tutorias` para su procesamiento.
    *   `Calcular Estadísticas` (Code in JavaScript): Script que procesa los datos y calcula las siguientes métricas de gestión:
        *   Total de solicitudes de tutorías procesadas en la plataforma.
        *   Número de tutorías activas (estado "Asignada").
        *   Número de tutorías anuladas (estado "Cancelada").
        *   Desglose exacto de demanda por asignatura (`Software`, `Inglés` y `Ser`).
    *   `Send a text message` (Telegram): Envía el reporte analítico gerencial a un único Chat ID fijo (del administrador/coordinador) para evitar spam en los chats de los alumnos.

### 4.10. Flujo de Resiliencia: `TutorBot_Manejador_Errores` 🚨 (Manejador de Errores Global)
*   **Función:** Interceptar de forma centralizada cualquier fallo en tiempo de ejecución de cualquiera de los flujos de TutorBot y enviar alertas detalladas al administrador para su depuración inmediata.
*   **Nodos Clave:**
    *   `Error Trigger`: Disparador global que se activa automáticamente al fallar cualquier nodo de los flujos vinculados.
    *   `Send a text message` (Telegram): Envía un mensaje estructurado con el nombre del flujo afectado, el nodo donde ocurrió el error, el mensaje del fallo y el ID de ejecución para facilitar el soporte inmediato.

### 4.11. Módulo de IA: `AI Agent` (Fallback Inteligente) 🧠
*   **Función:** Interceptar mensajes de texto libre que no corresponden a opciones del menú tradicional (ej. "Hola", "¿quién da clases de Software?", "¿a qué hora atiende Pilar?") y responder cordialmente en lenguaje natural guiando al alumno a autogestionarse.
*   **Nodos Clave:**
    *   `AI Agent` (LangChain): El nodo inteligente central conectado al flujo principal.
    *   `Google Gemini Chat Model`: Motor inteligente configurado con el modelo `gemini-2.5-flash` para optimizar tokens y tiempo de respuesta.
    *   `Simple Memory` (Window Buffer Memory): Memoria selectiva de hasta 10 interacciones para recordar el contexto de la charla.
    *   `System Message`: Contiene el rol oficial de TutorBot, las reglas de asignación y las clases disponibles (Software, Inglés, Ser), además de directrices estrictas de formato para evitar errores de renderizado en Telegram.

## 🚀 5. Instrucciones de Despliegue y Uso

Sigue estos pasos para implementar TutorBot en tu propio entorno:

1.  **Clonar este repositorio** en tu cuenta de GitHub. Asegúrate de que el nombre del repositorio siga la estructura obligatoria de entrega: `Proyecto_TutorBot_ApellidoNombre` .
2.  **Crear tu bot en Telegram:**
    *   Busca a `@BotFather` en Telegram, ejecuta `/newbot`, define el nombre del bot y guarda el **token de la API** proporcionado .
3.  **Configurar la Base de Datos:**
    *   Crea una hoja de cálculo en Google Sheets llamada **`TutorBot_DB`** .
    *   Crea las pestañas `tutores`, `estudiantes`, `disponibilidad`, `tutorias`, y `secciones` con la estructura de columnas detallada en la Sección 2 .
    *   Comparte la base de datos con permisos de **Lectura** al correo electrónico del profesor/evaluador .
4.  **Importar los flujos en n8n:**
    *   Crea un nuevo proyecto en tu espacio de n8n .
    *   Crea flujos independientes para cada uno de los archivos JSON ubicados dentro de la carpeta `/workflow/` de este repositorio .
    *   En cada flujo importado, vincula tus propias **credenciales** seguras de Telegram Bot API y Google Sheets OAuth2 API .
5.  **Configurar el Formulario de Registro:**
    *   En el flujo `formulario_registro`, abre el nodo `On form submission` (Trigger Form), ve al apartado `Options` -> `Add Option` -> `Redirect URL` y pega el enlace de tu bot de Telegram (`https://t.me/TuNombreDeUsuarioDeTelegramBot?start=hola`) para lograr el retorno fluido y automático de los estudiantes al chat de Telegram .
6.  **Activar flujos:**
    *   Enciende el interruptor (Active) en todos los flujos de n8n para que queden escuchando eventos en tiempo de producción . ¡Disfruta de la automatización académica de TutorBot! 🎓🚀

---

## 📈 6. Resultados Esperados y Métricas de Éxito

*   **Reducción del 90%** en el tiempo promedio de coordinación y asignación de tutorías entre el estudiante y el docente .
*   **Trazabilidad total (100% de datos):** Registro detallado de solicitudes, cancelaciones, tutores y horarios asignados con códigos de auditoría estables .
*   **Disponibilidad 24/7:** El bot de Telegram y el Agente de IA atienden y asisten a los estudiantes en cualquier momento del día, agilizando el flujo académico .
*   **Cero Cruces de Agenda:** Garantía absoluta de que ningún tutor o alumno sufrirá de traslapes horarios gracias a la validación matemática integrada en JavaScript .
