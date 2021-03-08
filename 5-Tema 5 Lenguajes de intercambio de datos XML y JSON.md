# Tema 5 Lenguajes de intercambio de datos XML y JSON

Las aplicaciones remotas:

- Pueden residir en otras máquinas o incluso acceder a la capa modelo a través de Internet.
- Pueden estar escritas en cualquier lenguaje de programación.
- Proporcionarán su propia interfaz gráfica (si la tienen) y pueden utilizar otras bases de datos o aplicaciones adicionales además de la propia.

Son necesarios lenguajes o formatos de datos robustos, que permitan expresar y transmitir información compleja entre sistemas heterogéneos (XML, JSON, YAML...).

Campos de aplicación:

- Intercambio de datos entre aplicaciones heterogéneas.
- Generación de vistas (HTML, WML, PDF...) a partir de documentos de datos.
- Configuración de aplicaciones.
- Bases de datos.

## XML (_eXtensible Markup Language_)

- Lenguaje de _tags_ (similar en sintaxis a HTML).
- Estandarizado por el W3C.
- Extensible:
  - XML no impone un conjunto de _tags_, sino sólo unas pocas normas sobre cómo usarlos.
    - Los _tags_ se abren (`<tag>`) y se cierran (`</tag>`) y en medio pueden tener otros _tags_ anidados.
    - Todos los documentos tienen un _tag_ raíz.
    - Los _tags_ pueden tener atributos.
- Permite expresar información estructurada y fácilmente parseable.

### Formato de los documentos XML

Documento XML: Secuencia de caracteres (fichero, secuencia de caracteres que se envía por un socket...) que tiene texto en formato XML.

Aplicación XML: Conjunto particular de _tags_ que permiten representar determinada información.

Un documento XML está **bien formado** (_well-formed_) si cumple con un conjunto de reglas:

- Se distingue entre mayúsculas y minúsculas.
- Comentarios:
  - Empiezan con "`<!--`" y terminan con "`-->`".
  - Pueden englobar varias líneas.
  - Dentro de un comentario no puede aparecer la secuencia "`--`".
    - Comentarios como "`<!---- Comentario ---->` no son válidos.

Estas reglas permiten construir _parsers_ eficientes.

Declaración XML:

```xml
<?xml version="1.0" encoding="UTF-8"?>
```

- No es obligatoria (aunque sí aconsejable).
  - Si aparece tiene que hacerlo justo al principio del documento.
- `version`: Indica la versión de la especificación XML (y no la versión del documento) a la que es conforme el documento.
- `encoding`: Indica la codificación de los caracteres del documento.
  - Cuando se utiliza XML para intercambio de información entre un cliente y un servidor, lo más normal es usar codificación UTF-8

### Elementos y atributos

Todo documento debe tener un elemento raíz.

Entre el _tag_ de inicio (`<tag>`) de un elemento y el de fin (`</tag>`) puede haber otros elementos o texto.

- No se puede mezclar el orden de los _tags_ anidados.
- El primer elemento que se abre debe ser el último que se cierra.

Un elemento puede tener atributos:

- El valor del atributo tiene que ir entrecomillado con comillas dobles (`"`) o simples (`'`).
- Para un elemento dado, un atributo sólo puede tener un valor.

Elementos vacíos:

- No tienen elementos anidados ni texto.
- Puede tener atributos.
- Notaciones:

```xml
<tag attr-1="val1" attr-2="val2"></tag>

<!-- Por comodidad, se puede usar la notación: -->
<tag attr-1="val1" attr-2="val2"/>
```

Convención de uso (elementos vs atributos):

- Usar elementos para datos multivaluados.
- Usar contenido de elementos para datos de gran cantidad de texto.
- Usar atributos o contenido de elementos en cualquier otro caso.

### Referencias a entidades predefinidas

Para poder incluir ciertos caracteres en el valor de un atributo o en el texto de un elemento, es preciso emplear referencias a entidades:

| Referencia | Carácter |
| :--------: | :------: |
|  `&quot;`  |   `"`    |
|  `&apos;`  |   `'`    |
|  `&amp;`   |   `&`    |
|   `&lt;`   |   `<`    |
|   `&gt;`   |   `>`    |

### Espacio de nombres

Un documento XML puede hacer uso de espacios de nombres.

- Permiten evitar conflictos de nombres cuando en un documento XML se usan elementos y atributos de distintas aplicaciones XML.
- Cada espacio de nombres está asociado a una URI, que debe ser única.
  - Una URI es un identificador de un recurso, y típicamente es una URL.
  - Se aconseja usar URLs(porque es una forma fácil de elegir nombres únicos).
  - No tienen por qué tener una existencia real (y de hecho, no suelen tenerla).

```xml
<!-- xmlns (xml name space) es la única palabra clave -->
<tag xmlns="uri1" xmlns:alias="uri2" ...>
```

`xmlns="uri1"`: Espacio de nombres por defecto. Todos los elementos contenidos dentro del elemento "tag" que no empiecen por el "alias`:`", y el propio elemento "tag", pertenecen al espacio de nombres "uri1".

`xmlns:alias="uri2"`: Todos los elementos y atributos contenidos dentro del elemento "tag" que empiezan por el "alias`:`" pertenecen al espacio de nombres "uri2". "alias" actúa sólo como un alias de la URI del espacio de nombres, es decir, el identificador del espacio de nombres no es "alias", sino la URI.

### Validación de documentos XML

Existen varios tipos de esquemas para expresar los elementos y atributos válidos de una aplicación XML y sus restricciones.

Existen dos estándar del W3C para especificar esquemas XML:

- DTD (_Document Type Definition_): Sencillo pero menos potente.
- Esquema XML (_XML Schema_):
  - Complejo.
  - Semánticamente muy rico.
  - Un esquema XML es a su vez un documento XML.
    - .xsd es la extensión que típicamente se utiliza para los esquemas XML (_XML Schema Definition_).
  - Es buena práctica que cada esquema defina su propio espacio de nombres.

Un documento XML que cumple las restricciones de un esquema (DTD, esquema XML...) se dice que es **válido**.

Una aplicación que parsee un documento XML:

- Siempre comprueba que esté bien formado.
- Puede decidir comprobar si es válido. En caso de decidir validarlo:
  - El documento puede indicar (normalmente a través de URLs) cómo obtener los esquemas que utiliza.
  - La aplicación también puede tener copias locales de los esquemas.

#### Esquemas XML

```xml
<xsd:schema xmlns:xsd="http://www.w3.org/2001/XMLSchema"
    targetNamespace="uri1"
    xmlns="uri1"
    elementFormDefault="qualified">
```

Los elementos y tipos que se utilizan para definir un esquema XML están definidos en el esquema asociado al espacio de nombres <http://www.w3.org/2001/XMLSchema>.

`targetNamespace`: Especifica la URI del espacio de nombres de los elementos definidos en este esquema.

`xmlns`: Especifica el espacio de nombres por defecto para los elementos y tipos que no se prefijen (típicamente es el espacio de nombres especificado en `targetNamespace`).

`elementFormDefault`: En los documentos XML que utilicen este esquema, todos los elementos deberán escribirse cualificados (bien porque se ha utilizado el espacio de nombres por defecto o porque se les ha prefijado explícitamente). En general, es buena práctica especificar siempre `qualified` (por defecto, el valor de `elementFormDefault` es `unqualified`).

##### Tipos en XML

Tipo simple (`simpleType`): Tipos (de elementos o atributos) que tienen sólo valores y no otros atributos o elementos.

- Existen varios tipos predefinidos: `string`, `int`, `long`, `short`, `float`, `double`, `boolean`, `byte`, `dateTime`...
- Restricciones:
  - Permiten definir un tipo simple a partir de otro restringiendo los valores de este último.
  - Representan uno de los mecanismos disponibles para definir tipos derivados de otros tipos simples.

Tipo complejo (`complexType`): Tipos (de elementos) que tienen atributos y/o elementos.

- Se define con compositores:
  - `sequence`: define una secuencia ordenada de elementos.
  - `all`: los elementos pueden aparecer en cualquier orden. Tiene restricciones específicas.
  - `choice`: sólo puede aparecer uno de los elementos.
- Es posible definir tipos complejos de atributos y elementos.
  - La lista de atributos se especifica al final, después del compositor usado para definir los elementos.
- Es posible definir tipos complejos por derivación (se usan restricciones).

##### Elementos

Cada elemento se define dando:

- Su nombre.
- Su tipo.
- El número mínimo y máximo de ocurrencias posible (opcionales).
  - Por defecto: `minOccurs="1"` y `maxOccurs="1"`.
  - Valor `unbounded` para ilimitados.

## JSON (_JavaScript Object Notation_)

JSON está basado en un subconjunto del lenguaje de programación JavaScript (estándar ECMA-262).

Estandarizado en 2013 en el RFC 7158 y ECMA-404:

- El estándar ECMA describe solamente la sintaxis, mientras que el RFC cubre adicionalmente algunos aspectos de seguridad e interoperabilidad.
- El último RFC (8259), que define el estándar fue publicado en 2017 y sigue siendo consistente con ECMA-404.

Permite expresar información estructurada y fácilmente parseable. Se basa en dos estructuras soportadas por cualquier lenguaje de programación: objetos (registros, _structs_) y _arrays_.

### Formato de un documento JSON

Documento JSON: Secuencia de caracteres que tiene texto en formato JSON.

- Codificación de caracteres UTF-8.

Un documento JSON está **bien formado** (_well-formed_) si cumple con un conjunto de reglas:

- Contiene un valor que puede ser de cualquiera de los 6 tipos:
  - Un objeto:
    - Se especifica entre llaves: `{}`.
    - Conjunto sin orden de pares campo-valor.
    - Las claves son cadenas.
    - Los valores pueden ser de cualquier tipo JSON válido.
    - Las claves se separan de los valores por dos puntos.
    - Los pares campo-valor se separan por comas.
  - Un _array_:
    - Se especifican entre corchetes: `[]`.
    - Colección ordenada de valores.
    - Pueden ser de cualquier tipo válido.
    - Se separan por comas.
  - Una cadena:
    - Entre comillas dobles: `""`.
    - Cualquier carácter Unicode excepto `"` o `\` o caracteres de control.
      - El carácter \se utiliza para:
        - Escapar caracteres.
        - Especificar un carácter a partir de su código Unicode: \u + 4 dígitos (hexadecimal).
    - No pueden contener caracteres de retorno de carro, nueva línea, tabulador...
  - Un número.
  - Un booleano (`true`/`false`).
  - `null`
- Se distingue entre mayúsculas y minúsculas.

Estas reglas permiten construir _parsers_ eficientes. Se pueden insertar espacios en blanco entre dos tokens cualesquiera.

Consideraciones respecto a XML:

- No hay comentarios.
- No existe el concepto de atributo.
- No existe el concepto de espacio de nombres.
- Los objetos y _arrays_ no tienen nombre si no son el valor de un campo de un objeto.
- En XML un elemento con un único subelemento no se puede distinguir si se trata de un arrayo no a no ser que tengamos un esquema contra el que validarlo.

Se pueden utilizar convenciones para que haya una equivalencia entre un documento JSON y XML, incluso para representar espacios de nombres:

- Utilizar un campo con un nombre especial para los comentarios.
- Considerar que las propiedades que comienzan por @ son atributos del objeto padre.
- Crear un objeto con una única propiedad para los _arrays_ y objetos no asignados a un campo.

### Validación de documentos JSON

Los esquemas JSON (JSON _Schema_) permiten expresar restricciones sobre el contenido de un documento JSON.

- <https://json-schema.org>
- No está estandarizado.

Un documento JSON que cumple las restricciones de un esquema JSON se dice que es **válido**.

Una aplicación que parseeun documento JSON:

- Siempre comprueba que esté bien formado.
- Puede decidir comprobar si es válido. Validar tiene ventajas e inconvenientes.

#### Esquemas JSON

Los esquemas JSON son a su vez documentos JSON.

- A diferencia de XML, no existe una forma estandarizada para que un documento indique cómo obtener el esquema que utiliza.
- Contienen un objeto con campos que imponen un conjunto de restricciones:
  - El esquema más simple (`{ }`) no impone ninguna restricción y por tanto cualquier documento JSON es válido según ese esquema.
  - Existen una serie de campos que son solamente descriptivos y no expresan restricciones:
    - `$schema`: Indica que el documento JSON es un esquema, y además puede indicar conforme a qué versión. No es obligatorio pero es una buena práctica utilizarlo.
    - `$id`: Para asignar un identificador único al esquema. Tampoco es obligatorio pero es una buena práctica utilizarlo.
    - `$comment`: Para añadir comentarios.
  - Campo `type`: Para restringir los documentos válidos a uno o varios de los 6 tipos existentes en JSON(aunque para numéricos hay dos tipos diferentes). Su valor puede ser:
    - Una cadena especificando el nombre de un tipo: Numérico (`integer` o `number`), `string`, `object`, `array`, `boolean` o `null`.
    - Un _array_ de cadenas especificando varios nombres de tipos, en cuyo caso pasaría la validación si fuese de cualquiera de esos tipos. Útil para valores de cualquier tipo que pueden ser nulos.

##### Tipos en JSON

Tipo `array`:

- Por defecto los elementos pueden ser de cualquier tipo.
- Para restringir el tipo de los elementos se usa el campo `items`: Se especifica como un objeto que a su vez es un JSON _schema_.
- Para restringir el tamaño del _array_ se puede usar `minItems` y `maxItems`.

Tipo `object`:

- Por defecto, cualquier objeto es válido.
- Para restringir las propiedades que puede tener el objeto, se usa el campo `properties`: El valor es un objeto en el que los nombres de los campos especifican el nombre de la propiedad y el valor es un objeto (a su vez un JSON _Schema_) que especifica las restricciones del valor de la propiedad.
- Por defecto, solo se valida que si el objeto tiene esas propiedades, entonces tienen el tipo adecuado.
  - `additionalPropertiesse` utiliza para especificar si el objeto puede tener o no propiedades que no estén en el esquema.
  - `required` se utiliza para especificar qué propiedades son obligatorias.

Tipos básicos:

- `string`:
  - Restricciones:
    - Longitud máxima (`maxLength`) y mínima (`minLength`).
    - Patrón (`pattern`).
    - Formatos o patrones predefinidos (`format`): `date`, `time`, `date-time`, `email`...
- Tipos numéricos:
  - `integer`: enteros.
  - `number`: enteros o decimales.
  - Restricciones:
    - Múltiplos (`multipleOf`).
    - Rango (`maximum`, `minimum`, `exclusiveMaximum`, `exclusiveMinimum`).

- `boolean`.
- `null`.

Enumerados:

- Se especifican con el campo `enum`.
- El valor es un _array_ con al menos un elemento y sus valores deben ser únicos.
- Pueden ser de diferentes tipos.

## _Parsing_ de documentos XML y JSON

Un documento XML o JSON se apoya en dos ideas:

- Tiene que estar **bien formado**, y en consecuencia, estar construido en base a las normas.
- El contenido del documento puede estar descrito formalmente en algún tipo de esquema de manera que se pueda comprobar que el documento es **válido**

Consecuencia: Es posible construir _parsers_ genéricos que comprueban que el documento está bien formado, y si se desea, que es válido.

- Existen _parsers_ para los lenguajes más usuales.

Tipos de _parsers_:

- _Parsers_ tipo DOM (_Document Object Model_): Construyen un árbol en memoria equivalente al documento.
  - Pueden hacerlo dado que el documento sigue un modelo jerárquico.
  - Ventajas:
    - Sencillos de utilizar: para acceder a la información del documento, basta recorrer el árbol.
    - Suelen permitir modificar/crear un árbol y generar un documento a partir de él.
  - Desventajas:
    - Consumo de memoria alto en aplicaciones servidoras que reciben muchas peticiones que involucran parsear/generar documentos grandes.
      - Normalmente las aplicaciones servidoras sirven cada petición en un _thread_.
- _Parsers_ tipo _streaming_: No construyen un árbol en memoria, sino que procesan secuencialmente el documento en bloques.
  - Ventajas:
    - Mínimo consumo de memoria.
    - Especialmente útiles en las situaciones en las que los _parsers_ de tipo DOM son prohibitivos.
  - Desventajas:
    - No todos tienen soporte para generar el documento.
    - Más difíciles de usar que los _parsers_ de tipo DOM.
