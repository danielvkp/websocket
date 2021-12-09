## Implementación Websocket.

Se necesita el id de el usuario para crear un canal privado único, esto servirá para mostrar la notificación al usuario en caso de que este fuera del chat.

Evento Nuevo Mensaje:

```
App\Events\NuevoMensaje
```

Se subscribe a `chat.<idUsuario>`

Recibe dos parametros ($usuarioId, $fromId) donde el $usuarioId es para quien va el mensaje y $fromId es el id del usuario que ha enviado el mensaje (esto para que luego en el front puedan identificar de quien es el mensaje)

Nota: `$fromId` tambien puede ser la nueva variable $idChat que se creara, esto es solo para identificar quien ha enviado el mensaje.

Evento Nuevo Mensaje:

```
App\Events\MensajePrivado
```

Aqui el usuario al abrir el chat (que el front debe identificar con la llamada previa) se debera subscribir a un canal privado `chat_privado.<idChat>`

Al enviar un mensaje, despues de guardarlo en la base de datos, se disparan dos eventos

```
broadcast(new NuevoMensaje(<id del usuario que recibe el mensaje>, <id del usuario que ha enviado el mensaje>));

broadcast(new MensajePrivado($mensaje, <idChat>));
```

`$mensaje` debe ser de tipo `App\Models\Mensaje`

## Parte Frontend

Configurarlo de acuerdo al stack que este usando.
en mi caso lo he puesto asi

```
window.Pusher = require('pusher-js');

window.Echo = new Echo({
    broadcaster: 'pusher',
    key: 'my-key',
    forceTLS: false,
    wsHost: window.location.hostname,
    wsPort: 6001,
    disableStats: true,
});

window.Echo.connector.pusher.connection.bind('connected', () => {
    console.log('connected');
    /*esto es para comprobar que hizo conexion al socket*/
});
```

## Subscribirse y recibir

Esto debe ejecutarse apenas el usuario hace login, asi se crea su canal y empieza a recibir data

```
Echo.channel(`chat.<idUsuarioAutenticado>`).listen('NuevoMensaje', (e) => {
    console.log('nuevo_mensaje_recibido');
    /*comprobar de quien ha sido el mensaje para mostar la notificación*/
});

```


## Chat Privado

Aqui se subscribe al chat entre paciente y medico

```
Echo.channel(`chat_privado.<idChat>`).listen('MensajePrivado', data => {
    /* dentro de data estara la informacion del mensaje en json, aqui ya puede agregarlo */


    /* es necesario comprobar si el mensaje tiene el campo adjunto diferente de null para crear un link que permita descargar el archivo */

    /*ejemplo: this.mensajes.push(data.data)*/
});

```

Luego en el componente que muestra el mensaje solo debera comprobar si hay adjunto con v-if y mostrar el link de descarga.

# Supervisor

Esto es mejor dejarlo para cuando el proyecto este muy terminado, en caso de que haya que tocar alguna configuracion no tener que mover todo de nuevo

## Instalacion supervisor
```
apt install supervisor
```

## Configurarlo

nano `/etc/supervisor/conf.d`

```
[program:websockets]
command=/usr/bin/php /<ruta_proyecto> artisan websockets:serve
numprocs=1
autostart=true
autorestart=true
user=laravel-echo
```

En mi caso tengo

```
[program:websockets]
command=/usr/bin/php /var/www/html/proyecto_web artisan websockets:serve
numprocs=1
autostart=true
autorestart=true
user=laravel-echo
```

Luego

`supervisorctl update`
`supervisorctl start websockets`

Con esto en caso de algun fallo el servidor automaticamente volvera a correr el socket.
