# Diseño e implementación de la capa Modelo

## Casos de uso

Un **caso de uso** es cada una de las funcionalidades que ofrece la capa Modelo. Muchas veces un caso de uso a nivel de modelo corresponde a una funcionalidad concreta que el usuario de la aplicación puede desear ejecutar.

---

## Principios de diseño

La capa Modelo debe ofrecer una API que:

- Permita invocar cada caso de uso de manera sencilla: Facilidad de uso para el desarrollador de la capa superior.
- Oculte detalles de implementación: Es posible modificar la implementación sin que afecte a la capa superior.

---

## Método de desarrollo

1. Modelar las entidades (clases persistentes).
2. Para cada entidad, crear una abstracción
que permita gestionar su persistencia.
3. Definir una API sencilla para invocar los
casos de uso.
4. Implementar los casos de uso.
5. Implementar las pruebas de integración de
los casos de uso.

---

## Modelado de entidades

Las entidades son la información que queremos guardar de forma persistente. De la descripción de los casos de uso se deduce qué hay que guardar en la BD.

---

## Gestión de la persistencia

Se crean las tablas en la BD de datos par guardar las entidades.

Para implementar los casos de uso se pueden necesitar para cada entidad:

- Operaciones básicas: insertar/leer/actualizar/eliminar una instancia.
  - **CRUD** (_Create Read Update Delete_).
- Operaciones avanzadas: Búsquedas que devuelven una colección de instancias, actualizaciones múltiples...

### Objetos de acceso a datos

Varios casos de uso pueden necesitar las mismas operaciones de persistencia. Para facilitar la implementación de los casos de uso, se necesita una abstracción que permita gestionar la persistencia de cada entidad.

Sería ideal que la abstracción ocultase la BD concreta, el tipo de BD y la tecnología usada para acceder a la BD: Si se cambia de BD o tecnología de acceso, no sería necesario cambiar la implementación de los casos de uso.

Esta abstracción corresponde al patrón de diseño **DAO** (_Data Access Object_):

- Típicamente, un DAO dispone de una interfaz y (al menos) una clase de implementación.
- Convención de nombrado: SqlXxxDao
- Conceptualmente, los DAOs corresponden a la capa Acceso a Datos. No deben realizar comprobaciones de lógica.

En los casos en los que no se pueda separar la implementación de la BD concreta, el tipo de BD y la tecnología usada para acceder a la BD, se puede crear una **clase abstracta** que implemente las funciones del DAO que sean independientes de todo eso y otra clase que herede de esta que implemente las funcionalidades dependientes.

---

## Definición del API de la capa Modelo

1. Agrupar los casos de uso de manera lógica.
2. Crear una interfaz para cada grupo:
   - Normalmente contiene un método por cada caso de uso.
   - Los argumentos de cada método corresponden a los datos de entrada del caso de uso y los valores de retorno a los datos de salida.
   - Cada una de estas interfaces corresponde a la aplicación del patrón de diseño **Fachada**.
   - Convención de nombrado: XxxService

---

## Implementación de los casos de uso

Una clase implementa los casos de uso utilizando los DAOs:

- Convención de nombrado: XxxServiceImpl.
- Conceptualmente corresponde a la capa Lógica de Negocio.

Hay que tener en cuenta:

- Gestión de transacciones.
- Obtención de referencias a _DataSource_.
- Obtención de referencias a DAOs.
- Obtención de referencias al propio servicio desde la capa cliente.

### Gestión de transacciones

Se sigue un enfoque sistemático:

- Casos de uso que sólo ejecutan una operación de lectura contra la BD:
  - Usar el modo _auto-commit_ y el nivel de aislamiento por defecto.
- Resto de casos de uso:
  - Validar datos de entrada si es necesario.
  - Empezar transacción:
    - Deshabilitar el modo _auto-commit_ de la conexión.
    - Establecer el nivel de aislamiento.
  - Comprobar que es posible ejecutarlo en función de otros datos en BD si es necesario.
  - Si no es posible:
    - `commit` para finalizar la transacción.
    - Lanzar una excepción _checked_.
  - Implementar la lógica de negocio usando los DAOs
  - Si se produce una excepción correspondiente a un error grave:
    - `rollback`.
    - Lanzar excepción.
  - En otro caso:
    - `commit`.

### Obtención de referencias a _DataSource_

El código tiene que ser capaz de obtener una referencia al _DataSource_ de la aplicación en dos situaciones distintas:

- Cuando la capa Modelo se ejecuta dentro de una aplicación Web o un servicio Web:
  - La aplicación Web o servicio Web se ejecuta dentro de un entorno (el servidor de aplicaciones) que proporciona _DataSources_ (típicamente con pool de conexiones).
- Cuando la capa Modelo es invocada por las pruebas de integración de la capa Modelo:
  - Las pruebas se ejecutan en un entorno, generalmente, la línea de comandos o un IDE, que NO proporciona _DataSources_. Es responsabilidad del desarrollador de las pruebas proporcionar el _DataSource_ a la capa Modelo.

### Obtención de referencias a DAOs

Para que sea posible obtener una referencia a un DAO sin conocer su clase de implementación, se utiliza el patrón **Factoría**:

- La clase factoría lee un parámetro de configuración y crea una única instancia de esa clase (atributo estático dao), que es la que siempre devuelve su _getter_.
- La factoría trata al DAO como un **_Singleton_**, dado que es un objeto que no tiene estado, y en consecuencia puede ser usado trivialmente por múltiples _threads_.
- Si se desea usar otra estrategia de generación de identificadores numéricos o se cambia a una BD en la que el DAO especificado no sea válido, basta proporcionar una nueva implementación del DAO y modificar el fichero de configuración.
- Convención de nombrado: SqlXxxDaoFactory.

### Obtención de referencias al propio servicio

Para que la capa cliente del modelo no esté acoplada a la clase de implementación de la interfaz, también se proporciona una factoría que funciona de manera similar a las factorías de los DAOs:

- Convención de nombrado: XxxServiceFactory.
- Al igual que en el caso de los DAOs, la factoría trata al servicio como un _Singleton_, dado que no mantiene estado.

---

## Implementar las pruebas de integración de los casos de uso

Son pruebas de integración, y no de unidad, porque prueban el correcto funcionamiento de los servicios e indirectamente lo que estos utilizan internamente (los DAOs que acceden a la BD).
