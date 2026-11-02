# TempQuest para Android

Proyecto Android nativo (APK) para monitorear varios servidores, leídos desde `/api/status` del servidor Flask. La pantalla principal muestra un servidor a la vez, inicialmente el más caliente. La flecha despliega una lista desplazable de servidores locales y el botón para agregar otro. El alta solicita el ID de un dispositivo activo ya existente en MySQL y un nombre opcional (por defecto, Servidor N); el vínculo y el nombre se guardan únicamente en el teléfono y no escriben en MySQL. La barra inferior tiene **Inicio** y **Alertas**. Alertas muestra los incidentes del servidor cuando está disponible y conserva localmente en el teléfono las pruebas, con fecha, hora, servidor y valor. El máximo permitido es 80 °C. El botón **Probar alarma · 84 °C** prueba el sonido, la vibración y la pantalla de alarma, y guarda la prueba en el teléfono; no modifica MySQL.

Mientras la API no entregue lecturas reales válidas para ambos servidores, la app muestra una vista marcada como **MODO PRUEBA** con 84 °C para el servidor 1 y 65 °C para el servidor 2. Estas lecturas de vista previa no generan alertas automáticas. Cuando estén disponibles las dos lecturas reales, la app las muestra y ordena automáticamente por temperatura.

El botón de sol y luna junto al logo alterna entre los temas claro y oscuro, y recuerda la selección en el teléfono.

## Abrir y compilar en Android Studio

1. En Android Studio, abre la carpeta `tempquest-android`.
2. Acepta **Sync Project with Gradle Files** y la descarga de componentes si la solicita.
3. Para el emulador, deja el servidor ejecutándose en la computadora. La app usa por defecto `http://10.0.2.2:5000`.
4. Pulsa Run para instalarla en el emulador. Para obtener el archivo, elige **Build > Build APK(s)**. El APK de depuración queda bajo `app/build/outputs/apk/debug/`.

Toca la etiqueta de estado de conexión en la cabecera para cambiar la dirección del servidor. En un teléfono físico se debe usar una dirección del servidor accesible desde ese teléfono (por ejemplo, `http://192.168.1.10:5000`); para uso fuera de la red local se requiere un dominio HTTPS y acceso remoto seguro.

## Notificación con la app en segundo plano o cerrada

El servidor consulta MySQL cada cinco segundos. Al cruzar el umbral registra un incidente en `instance/alerts.sqlite3` y lo envía mediante Firebase Cloud Messaging (FCM). Cuando el teléfono recibe la alerta, Android inicia una alarma con sonido repetido y vibración; se detiene desde la notificación. El servidor registra un solo incidente por cada episodio sobre el límite y vuelve a armarse cuando la lectura baja del umbral.

Para habilitar FCM hacen falta credenciales de un proyecto Firebase de tu propiedad:

1. En Firebase Console, crea un proyecto y agrega una aplicación Android con el identificador `com.tempquest.monitor`.
2. Descarga `google-services.json` y colócalo en `app/google-services.json`.
3. Activa Firebase Cloud Messaging y compila nuevamente desde Android Studio.
4. Genera una clave de cuenta de servicio para Firebase Admin SDK en el mismo proyecto. En la computadora siempre encendida define `GOOGLE_APPLICATION_CREDENTIALS` con la ruta a esa clave antes de ejecutar `app.py`. No copies esa clave al teléfono ni la compartas en el chat.
5. Reinicia el servidor. Al abrir la app, el teléfono registra su token y queda habilitado para recibir las alertas.

Sin Firebase configurado, la pantalla puede conectarse al servidor y mostrar las mediciones e historial; **PROBAR ALARMA** comprueba la alarma local, pero el envío remoto todavía no está habilitado. Acepta el permiso de notificaciones en Android y deja habilitada la categoría **Alarmas de temperatura**.

El servidor debe permanecer encendido, conectado a MySQL e Internet. El teléfono necesita red y permiso de notificaciones. Android puede silenciar sonido o vibración con No molestar, ahorro extremo de batería o permisos revocados; ningún APK puede garantizar avisos si el equipo está apagado, sin conexión o si la app fue forzada a detenerse.

## API

- `GET /api/status`: última lectura de cada dispositivo activo y umbral.
- `GET /api/history`: lecturas recientes para el gráfico.
- `GET /api/alerts`: incidentes persistidos.
- `POST /api/register-device`: registra el token FCM del teléfono.

Antes de publicar el servidor en Internet, agrega autenticación y HTTPS; la configuración inicial está pensada para una prueba controlada en la red local.
