### TutorBot 🤖 - Sistema de Asesorías Académicas Automatizado
**Solución optimizada presentada por: Exneider Nava**
Bienvenidos al repositorio oficial de  **TutorBot** , una solución integral de automatización para la gestión y asignación inteligente de tutorías académicas. Este proyecto ha sido desarrollado en la plataforma  **n8n**  utilizando un diseño arquitectónico distribuido, robustas reglas de negocio basadas en JavaScript, y un Agente de Inteligencia Artificial (Gemini) que actúa como asistente conversacional interactivo.

--------------------------------------------------------------------------------

#### 🚀 ¡Pruébalo ahora en vivo!
Si quieres probar el funcionamiento de este flujo de forma inmediata, puedes interactuar directamente con el bot en Telegram ingresando al siguiente enlace:
👉 [t.me/exneider_bot](https://t.me/exneider_bot)

--------------------------------------------------------------------------------

#### 🎯 1. Definición del Problema y Estrategia
##### El Problema 🛑
Tradicionalmente, la coordinación de asesorías académicas en la institución ha sido un proceso caótico, ineficiente y completamente manual. Los estudiantes dependen de correos electrónicos o mensajes informales para localizar a un tutor, mientras que los docentes carecen de una agenda centralizada para gestionar sus horarios disponibles. Esto genera:
*  Cruces y traslapes constantes de horarios.
*  Tutorías que no se concretan o se quedan sin atender.
*  Falta absoluta de trazabilidad y estadísticas para la dirección de la institución.

##### La Estrategia 🚀
**TutorBot**  ataca este problema mediante un flujo inteligente y conversacional guiado tipo  **Wizard**  (asistente paso a paso) que automatiza por completo el registro, agendamiento, consulta y cancelación de tutorías.
*   **Orquestación en n8n:**  n8n funciona como el coordinador y cerebro lógico que procesa el estado de sesión de cada usuario, gestiona el emparejamiento tutor-estudiante y despacha las notificaciones automáticas.
*   **Base de datos centralizada:**  Se utiliza Google Sheets como base de datos organizada en pestañas estructuradas para el almacenamiento seguro y persistente de los datos.
*   **Control Matemático de Conflictos:**  Un motor en JavaScript valida colisiones de agenda en tiempo real para eliminar por completo la duplicidad de reservas.

--------------------------------------------------------------------------------

#### 📂 2. Estructura de la Base de Datos
Toda la persistencia de datos de los tutores, el registro de estudiantes, la disponibilidad horaria de cada clase, las tutorías agendadas y el control dinámico de las sesiones en vivo se gestionan a través de una base de datos centralizada en Google Sheets.
👉 [Ingrese aquí para ver la estructura de la base de datos](https://docs.google.com/spreadsheets/d/1xbT4dwdOO3NaEmIXSQr5zSATfRVcMRbENJRz2PyWZhE/edit?usp=sharing)

--------------------------------------------------------------------------------

#### ⚙️ 3. Arquitectura y Lógica de Sesiones
Para mantener un sistema ordenado y evitar flujos gigantescos difíciles de mantener en n8n, implementamos un control de estado de sesión centralizado en la pestaña secciones de Google Sheets.
El proceso es sumamente sencillo e intuitivo:
1.  **Filtro de Entrada:**  Cada vez que escribes al bot, el sistema consulta si ya te encuentras registrado como estudiante. Si no existes, te desvía inmediatamente al formulario de registro en línea.
2.  **Control de Estados:**  Si ya existes, lee tu sesión activa de la base de datos. El bot sabe exactamente si estás navegando en el Menú Principal, si estás solicitando una tutoría, o si estás en proceso de cancelar una asesoría.
3.  **Enrutamiento:**  Con base en ese estado registrado, redirige automáticamente tu mensaje al subflujo secundario correspondiente para procesarlo. Si escribes cualquier texto libre fuera de las opciones numéricas, un Agente de IA intercepta el mensaje para guiarte cordialmente en lenguaje natural.

--------------------------------------------------------------------------------

#### 🛠️ 4. Explicación Detallada de Cada Flujo
El sistema completo se encuentra distribuido de forma profesional en  **múltiples flujos autónomos**  que se interconectan para garantizar modularidad y mantenibilidad:

##### 4.1. Flujo Principal: validacion_usuario 🔌
*   **Función:** Orquestar el flujo inicial del bot, verificar la autenticación del alumno, consultar las sesiones activas en la base de datos y enrutar el mensaje hacia el subflujo correcto.
*   **Nodos Clave:**
    *   `usuario_escribe` (Telegram Trigger): Despierta el flujo al recibir cualquier interacción del estudiante.
    *   `consulta_existencia` (Google Sheets): Busca el Chat ID en la pestaña `estudiantes`.
    *   `If` / `formulario_registro`: Si el alumno no existe, le envía el formulario web único de n8n para registrarse.
    *   `consulta_seccion` (Google Sheets): Obtiene el estado conversacional del alumno desde la pestaña `secciones`.
    *   `Switch` (Enrutador de Estados): Redirige el flujo lógicamente a los flujos secundarios mediante el nodo `Execute Workflow`.

<img width="1853" height="871" alt="image" src="https://github.com/user-attachments/assets/7925968b-ae2a-422f-889b-2aeb387e25c8" />


##### 4.2. Flujo de Soporte: formulario_registro 📝
*   **Función:** Recibir los datos del formulario web de registro de n8n, agregarlos a la base de datos de estudiantes y dar inicio a su sesión.
*   **Nodos Clave:**
    *   `On form submission` (Form Trigger): Formulario nativo de n8n que solicita Nombres y Apellidos del alumno, capturando de fondo su Chat ID de Telegram mediante un campo oculto (`idestudiante`).
    *   `Append row in sheet` (Google Sheets): Añade al alumno a la pestaña `estudiantes`.
    *   `Append row in sheet1` (Google Sheets): Crea el registro inicial en la pestaña `secciones` con estado de pantalla `Menu` para desbloquear el bot.
    *   *Mejora de UX (Redirección):* Configurada la opción nativa `Redirect URL` para enviar al estudiante automáticamente de regreso al chat de Telegram al dar clic en enviar.

<img width="1851" height="838" alt="image" src="https://github.com/user-attachments/assets/2a6c8a35-62d0-40d0-a4e3-3202d49c8929" />


##### 4.3. Flujo de Soporte: menu_usuario 📱
*   **Función:** Registrar la sesión en estado `Menu` dentro de Google Sheets y enviar el mensaje de bienvenida y autogestión de manera estática.
*   **Nodos Clave:**
    *   `Append or update row in sheet` (Google Sheets): Setea la pantalla en `Menu` y el paso en `eligiendo opciones`.
    *   `Send a text message` (Telegram): Envía las opciones interactivas para que el alumno elija (1. Solicitar, 2. Consultar, 3. Cancelar).

<img width="1852" height="888" alt="image" src="https://github.com/user-attachments/assets/d5af91c3-c39f-45c9-bf50-8f4b6224a387" />


##### 4.4. Flujo de Soporte: solicitar_tutoria 🗓️
*   **Función:** Consultar el catálogo de materias configuradas en la base de datos de tutores y construir un menú dinámico y estético para Telegram.
*   **Nodos Clave:**
    *   `Get row(s) in sheet` (Google Sheets): Lee todos los registros de la pestaña `tutores`.
    *   `Code in JavaScript` (Code): Recorre las materias configuradas y arma una lista numerada usando emojis (1️⃣ Software, 2️⃣ Inglés, etc.) de forma dinámica.
    *   `Send a text message` (Telegram): Envía el listado de selección al chat de Telegram del estudiante.

<img width="1850" height="893" alt="image" src="https://github.com/user-attachments/assets/13ff6a4c-05ae-4954-8596-deaecb0ed087" />


##### 4.5. Flujo de Soporte: eligiendo_materia ⚠️ (Con Validación de Duplicidad)
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

<img width="1845" height="851" alt="image" src="https://github.com/user-attachments/assets/15f9b12a-780a-42ae-80a9-93c9f630fc00" />


##### 4.6. Flujo de Soporte: consultar_tutoria 🔎
*   **Función:** Consultar las tutorías activas agendadas en la base de datos de Google Sheets que le corresponden al estudiante que hace la petición, cruzando los nombres reales de los tutores asignados.
*   **Nodos Clave:**
    *   `Get row(s) in sheet` (Google Sheets): Lee las citas de la pestaña `tutorias` asociadas al Chat ID del usuario.
    *   `Get row(s) in sheet1` (Google Sheets): Trae el catálogo de tutores oficiales.
    *   `Code in JavaScript` (Code): Realiza un cruce lógico indexando los tutores por su ID y filtra de forma segura las tutorías activas (estado "Asignada"). Construye un reporte de texto estructurado y elegante en Markdown con emojis.
    *   `Send a text message` (Telegram): Despacha el reporte consolidado directo al Telegram del estudiante.

<img width="1853" height="897" alt="image" src="https://github.com/user-attachments/assets/99123a95-1d66-46b9-9a7f-56990c2e9ad4" />


##### 4.7. Flujo de Soporte: cancelar_tutoria ❌ (Con Excepción de Cero Citas y Guardián "0")
*   **Función:** Mostrar las tutorías activas del alumno para que digite el código de referencia (`TUT-XXXX`), actualizar el estado de la cita a "Cancelada" y restablecer su sesión, incluyendo un guardián inteligente que valida códigos reales y una opción de pánico (`0`) para anular la acción de forma segura.
*   **Nodos Clave:**
    *   `Switch` (Operación): Evalúa la acción (`listar` para mostrar la lista o `confirmar` para dar de baja).
    *   `¿Es Cero?` (If): Evalúa si el usuario escribió `0` para salir de emergencia, en cuyo caso ejecuta `Resetear Sesión a Menú` y envía un mensaje de confirmación de anulación limpia sin alterar datos.
    *   `Leer Tutorías Confirmar` (Google Sheets): Si el estudiante ingresó un código, lee sus citas activas para comprobar la validez.
    *   `Validar Código Ingresado` (Code in JavaScript): Compara el texto ingresado con los códigos de tutorías activas asignadas para evitar falsas cancelaciones o errores lógicos.
    *   `¿Código es Válido?` (If):
        *   **Rama True:** Llama a `Update row in sheet` para marcar el estado como `cancelada` y actualiza la sesión a `reinicio` enviando la confirmación de éxito.
        *   **Rama False:** Mantiene el estado `Cancelando tutoria` y dispara `Notificar Código Inválido` para indicarle al alumno el error y recordarle el botón de escape `0`.

<img width="1866" height="871" alt="image" src="https://github.com/user-attachments/assets/607c9437-27ec-4a89-8502-725cf7358619" />


##### 4.8. Flujo Programado: reporte_diario_coordinacion 🔔 (Recordatorios Diarios)
*   **Función:** Ejecutarse de manera automática todos los días para enviar un recordatorio masivo e individualizado con su respectivo enlace de conexión a cada estudiante que tenga una clase asignada para el día de hoy.
*   **Nodos Clave:**
    *   `Schedule Trigger`: Disparador automático configurado de forma diaria a una hora específica de la mañana.
    *   `Leer Estudiantes` / `Leer Tutorias` / `Leer Tutores` (Google Sheets): Consulta y extrae las tres hojas de la base de datos.
    *   `Consolidar Reporte` (Code in JavaScript): Calcula la fecha actual en la zona horaria local, filtra las tutorías activas de hoy y asocia el nombre de los tutores. Retorna una lista con la información individualizada de cada estudiante.
    *   `Send a text message` (Telegram): Gracias a la iteración nativa por ítems de n8n, envía de forma secuencial y paralela un mensaje personalizado a cada estudiante a su Chat ID de Telegram correspondiente, sin necesidad de bucles complejos de programación.

<img width="1856" height="865" alt="image" src="https://github.com/user-attachments/assets/3bd2a88e-07ba-409b-890a-b71558f77318" />


##### 4.9. Flujo Programado: reporte_semanal_coordinacion 📊 (Métricas de Gestión)
*   **Función:** Recopilar estadísticas acumuladas de actividad en el sistema y enviarlas semanalmente de forma consolidada al administrador o coordinación académica.
*   **Nodos Clave:**
    *   `Schedule Trigger`: Disparador automático programado para ejecutarse todos los lunes a las 08:00 AM.
    *   `Leer Tutorias` (Google Sheets): Extrae todo el historial de la pestaña `tutorias` para su procesamiento.
    *   `Calcular Estadísticas` (Code in JavaScript): Script que procesa los datos y calcula las siguientes métricas de gestión:
        *   Total de solicitudes procesadas.
        *   Número de tutorías activas (estado "Asignada").
        *   Número de tutorías anuladas (estado "Cancelada").
        *   Desglose exacto de demanda por asignatura (`Software`, `Inglés` y `Ser`).
    *   `Enviar Reporte` (Telegram): Envía el reporte analítico gerencial a un único Chat ID fijo (del administrador/coordinador) para evitar spam en los chats de los alumnos.

<img width="1848" height="892" alt="image" src="https://github.com/user-attachments/assets/376cbb67-bdca-4927-bc5d-c7d737fc58cb" />


##### 4.10. Flujo de Resiliencia: TutorBot_Manejador_Errores 🚨 (Manejador Global de Errores)
*   **Función:** Interceptar de manera centralizada y asíncrona cualquier error en tiempo de ejecución ocurrido en cualquiera de los otros subflujos del sistema, enviando una alerta detallada al administrador para garantizar resiliencia absoluta de producción.
*   **Nodos Clave:**
    *   `Error Trigger`: Disparador global que se activa automáticamente al fallar un nodo.
    *   `Send a text message` (Telegram): Despacha una tarjeta detallada al Chat ID del administrador con el nombre del flujo fallido, el nodo culpable, el ID de ejecución y el mensaje de error de la API o sistema.

<img width="1847" height="892" alt="image" src="https://github.com/user-attachments/assets/e4918f7d-1083-4bbb-ac22-66fe4d9c854c" />

--------------------------------------------------------------------------------

#### 🚀 5. Instrucciones de Despliegue y Uso
Sigue estos pasos para implementar TutorBot en tu propio entorno:
1.  **Clonar este repositorio**  en tu cuenta de GitHub. Asegúrate de que el nombre del repositorio siga la estructura obligatoria de entrega: Proyecto_TutorBot_ApellidoNombre.
2.  **Crear tu bot en Telegram:**
    *  Busca a @BotFather en Telegram, ejecuta /newbot, define el nombre del bot y guarda el  **token de la API**  proporcionado.
3.  **Configurar la Base de Datos:**
    *  Crea una hoja de cálculo en Google Sheets llamada  **TutorBot_DB** .
    *  Crea las pestañas tutores, estudiantes, disponibilidad, tutorias, y secciones con la estructura correspondiente.
    *  Comparte la base de datos con permisos de  **Lectura**  al correo electrónico del profesor/evaluador.
4.  **Importar los flujos en n8n:**
    *  Crea un nuevo proyecto en tu espacio de n8n.
    *  Crea flujos independientes para cada uno de los archivos JSON ubicados dentro de la carpeta /workflow/ de este repositorio.
    *  En cada flujo importado, vincula tus propias  **credenciales**  seguras de Telegram Bot API y Google Sheets OAuth2 API.
5.  **Configurar el Formulario de Registro:**
    *  En el flujo formulario_registro, abre el nodo On form submission (Trigger Form), ve al apartado Options -> Add Option -> Redirect URL y pega el enlace de tu bot de Telegram (https://t.me/TuNombreDeUsuarioDeTelegramBot?start=hola) para lograr el retorno fluido y automático de los estudiantes al chat de Telegram.
6.  **Activar flujos:**
    *  Enciende el interruptor (Active) en todos los flujos de n8n para que queden escuchando eventos en tiempo de producción. ¡Disfruta de la automatización académica de TutorBot! 🎓🚀

--------------------------------------------------------------------------------

#### 📈 6. Resultados Esperados y Métricas de Éxito
*   **Reducción del 90%**  en el tiempo promedio de coordinación y asignación de tutorías entre el estudiante y el docente.
*   **Trazabilidad total (100% de datos):**  Registro detallado de solicitudes, cancelaciones, tutores y horarios asignados con códigos de auditoría estables.
*   **Disponibilidad 24/7:**  El bot de Telegram y el Agente de IA atienden y asisten a los estudiantes en cualquier momento del día, agilizando el flujo académico.
*   **Cero Cruces de Agenda:**  Garantía absoluta de que ningún tutor o alumno sufrirá de traslapes horarios gracias a la validación matemática integrada en JavaScript.
