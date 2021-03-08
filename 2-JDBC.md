# Java DataBase Connectivity

## JDBC

JDBC (Java DataBaseConnectivity) es una API estándar que permite lanzar consultas a una BD relacional.

El desarrollador siempre trabaja contra los paquetes **java.sql** y **javax.sql**. Forman parte de Java SE y contienen un buen número de interfaces y algunas clases concretas, que conforman la API de JDBC.

### _Drivers_

Para poder conectarse a la BD y lanzar consultas es preciso tener un _driver_ adecuado para ella:

- Un _driver_ suele ser un fichero .jar que contiene una implementación de todas las interfaces de la API de JDBC.
- El _driver_ lo proporciona el fabricante de la BD o un tercero.
- El código nunca depende del _driver_.

#### Independencia de la BD

Idealmente, si una aplicación cambia de BD, no se necesitaría cambiar el código sino otro _driver_. Sin embargo, las BBDD relacionales usan distintos dialectos de SQL a pesar de que en teoría es un estándar:

- Tipos de datos: varían mucho según la BD.
- Generación de identificadores: secuencias, autonumerados, etc.

Cuando se desea que el código sea independiente de la BD, es posible utilizar patrones para hacer frente a este problema.

---

## Accesos básicos

### Obtención de conexiones

Interfaz `java.sql.Connection`:

- Representa una conexión a la BD.

Clase `java.sql.DriverManager`:

- Dispone del método estático `.getConnection()` que permite obtener una conexión a la BD a partir de:
  - Una URL:
    - Formato especificado en la documentación del _driver_.
    - Indica la máquina en la que corre la BD y el nombre del esquema.
  - El identificador y contraseña de un usuario que tenga permisos de acceso.

En una aplicación real, para que no sea necesario modificar el código cuando se cambia la configuración de acceso a la base de datos, la URL, el usuario y la contraseña se deben leer de un fichero de configuración.

### Formateo de los datos

Interfaz `java.sql.PreparedStatement`:

- Contiene la consulta SQL parametrizada que se va a lanzar.
- Parámetros:
  - Los caracteres "?" que aparecen en la consulta.
  - Se numeran de 1 en adelante.
- Dispone de métodos `.setXxx()` para dar valor a los parámetros.
- Método `.executeUpdate()`:
  - Permite lanzar consultas SQL de actualización (e.g. INSERT, UPDATE, DELETE, etc.).
  - Devuelve el número de filas afectadas por la actualización.
- Método `.executeQuery()`:
  - Permite lanzar consultas SQL de lectura (e.g. SELECT).

Cuando se lanza la consulta, el driver sustituye y formatea automáticamente los parámetros en el formato requerido por la BD:

- Cómodo: el driver formatea los datos, no el desarrollador.
- Portable: el driver usará el formato adecuado para la base de datos subyacente.

### Procesamiento de filas resultado

Interfaz `java.sql.ResultSet`:

- Representa todas las filas que han concordado con la consulta de búsqueda.
- Mantiene un cursor inicialmente posicionado antes de la primera fila. Si no quedan filas por leer, `.next()` devuelve `false`. En otro caso, avanza el cursor en una posición y devuelve `true`.
- Dispone de métodos `.getXxx()` para acceder a los valores de las columnas de la fila en la que está posicionado el cursor.

Dado que el número de filas que pueden concordar con una consulta puede ser muy grande, los _drivers_ suelen implementar `ResultSet` de manera que se van trayendo bloques de filas de la BD a medida que se va necesitando.

### Exceptiones de SQL

Los métodos de la API de JDBC reportan cualquier error lanzando `SQLException("checked")` de `java.sql.SQLException` o una de sus hijas.

---

## Liberación de recursos

### _try-with-resources_

La construcción `try-with-resources` es una extensión de `try-catch` introducida en Java 7.

Permite declarar uno o más recursos después de la palabra clave `try`:

- Son objetos que implementan la interfaz `java.lang.AutoCloseable`, que sólo dispone del método `.close()`.
- Deben crearse cuando se declaran.
- Se declaran entre paréntesis separados por "`;`".
- El compilador de Java garantiza que se invocará al método `.close()` de los recursos declarados cuando se termine el bloque `try`, tanto si se producen excepciones como si no. Cuando se ejecuten los posible bloques `catch` o `finally`, los recursos declarados ya estarán cerrados.

`java.sql.Connection` extiende `java.lang.AutoCloseable`.

Si no se utiliza `try-with-resources`, tendría que cerrarse la conexión explícitamente.

### Cierres del _driver_

El driver JDBC garantiza que:

- Cuando se cierra una conexión, se cierran todos sus `PreparedStatement` asociados.
- Cuando se cierra un `PreparedStatement`, se cierran todos sus `ResultSet` asociados.

### _finalize_

`.finalize()` es un método definido en `Object`. El recolector de basura lo invoca antes de eliminar un objeto de memoria.

La implementación de la interfaz `Connection` debe redefinir `.finalize()` para que invoque a `.close()` en caso de que el desarrollador no lo haya hecho.

---

## Tipos Java y SQL

Un dato de tipo Java se puede almacenar en una columna cuyo tipo SQL sea consistente con el tipo Java.

Las BBDD suelen emplear nombres distintos para los tipos que proporcionan.

No afecta al código Java (excepto que cree tablas, lo que en general, no debe hacerse).

### Correspondencia entre tipos Java y SQL estándar

Tipo Java|Tipo SQL|
:--------|:-------
boolean|BIT
byte|TINYINT
short|SMALLINT
int|INTEGER
long|BIGINT
float|REAL
double|DOUBLE
java.math.BigDecimal|NUMERIC
String|VARCHAR o LONGVARCHAR
byte[]|VARBINARY o LONGVARBINARY
java.sql.Date|DATE
java.sql.Time|TIME
java.sql.Timestamp|TIMESTAMP

---

## DataSources

Interfaz `javax.sql.DataSource`:

- Entre otros, dispone del método `.getConnection()`.
- Cuando se utiliza esta interfaz, el desarrollador no tiene que especificar la URL, el usuario y la contraseña para pedir la conexión.

Los servidores de aplicaciones Java y algunos _frameworks_ ofrecen implementaciones de la interfaz `DataSource`:

- A nivel de implementación utilizan `DriverManager.getConnection()` para obtener las conexiones, aunque la estrategia puede ser compleja.
- Utilizan ficheros de configuración para especificar, como mínimo, la URL, el usuario y la contraseña.

---

## _Pool_ de conexiones

En servidores de aplicaciones que reciben muchas peticiones HTTP por minuto es posible pedir una conexión a la BD con `DriverManager.getConnection()` o el método `.getConnection()` de un objeto `DataSource`:

- En una implementación básica de `DataSource`, el método `.getConnection()` también invoca `DriverManager.getConnection()`.
- `DriverManager.getConnection()` pide una conexión directamente a la BD:
  - Es una operación lenta: se convierte en cuello de botella.
  - Si en ese momento la BD ya no admite más conexiones porque se supera el máximo permitido, los métodos `.getConnection()` devuelven una excepción.

Los servidores de aplicaciones Java proporcionan implementaciones de `DataSource` que utilizan la estrategia **_pool_ de conexiones**:

- El objeto `DataSource` gestiona un conjunto de conexiones que previamente ha solicitado a la BD.
- El desarrollador sólo trabaja contra la interfaz `DataSource`.
- La estrategia es transparente al desarrollador.

&nbsp;
![UML_ConnectionPool](./images/ConnectionPool.PNG)
&nbsp;

`ConnectionPool`:

- Cuando se crea, pide "n" conexiones a la BD usando `DriverManager.getConnection()` y las almacena en una lista.
- `.getConnection()`: Si quedan conexiones libres en la lista, elige una, la marca como usada, y devuelve un objeto `ConnectionProxy` que la contiene. En otro caso, deja durmiendo al _thread_ llamador.
- `.releaseConnection()`: Devuelve la conexión a la lista, la marca como libre, y notifica a los posibles _threads_ que esperan por una conexión.

`ConnectionProxy`:

- Proxy de la conexión real.
- `.close()`: Usa `.releaseConnection()` para devolver la conexión real al _pool_.
- `.finalize()`: Si no se ha llamado a `ConnectionProxy.close()`, lo llama.
- Resto de operaciones: Delegan en la conexión real.

Cuando el desarrollador invoca `.getConnection()` sobre el objeto `DataSource`:

- Si hay una conexión libre se le devuelve rápidamente de la lista (no se accede a BD).
- Si no hay ninguna conexión libre el _thread_ llamador se queda dormido hasta que haya una.

Las conexiones reales no se cierran (se devuelven al _pool_).

### Caídas de la BD

Si la BD se cae, las conexiones del _pool_ se invalidan aunque se vuelva a rearrancarla BD porque los _sockets_ subyacentes ya no son válidos.

Para hacer frente a este problema, la implementación de `.getConnection()` puede comprobar si la conexión que devuelve es correcta:

- Opción 1: haciendo uso de una API específica del fabricante del _driver_.
- Opción 2: lanzando una consulta poco costosa a la BD (si no se produce una `SQLException`, la conexión es correcta).

### Configuración del pool

Además de la configuración básica de un `DataSource`, se puede especificar el número de conexiones a la BD que se solicitan inicialmente, la consulta de comprobación de conexión viva (si se requiere), etc.

---

## Transacciones

Permiten ejecutar bloques de código con las propiedades **ACID** (_Atomicity Consistency Isolation Durability_).

Por defecto, cuando se crea una conexión está en modo _auto-commit_: Cada consulta lanzada se ejecuta en su propia transacción.

Para ejecutar varias consultas en una misma transacción es preciso:

- Deshabilitar el modo auto-commitde la conexión.
- Lanzar las consultas.
- Terminar con `connection.commit()` si todo va bien, o `connection.rollback()` en otro caso.

### Nivel de aislamiento de transacciones

`java.sql.Connection` proporciona el método `.setTransactionIsolation()`, que permite especificar el nivel de aislamiento deseado:

- TRANSACTION_NONE: transacciones no soportadas.
- TRANSACTION_READ_UNCOMMITTED: pueden ocurrir _dirtyreads_, _non-repeatablereads_ y _phantomreads_.
- TRANSACTION_READ_COMMITTED: pueden ocurrir _non-repeatablereads_ y _phantomreads_.
- TRANSACTION_REPEATABLE_READ: pueden ocurrir _phantomreads_.
- TRANSACTION_SERIALIZABLE: elimina todos los problemas de concurrencia.

A mayor nivel de aislamiento la BD realiza más bloqueos, lo que conlleva menos concurrencia.
