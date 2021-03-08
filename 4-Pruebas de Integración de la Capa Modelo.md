# Pruebas de integración de la capa Modelo

Son pruebas de integración, y no de unidad, porque prueban el correcto funcionamiento de los servicios e indirectamente lo que estos utilizan internamente (los DAOs que acceden a la BD):

- Convención de nombrado: XxxServiceTest
- En general, para cada caso de uso se diseñan varios casos de prueba, cada uno implementado en un método `@Test`.
- A veces resulta conveniente probar más de un caso de prueba en un único método `@Test`.

---

## Pruebas en Maven

Con Maven el código de pruebas se ubica en el directorio `src/test` (que tiene una estructura de directorios similar a la del directorio `src/main`).

Las pruebas se pueden ejecutar automáticamente desde Maven (a través del plugin _surefire_) haciendo que se ejecute la fase test (e.g. `mvn test`):

- Maven genera un informe detallado en `target/surefire-reports`.
- Por defecto, se consideran clases de pruebas aquellas clases de `src/test/java` cuyo nombre empieza o termina por _Test_.

---

## Independencia entre casos de prueba

Para favorecer el mantenimiento del código es importante tender, en la medida de lo posible, a que cada caso de prueba:

1. Crear los datos que se precisen.
2. Invocar a la operación a probar.
3. Realizar las comprobaciones necesarias.
4. Eliminar los datos que crearon en el paso 1 y los que se pudieron generar en el paso 2.

De esta manera, cada caso de prueba es independiente del resto:

- La ejecución de un caso de prueba puede asumir que sólo existen en BD los datos que él mismo ha creado en 1.
- Modificar, eliminar o añadir un caso de prueba no tiene incidencia en el resto de casos.
- Esta independencia favorece que distintas personas a lo largo del tiempo puedan modificar las clases de prueba de manera ágil.

---

## Uso de DAOs

En general los casos de prueba necesitan crear los datos necesarios antes de invocar al caso de uso a probar, y finalmente eliminar los datos creados:

- Si el servicio ofrece un método apropiado, se usa.
- En otro caso, se usa directamente el DAO correspondiente.

Cuando en un caso de prueba se realizan las comprobaciones necesarias, puede ser necesario hacer alguna consulta a la BD:

- Mismo convenio que el caso anterior.

---

## Uso de una BD específica para ejecutar las pruebas

Es una buena práctica disponer de varias BDs con el mismo esquema, por ejemplo:

- Una BD para probar la aplicación como humano en la fase de desarrollo.
- Una BD para ejecutar las pruebas automatizadas.
- Una BD para producción.

De esta manera, el probar la aplicación como humano no altera la BD de pruebas: Los casos de prueba no se encontrarán con datos inesperados.

---

## Inicialización

A lo largo de los casos de prueba se utilizan variables estáticas. Estas variables se inicializan en un método anotado con `@BeforeAll` que se ejecuta una única vez, antes de que se ejecuten los métodos anotados con `@Test`.

---

## Configuración

Con Maven la configuración de las pruebas reside en `src/test/resources`. Cuando se ejecuta las pruebas sobre un módulo Maven, el _classpath_ está formado por (en este orden):

- `target/test-classes`.
- `target/classes`.
- Los JARs de las dependencias.

---

## _DataSourceLocator_

El test del servicio tiene que registrar un _DataSource_ explícitamente antes de poder invocar al servicio de la capa Modelo.

Sin embargo, cuando la capa Modelo se ejecuta dentro de un servidor de aplicaciones, es éste quien proporciona una implementación de un _DataSource_ (típicamente con pool de
conexiones). Esta será la situación cuando la capa Modelo se ejecute como parte de un servicio Web (o una aplicación Web).

Una aplicación (aplicación Web o servicio Web) instalada en un servidor de aplicaciones Java puede obtener referencias a
algunos objetos gestionados por el servidor de aplicaciones usando la API estándar JNDI (_Java Naming and Directory Interface_):

- Es una API (`javax.naming`) que forma parte de la API básica de Java, para localizar y registrar objetos asociándoles nombres jerárquicos.
- Los servidores de aplicaciones exponen los objetos _DataSource_ gestionados por el servidor de aplicaciones
mediante la API de JNDI.
- Los servidores de aplicaciones tienen la obligación de exponer los objetos _DataSource_ en el contexto `java:comp/env`.
- Cuando el administrador de un servidor de aplicaciones configura un DataSource, por convenio, se aconseja que lo registre bajo el subcontexto jdbc, de manera que el nombre completo queda como `java:comp/env/jdbc/<<nombre simple del DataSource>>`.
