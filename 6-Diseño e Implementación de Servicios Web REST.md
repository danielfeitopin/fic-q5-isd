# Diseño e Implementación de Servicios Web REST

## Capa Servicios

Expone remotamente la capa modelo sin exponer directamente sus objetos. La razón es que se asume que el acceso remoto lo van a realizar aplicaciones externas y en la BD hay datos internos que no se desean exponer.

- Defineuna sencilla API para acceder al servicio.
- Oculta la tecnología usada para acceder al servicio.
- Permiten que el cliente pueda acceder a distintas implementaciones del servicio sin necesidad de recompilarlo.
  - Basta especificar en la configuración la implementación que se desea.
  - Los clientes son los mismos.

Lo habitual es que cada cliente use sus propios objetos ya que:

- Permite adaptar los objetos a lo que el cliente necesita.
- Es más natural tener objetos con sólo lo que se necesita.

**DTO** (_Data Transfer Object_): Objeto para transferir datos entre aplicaciones.

- Encapsula diferencias con los objetos internos de cada aplicación.
  - Porque no son relevantes externamente.
  - Porque se quieren objetos más gruesos para minimizar llamadas remotas.

Es común que a las aplicaciones externas se les exponga una API diferente que a las internas.

- Diferencias en los objetos expuestos.
- Operaciones no expuestas.
- En casos más complejos podrían ser necesarias validaciones o pasos adicionales en la lógica de negocio.

En un caso real, posiblemente habría al menos dos tipos de clientes:

- Uno para las labores de administración que se ejecutaría normalmente desde la intranet del servicio.
- Otro que se usaría normalmente desde Internet.

## Introducción a los Servicios Web REST

La Web es la aplicación distribuida más exitosa de la historia.

REST (_REpresentational State Transfer_): Estilo arquitectónico (paradigma) para construir aplicaciones distribuidas inspirado en las características de la Web.

- Propuesto por Roy Fieldingen en 2000.
- Es independiente de la tecnología.
  - Se suele implementar usando directamente HTTP y algún lenguaje de intercambio sin tecnologías adicionales.
- A menudo los servicios REST reales no siguen estrictamente todos los principios del estilo arquitectónico REST.
- A los servicios que sí siguen fielmente todos los principios REST, se les denomina a veces servicios web RESTful.

### HTTP

HTTP (_HyperText Transfer Protocol_):

- Estandarizado por el W3C y la IETF.
- Protocolo cliente/servidor utilizado en la Web.
- Esquema petición/respuesta.
- Inicialmente construido para transferir páginas HTML, pero puede transferir cualquier contenido textual.
  - Usando MIME: contenidos binarios.
- Utiliza TCP para comunicar cliente y servidor (puerto reservado: 80).
- Desde HTTP 1.1, una conexión HTTP puede utilizarse para varias peticiones.
- La última versión es HTTP 3.
- Concepto clave: URL como identificador global de recursos.

Una petición HTTP consta de:

- Una URL, que identifica al recurso sobre el que actúa la petición.
- Un método de acceso (GET, POST, PUT...), que especifica la acción a realizar sobre el recurso.
  - GET: Solicita una representación del recurso especificado.
    - Es seguro.
      - Semántica esencialmente de solo lectura.
      - El cliente no pide ni espera que se produzcan cambios en el servidor como resultado de realizar una petición sobre un recurso utilizando este método.
      - Un cliente puede utilizarlo sin miedo a causar efectos indeseados.
    - Pueden especificarse parámetros en la URL (consultas). Se especifican como pares campo=valor, separados por el carácter `&`.
  - PUT: Carga en el servidor una representación de un recurso.
    - La nueva representación del recurso va en el cuerpo de la petición.
    - No es seguro.
  - DELETE: Elimina el recurso especificado.
    - No es seguro.
  - POST: Envía datos al recurso indicado para que los procese.
    - Los datos van en el cuerpo de la petición.
    - Puede utilizarse para:
      - Crear un nuevo recurso.
      - Modificar un recurso existente.
    - No es seguro.
- Cabeceras: Metainformaciónde la petición que indican información adicional que puede ser necesaria para procesar la petición.
  - Tipo MIME de datos esperados.
  - _Encoding_ esperado.
  - Lenguaje esperado.
  - Peticiones condicionales (If-Modified-Since, Etags).
    - Solicita una representación sólo si ha cambiado desde un momento determinado.
    - Caches en el cliente.
  - Credenciales de autenticación/autorización.
  - Información para _proxies_.
  - Agente del usuario.
- Cuerpo del mensaje (opcional): Solo con algunos métodos de acceso.

Las peticiones GET, PUT y DELETE deben ser idempotentes: Múltiples peticiones iguales deben tener el mismo efecto en el servidor que una sola. Las peticiones POST no tienen porque serlo.

Una respuesta HTTP contiene:

- Código de _Status_:
  - 200 OK.
  - 201 Created.
  - 400 Bad Request.
  - 403 Forbidden.
  - 404 Not Found.
  - 410 Gone.
  - 500 InternalError.
- Cabeceras: Metainformaciónde la respuesta:
  - Tipo MIME de datos devueltos.
  - _Encoding_ de la respuesta.
  - Lenguaje de la respuesta.
  - Antigüedad de la respuesta.
  - Longitud de la respuesta.
  - Control de cache: si el recurso se puede cachear o no, tiempo de expiración (si se puede cachear), momento de última modificación...
  - Control de reintentos.
  - Identificación del servidor.
- Cuerpo del mensaje (opcional):
  - GET: representación del recurso invocado.
  - POST: vacío o con el resultado del procesamiento realizado por el servidor.
  - PUT: normalmente, el cuerpo viene vacío.
  - DELETE: normalmente, el cuerpo viene vacío.
  - En caso de error, puede incluir información con más detalle.

### Características Servicios REST

Los servicios web REST se estructuran de forma similar a un sitio de la WWW. Sin embargo, en lugar de páginas HTML, normalmente se accederá a información estructurada.

- Al igual que la WWW está compuesta de muchos sitios web autónomos, los servicios web REST se consideran autónomos entre sí.
- Al igual que en la WWW, un servicio REST puede referenciar a otro y habrá una serie de servicios generales que pueden funcionar sobre cualquier servicio.
- Cliente –Servidor.
- Servicios _stateless_.
- Interfaz uniforme.
- Sistema en capas: Intermediarios.
- Hipermedia.
- Representaciones autodescriptivas.

#### Servicios _stateless_ vs _stateful_

Cliente-Servidor: Los clientes invocan directamente URLs para acceder a recursos.

Servicios sin estado: El servidor no guarda información de estado para cada cliente. Cada petición de un cliente debe contener toda la información necesaria para que el servidor la responda.

Servicio con estado: El servidor guarda el directorio en el que está cada cliente. El resultado de la ejecución de un comando depende de ese estado.

Ventajas de los servicios sin estado:

- La replicación del servicio en múltiples máquinas es muy sencilla: basta usar un balanceador de carga que dirige cada petición a una instancia del servicio.
  - En un servicio con estado, es necesario o bien garantizar que todas la peticiones de la misma sesión van al mismo servidor o bien usar un servidor de sesiones.
- El servidor no necesita reservar recursos para cada sesión.
  - Disminuye los recursos que necesita el servicio.
  - Si el cliente falla o se cae en el medio de la sesión, el servidor no tiene que preocuparse de detectarlo ni de liberar recursos.
- En general: mejora la escalabilidad de los servicios.

Inconveniente: puede ser necesario enviar información adicional con cada petición.

#### Recursos y Representaciones

Una aplicación REST se compone de recursos. Suelen corresponderse con las entidades persistentes.

Hay dos tipos de recursos:

- Colección: identifican una serie de recursos del mismo tipo.
- Individual: identifican un elemento concreto.

Cada recurso es identificado mediante un identificador único y global. Los identificadores (URLs) son globales:

- Todo recurso tiene un nombre único a nivel inter-aplicación.
- No existen espacios de nombres restringidos a un servicio/aplicación: no puede haber dos recursos en servicios distintos con el mismo identificador.
- Cualquier servicio puede referenciar y acceder a un recurso de cualquier otro servicio.
  - Eso no quiere decir que todos los recursos sean accesibles para todos.

Al invocar la URL usando GET, el cliente obtiene una representación del recurso.

- La representación debe contener información útil sobre el recurso:
  - Recursos colección: normalmente lista de los elementos de la colección con información resumen y un enlace para obtener la información completa.
  - Recursos individuales: datos del elemento individual.
- La representación de un recurso puede variar en el tiempo.
  - El identificador está ligado al recurso, no a la representación.
  - La URL apuntará siempre a la representación actual del recurso.

#### Interfaz Uniforme

Las operaciones disponibles sobre los recursos son siempre las mismas.

- Las normas REST no dicen cuáles deben ser esas operaciones, sólo que deben ser siempre las mismas y funcionar igual en todos los servicios.

Los tipos de respuesta (tanto de éxito como de error) que pueden devolver estas operaciones deben estar también estandarizados.

- REST no impone un conjunto de códigos de respuesta concretos, sólo que deben ser los mismos para todos los servicios.

En la práctica, se utiliza HTTP:

- GET:
  - Acceso a representaciones.
  - Consultas sobre recursos colección.
  - Es segura y es idempotente.
- PUT:
  - Reemplaza la representación de un recurso.
  - Si la URL no existe, y el servicio lo permite, crea el recurso.
  - No suele permitirse sobre recursos colección.
  - No es segura y es idempotente.
- DELETE:
  - Borra el recurso.
  - No suele permitirse sobre recursos colección.
  - No es segura y es idempotente.
- POST:
  - Sobre recursos colección, suele usarse para crear un nuevo recurso en la colección.
    - El servidor devuelve la URL del nuevo elemento usando una cabecera estándar HTTP.
  - Sobre recursos individuales (o colección), puede usarse para modelar operaciones no seguras que no encajen con los otros métodos (overloaded POST).
    - En este caso la petición debe incluir en algún otro lugar (en la URI, cabeceras o cuerpo) información sobre la operación a invocar sobre el recurso.
  - Semejante a estilo RPC (no sigue principios de interfaz uniforme).
  - No es segura y puede no ser idempotente.

Códigos de error permanente/temporal:

- Es importante poder distinguir entre códigos de respuesta que indican que un error es permanente o temporal.
- La especificación HTTP no dice explícitamente si un código de error tiene asociada una semántica de error permanente o temporal.
- Dentro de una organización siempre es posible establecer convenciones sobre qué códigos representan errores permanentes o temporales.

Cabeceras HTTP estándar para enviar información adicional en la petición y en las respuestas.

#### Intermediarios

El uso de una interfaz uniforme permite que haya intermediarios entre el cliente y el servicio que proporcionan funcionalidad adicional.

Los intermediarios:

- Son transparentes para el cliente y el servicio.
- Pueden proporcionar funcionalidad adicional a cualquier servicio y a cualquier cliente sin necesidad de saber nada adicional sobre ellos.

#### REST en la práctica

- La mayoría de servicios reales no utilizan algunos de los principios de REST.
- Uso limitado de hipermedia.
- Uso limitado de representaciones autodescriptivas.

<!--## Diseño e Implementación de un Servicio REST

### Protocolo REST
-->
