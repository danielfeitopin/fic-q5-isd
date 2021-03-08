# Diseño e Implementación de Servicios RPC

## El Modelo RPC

RPC (_Remote Procedure Call_):

- Un proceso expone su funcionalidad como una serie de procedimientos que pueden ser invocados desde cualquier aplicación en cualquier punto de la red.
- El objetivo que persigue es que para el programador sea igual de sencillo invocar una operación de una aplicación remota que invocar una operación de una librería local.
- Se basa en generar automáticamente código que se encarga de abstraer al programador de los detalles de crear y parsear mensajes, y de enviarlos/recibirlos por la red.
- Primera implementación popular: SunRPC (80’s).
  - Desde entonces ha habido múltiples implementaciones: RMI, Corba, SOAP, Apache Thrift, gRPC...

Modo de Funcionamiento:

- El programador de la aplicación servidora.
- Modela la funcionalidad ofrecida a través de un conjunto de operaciones que pueden ser invocadas por los clientes.
- Describe las operaciones expuestas en un fichero de definición utilizando un lenguaje especial. Para cada operación se indica:
  - Nombre de la operación.
  - Nombre y tipo de los parámetros de entrada.
  - Nombre y tipo de los parámetros de salida o valores de retorno.
- Ejecuta un compilador especial del _framework_ RPC que recibe como entrada el fichero de definición y genera automáticamente el `skeleton`, formado por:
  - Un código fuente plantilla con un procedimiento vacío por cada operación, o una interfaz con las operaciones.
  - Una serie de librerías que se ocuparán de la comunicación con los clientes.
- Implementa las operaciones del código fuente plantilla o la interfaz.

El programador de la aplicación cliente:

- Ejecuta el compilador especial del _framework_ RPC pasándole como entrada el fichero de definición.
  - Genera automáticamente una librería local llamada `stub`.
  - El `stub` ofrece al programador del cliente operaciones con la misma firma que las indicadas en el fichero de definición.
  - Cada operación del `stub`, al ser invocada, se ocupa de los detalles de invocar la operación correspondiente en el servicio y obtener la respuesta.
- Programa su aplicación cliente normalmente, invocando a la operación correspondiente del `stub` cada vez que necesita invocar una operación remota.

Flujo de una invocación remota:

1. La aplicación cliente invoca una operación del `stub` pasándole los parámetros necesarios.
2. El `stub`:
   - Genera un mensaje encapsulando el nombre de la operación y los parámetros de entrada recibidos.
   - Envía el mensaje por la red al servidor y espera la respuesta.
3. El `skeleton`, cuando recibe el mensaje del `stub`, comprueba en el mensaje qué operación se desea invocar, extrae los parámetros de entrada, e invoca la operación correspondiente del código fuente plantilla (o de la implementación de la interfaz).
4. La implementación del código fuente plantilla (o de la interfaz) convierte los parámetros recibidos a los tipos adecuados.
5. La operación de la capa Modelo se ejecuta y devuelve un resultado.
6. La implementación del código fuente plantilla (o de la interfaz) convierte el resultado de la capa Modelo a los tipos con los que trabajan las operaciones del código fuente plantilla (o de la interfaz) y lo devuelve.
7. El `skeleton`:
   - Genera un mensaje encapsulando el resultado recibido.
   - Envía el mensaje por la red al cliente.
8. El `stub`: Al recibir la respuesta del servidor, extrae de la misma el resultado y se lo devuelve al cliente.
   - El programa cliente recibe la respuesta a su invocación y continua su ejecución.

Existe un compilador del _framework_ RPC para cada lenguaje de programación soportado.

Cada compilador genera los stubsy los skeletonsusando la sintaxis y tipos de datos del lenguaje objetivo. De esta manera, las aplicaciones cliente y servidora pueden escribirse en lenguajes diferentes.

El Modelo RPC ha sido muy exitoso:

- Uso muy intuitivo para un programador.
- Transparencia de la red para los programadores.

Sin embargo, en algunas implementaciones existe un alto acoplamiento cliente–servidor.

- Cambios en la interfaz obligan a regenerar el `stub`.
- El servidor puede ser una aplicación autónoma que no está bajo nuestro control.
- Este problema no existe en las implementaciones modernas del paradigma RPC.

## REST vs RPC

En el modelo RPC:

- No existe el concepto de identificador global.
- Normalmente los identificadores son locales a cada servicio.
- No se soportan intermediaros transparentes, ya que la semántica de las operaciones es opaca (específica de cada servicio).
- Aunque algunos _frameworks_ usen HTTP como transporte, suele usarse POST para todo, sea cuál sea la semántica de la operación.
- No existe el concepto de hipermedia.

En algunos _frameworks_ RPC (sobre todo en las implementaciones más antiguas) el acoplamientodel cliente con el servicio es más elevado:

- Casi cualquier cambio en la interfaz del servicio (incluso si se elimina o se añade un campo que nuestro cliente no usa), puede obligar a regenerar los `stubs` y recompilar.
- Con una librería local, está bajo control instalar o no una nueva versión, pero los cambios en una aplicación remota y autónoma no están bajo control.

Con las tecnologías usadas habitualmente con REST, puede conseguirse fácilmente que ciertos cambios no nos afecten.

- Con los _frameworks_ RPC modernos también se puede conseguir.

En cuanto a las tecnologías, sin tener en cuenta las diferencias arquitectónicas:

- Modelar un servicio con operaciones ad-hoc suele ser muy natural.
  - El uso de `stubs`/`skeletons` es más amigable al programador.
    - Con REST manejamos peticiones HTTP y _parsing_/generación de JSON/XML.
      - Aunque es posible utilizar tecnologías de más alto nivel que las vistas en esta asignatura que nos permitan un desarrollo más ágil.
- Pero con el tiempo, el uso de REST se ha hecho más popular en todos los lenguajes de programación.
