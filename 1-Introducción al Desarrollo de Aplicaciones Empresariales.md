# Introducción al desarrollo de aplicaciones empresariales

## Características de las aplicaciones empresariales

Por aplicación empresarial nos referimos a las que abordan necesidades críticas, no forzosamente en empresas. Deben satisfacer una serie de requisitos:

- **Escalabilidad**: Capacidad de soportar más usuarios o más carga al aumentar los recursos disponibles.
- **Alta disponibilidad**: Tolerancia a fallos mediante replicación de recursos.
- **Transaccionalidad**: Cumple las propiedades ACID.
  - _Atomicity_ (Atomicidad).
  - _Consistency_ (Consistencia).
  - _Isolation_ (Aislamiento).
  - _Durability_ (Durabilidad).
- **Seguridad**: No todos los usuarios pueden acceder a la misma funcionalidad.

Las aplicaciones modernas no funcionan de forma aislada, dependen de otras aplicaciones (accesibles a través de la red) para proporcionar sus servicios a los usuarios y a otras aplicaciones. Esto plantea una serie de problemas:

- Acceso a través de la red.
- Interacción con aplicaciones desarrolladas con otras tecnologías.
- Evolución independiente de las distintas aplicaciones.

---

## Diseño por Capas

El diseño por capas es una de las técnicas de diseño más usadas en Ciencias de la Computación.
Una capa inferior proporciona un servicio a otra capa superior:

- El servicio ofrecido se define mediante un “contrato de servicio”.
- Permite independizar el software de ambas capas:
  - A la capa superior no le importa cómo se implementa el servicio.
  - Cambios en la manera de implementar una capa no afectan a la otra.

Es clave para facilitar mantenimiento, escalabilidad y tolerancia a fallos.

Ventajas:

- Los desarrolladores de una capa no necesitan conocer las tecnologías usadas en otras capas.
- Cada capa puede ser desarrollada en paralelo con el resto de capas.
- Facilita el mantenimiento del software: cambios en la implementación de una no conllevan cambios en el resto de capas.
- Facilita la escalabilidad y la tolerancia a fallos: pueden aumentarse los recursos sólo de las capas que lo necesitan.

Riesgos:

- El software es más complejo.
- En ciertos casos una capa podría hacer su función de forma óptima si conociese como funciona otra capa.

---

## Arquitectura basada en Capas Típica

Separación clara entre las capas “modelo” e “interfaz”.

### Capa Modelo

Conjunto de clases que implementan la lógica de negocio de los casos de uso de la aplicación. Suele dividirse en dos subcapas:

- Capa Acceso a Datos: Bases de datos y otras aplicaciones.
- Capa Lógica de Negocio: Implementa la lógica de negocio, usando la capa de acceso a datos para leer o escribir los datos que necesita.

### Capa Interfaz

Se distinguen dos casos:

- Capa Interfaz Gráfica: interfaz gráfica que permite a los usuarios utilizar la funcionalidad de la capa modelo.
- Capa Servicios: interfaz programática orientada a que otras aplicaciones remotas utilicen la funcionalidad de la capa modelo.

Ventajas potenciales de la separación:

- Cada capa puede ser desarrollada por personas con un perfil distinto.
- Es posible reusar la capa modelo en distintas interfaces gráficas y por distintas aplicaciones.
- Cambios en la manera de implementar una capa no afectan a las otras capas.
- Aumenta la escalabilidad: Normalmente cada capa puede ejecutarse en máquinas diferentes.
- Facilita la escalabilidad y la tolerancia a fallos: Es posible replicar la capa modelo sin necesidad de replicar a las aplicaciones consumidoras.

---

## Distribución de las capas

### Capa Modelo e interfaz gráfica instaladas en la máquina cliente

La aplicación se instala completamente en la máquina, es una única aplicación que contiene todo. Este tipo de arquitectura tiene sentido en intranets en las que desde las máquinas cliente se necesita tener acceso a las BBDD.

Problemas:

- Cambios en la implementación:
  - Recompilación de toda la aplicación.
  - Reinstalación en todas las máquinas.
- Potenciales problemas de seguridad:
  - El código de la aplicación puede ser accesible desde los clientes.
  - Las BBDD pueden ser accesibles desde los clientes.

Solución:

- Mover la capa Modelo a un servidor intermedio: Cambios en la implementación del modelo sólo afectan al servidor.
- Clientes de escritorio: Sólo disponen de la interfaz gráfica. Acceden al servidor que implementa el modelo.

### Servidor intermedio para la capa Modelo

La capa Modelo se separa de la interfaz gráfica. Aparecen las capas Servicio y Acceso a Servicio para comunicar las anteriores.

En los clientes se instalan las interfaces gráficas y la capa Acceso a servicios, y en los servidores las capas Modelo y Servicios.

Problema: Un cambio en la interfaz gráfica sigue suponiendo tener que reinstalar en los clientes.

Solución: Usar una interfaz Web.

Ventaja de interfaces _standalone_ frente a interfaces Web: Pueden proporcionar cierta funcionalidad, incluso con conexión de red intermitente o lenta.

### Arquitectura en 3 capas

En el nombre de la arquitectura "capa" tiene un significado diferente: conjunto de máquinas que cumple una función diferenciada (cliente, servidores de aplicaciones y servidores de datos).

La interfaz web se instala junto la capa Modelo en un servidor de aplicaciones.

No se instala nada en los clientes. Acceden a la aplicación a través de internet/intranet mediante un navegador: Los cambios en la implementación solo afectan a los servidores de aplicaciones.

Los servidores de aplicaciones suelen tener soporte para escalabilidad y disponibilidad:

- Escalabilidad:
  - Se pueden replicar en varias máquinas (_cluster_ de máquinas)
  - Para atender más peticiones hay que añadir más máquinas al cluster
- Disponibilidad:
  - Se emplea un balanceador de carga: Recibe todas las peticiones HTTP.
  - Envía cada petición HTTP a uno de los servidores de aplicaciones.
  - Si una máquina se cae, todavía quedan otras instancias del servidor de aplicaciones en el _cluster_.

### Arquitectura en 4 capas

Se separa la interfaz Web de la capa Modelo.

Los clientes se conectan mediante el navegador a servidores con interfaces gráficas instaladas. Estos servidores se comunican con otros en los que se instala la capa Modelo haciendo uso de la capa Acceso a servicios y la capa Servicios.

Múltiples aplicaciones pueden invocar al modelo, sin necesidad de replicar el código del modelo en cada una.

Permite que la capa Interfaz Gráfica y la capa Modelo estén construidas con tecnologías diferentes.
