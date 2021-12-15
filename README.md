# Integrar websocket al front.

Agregar las siguientes dependencias.

```
"laravel-echo": "^1.11.3",
"pusher-js": "^7.0.3",
```


# Realizar la primera conexion desde vue

esto debe hacer en el primer lugar donde cargue las librerias de inicio, en caso de usar vue-cli

hacerlo desde `main.js` o el punto de entrada de la aplicacion, yo crearia un archivo `setup_socket.js` y lo importaria en el main, asi te queda mas separado

el siguiente codigo es el que va a realizar la conexion al socket, solo la conexion.

el valor de key debe ser la misma que esta en el servidor en

"config/websockets.php"

wsHost: window.location.hostname,
wsPort: 6001,

Esto va a depender del servidor.

```

import Echo from 'laravel-echo'

window.Pusher = require('pusher-js')

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
});

```

En consola siempre que dice connected es porque el socket esta funcionando correctamente

# Canales disponibles

En vue va a depender de como lo tengas, lo mejor seria referenciar Echo dentro del objeto data, si tienes que renderizar el mismo componente varias veces

```
{
  echo: Echo,
}
```

Escuchar canal: recuperar el id de usuario del storage o de vuex, y pasarlo al canal

# Canal privado, este canal es privado para cada usuario. solo va a recibir data cuando alguien le envie un mensaje.

```
this.echo.channel(`chat.${<aqui va el id del usaurio logeado>}`).listen('NuevoMensaje', (e) => {

    console.log('nuevo_mensaje_recibido ** puede ser de cualquier chat');

    /*
    aqui se debe comprobar que el mensaje no sea de el mismo - cuando este enviando desde el canal privado

    en  e.data va a llegar el id de quien envio el mensaje
    */

    /*ejemplo de comprobacion*/

    if (this.credenciales.userId != e.data) {
        this.contador_mensajes++
    }
});
```

# Canal privado entre medico y paciente

Aqui se va a escuchar un nuevo canal privado entre el medico y paciente, para eso necesitas tener la asociacion del chat.

```
this.echo.channel(`chat_privado.${<id_asociadion>}`).listen('MensajePrivado', data => {
   /*lo unico que hace es recibir el mensaje en este canal*/
    this.mensajes.push(data.data)
});
```

# Nota

Cerrar este canal (solo este, la otra debe permanecer abierta siempre) cuando se cambie de chat.

```
this.echo.leaveChannel(`chat_privado.${<chat_anterior>}`);
```
