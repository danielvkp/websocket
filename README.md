## Implementación Websocket.

Se necesita el id de el usuario para crear un canal privado único, esto servirá para mostrar la notificación al usuario en caso de que este fuera del chat.

Evento Nuevo Mensaje:

```
App\Events\NuevoMensaje
```

Se subscribe a `chat.<idUsuario>`

Recibe dos parametros ($usuarioId, $fromId) donde el $usuarioId es para quien va el mensaje y $fromId es el id del usuario que ha enviado el mensaje (esto para que luego en el front puedan identificar de quien es el mensaje)

Nota: $fromId tambien puede ser la nueva variable $idChat que se creara, esto es solo para identificar quien ha enviado el mensaje.

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
