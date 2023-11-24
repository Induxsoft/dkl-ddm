# Devkron Data Model (DDM)
Dialecto y herramientas para diseñar y generar estructuras de bases de datos relacionales

## Contexto
Al definir modelos de bases de datos relacionales generalmente se emplean dos clases de herramientas:

* CASE con una pesada interfaz de usuario y que requiere mucho trabajo para detallar a través de diagramas, adecuadamente los objetos y poder generar a partir de ellos el SQL DDL o bien,
* directamente SQL DDL o una aplicación de DBA con interfaz de usuario que lo genere con la desventaja del fuerte acoplamiento al motor de bases de datos.

## Necesidad
Una manera simple de definir estructuras relacionales a través de archivos de texto (código fuente) que no requiera herramientas con interfaz de usuario, 
que sea independiente del motor de bases de datos, que pueda generar el SQL DDL y que no tenga una sintaxis tan ofuscada como el SQL, ni laboriosa como 
el XML o el JSON.

## Solución
Aprovechando las características de definición de sintaxis de Devkron Language, se definió un dialecto declarativo que expresa en nuestra opinión, con claridad y simplicidad los 
objetos y sus relaciones, además de adicionar características como la herencia de abstracciones.

## Dialecto
DDM se basa en los siguientes conceptos:

### Modelo
Es un conjunto de abstracciones y entidades

### Abstracción
Es un modelo de tabla que no se implementará físicamente pero que incluye definiciones de columnas o índices que podrán heredar las entidades descendientes.

### Entidad
Es la definición de una tabla física e incluye columnas e índices

```
#include "ddm.dkh" // Los patrones de sintaxis de DDM

model "nombre del modelo"
{
  abstract "nombre_de_la_abstracción"
  {
    @"nombre_de_una_columna" tipo_de_datos atributos...
    ...
  }
  
  entity "nombre_de_la_entidad" : abstracción_de_la_abstracción_de_la_que_desciende // (opcional)
  {
    ... definiciones de columnas ...
  }
}
```

Una entidad puede descender de 0 o hasta 5 abstracciones.

#### Ejemplo
```
#include "ddm.dkh"
model "CRM Sales"
{
    abstract "induxsoftTableModel"
    {
        @"sys_pk" int key autoincrement
        @"sys_guid" string(32) unique required
        @"sys_dtcreated" datetime required
        @"sys_timestamp" datetime required
        @"sys_recver" int
        @"sys_deleted" bool
        @"sys_lock" int
        @"sys_info" string(32)
        @"sys_user" string(32)
    }

    abstract "induxsoftEnumModel"
    {
        @"id" int key 
        @"const" string(150)
    }

    entity "tabla_solita" //Una tabla de que no desciende de ninguna abstracción
    {
      @"campo1" string(10) key 
      @"campo2" int
    }
    
    entity "tuser" : "induxsoftTableModel"
    {
        @"userid" string required unique
        @"username" string(250) required
    }

    entity "domicilio" : "induxsoftTableModel"
    {
        @"codPos" string 
    }
    entity "agente" : "induxsoftTableModel"
    {
        @"codigo" string(15) required unique
        @"nombre" string(150) required unique
        @"uf_iuser" int ref:"tuser" ["sys_pk"] unique

        @"codnomina" string(50)
        @"email" string(50)
        @"notas" string(2048)
        @"pcomision" decimal(18,8)
        @"telefono" string(150)
        @"domicilio1" int ref:"domicilio" ["sys_pk"]
        @"uf_sys_baja" bool
        
        index "indice1" unique
        {
            field "uf_iuser"
            field "nombre" desc
        }
    }

    entity "cmdedocivil" : "induxsoftEnumModel" { }
    entity "cmdsexo" : "induxsoftEnumModel" { }

    entity "ctpersona" : "induxsoftTableModel"
    {
        @"descripcion" string(50)
    }
    entity "ciudad" : "induxsoftTableModel"
    {
        @"codigo" string(8)
    }
    entity "pais" : "induxsoftTableModel"
    {
        @"codigo" string(8)
    }
    entity "persona" : "induxsoftTableModel"
    {
        @"nombre" string(150) required
        // @"telefono" string(15)
        @"email" string(100)

        @"apellido1" string(50) required
        @"apellido2" string(50)
        @"autoid" string(150)
        @"codigo" string(32)
        @"codpostal" string(10)
        @"colonia" string(100)
        @"domicilio" string(255)
        @"email2" string(100)
        @"fnacimiento" date 
        @"msn" string(100)
        @"notas" memo
        @"skype" string(100)
        @"sys_guid_contacto" string(32)
        @"tel_casa" string(80)
        @"tel_movil" string(80)
        @"tel_trabajo" string(80)
        @"tratamiento" string(10)
        @"tel_movil" string(80)
        @"edocivil" int ref:"cmdedocivil" ["id"] 
        @"sexo" int ref:"cmdsexo" ["id"]
        @"icategoria" int ref:"ctpersona" ["sys_pk"] required
        @"iciudad" int ref:"ciudad" ["sys_pk"]
        @"iejecutivo" int ref:"agente" ["sys_pk"]
        @"nacionalidad" int ref:"pais" ["sys_pk"]
    }

    entity "ctorganizacion" : "induxsoftTableModel"
    {
        @"descripcion" string(50)
    }
    entity "organizacion" : "induxsoftTableModel"
    {
        @"nombre" string(150) unique required

        @"codigo" string(32)
        @"codpostal" string(10)
        @"domicilio" string(255)
        @"email" string(100)
        @"fax" string(80)
        @"nomcomercial" string(150)
        @"notas" memo
        @"sys_guid_cliente" string(32)
        @"sys_guid_otro" string(32)
        @"sys_guid_proveedor" string(32)
        @"tel1" string(80)
        @"tel2" string(80)
        @"tel3" string(80)
        @"website" string(150)

        @"icategoria" int ref:"ctorganizacion" ["sys_pk"] required
        @"iciudad" int ref:"ciudad" ["sys_pk"]
        @"iejecutivo" int ref:"agente" ["sys_pk"]
        @"nacionalidad" int ref:"pais" ["sys_pk"]
    }

    entity "sales_pipeline_stage" : "induxsoftTableModel"
    {
        @"ref_pipeline" ref:"sales_pipeline" required
        @"sequence" int required    //El número de secuencia de las etapas en el pipeline
        @"name" string(50) required
        @"probability" decimal(4,1)
        @"stuck_in_days" int
    }

    entity "sales_pipeline" : "induxsoftTableModel"
    {
        @"name" string(50)
        @"probability" bool
    }
}
```

### Definición de columnas
Las columnas se definen dentro del cuerpo de las entidades o abstracciones.

La sintaxis es: ```@"nombre_columna" tipo atributo1 atributo2 atributo3```

tipo es alguna de los siguientes:

* bool 
* int 
* datetime
* date
* time
* decimal(precisión, escala)
* string(tamaño)
* memo
* mmemo
* lmemo

Los atributos (son opcionales y) pueden ser los siguientes:

* unique - Valores de clave única
* required - No se admiten nulos 
* autoincrement - El valor se incrementa automáticamente por parte del motor
* key - Es el campo de clave primaria

Las claves foráneas se indican con la sintaxis:

Sintaxis simplificada de definición de referencia que dejará al generador la inferencia del tipo y campo clave de la tabla referida.
```@"nombre_columna" ref: "entidad_referenciada"```

Sintaxis completa en donde explícitamente se indica el tipo de la columna y el nombre del campo clave referido
```@"nombre_columna" tipo ref: "entidad_referenciada" [campo_clave_referido]```

### Definición de índices
Los índices se definen dentro del cuerpo de las entidades o abstracciones

```
  index "indice1" unique //modificador opcional unique para indicar que es un índice de clave única
  {
    field "campo1"
    field "campo2" desc // Modificador de orden del campo del índice
  }
```

## Instalación de las herramientas de DDM
Simplemente coloque en la carpeta de binarios de Devkron los archivos de la carpeta src de este repositorio.

Los archivos incluidos son:

* ddm.dkh - Las definiciones de sintaxis
* dbgen.dkl - El generador de código de primer paso
* gen_mysql.dkl - Generador de segundo paso para MySQL
* gen_sqlserver.dkl - Generador de segundo paso para SQLServer
* install-model.dkl - Instalador de modelos (DDM)

## Uso de las herramientas

### Generación de estrctura desde DDM

```
dkl dbgen "src=archivo_fuente" "fmt=programa_generador" "out=archivo_salida"

```
* `src`: es la ruta y el nombre de un archivo en dialecto DDM
* `fmt` (opcional): es el nombre del programa generador de segundo paso, si este parámetro se omite, se producirá como salida un objeto JSON que representa la estructura como resultado de la generación de primer paso.
* `out`: es el archivo en donde se escribirán los resultados de la generación, si se omite el resultado se imprimirá como salida de la consola.

Ejemplo de línea de comando para generar código MySQL
```
dkl dbgen "src=modelo.dkl" "fmt=gen_mysql.dkl" "out=script.sql"
```
Ejemplo de línea de comando para generar código SQLServer
```
dkl dbgen "src=modelo.dkl" "fmt=gen_sqlserver.dkl" "out=script.sql"
```

### Comparación de dos modelos y generación de actualización
```
dkl dbgen "a=archivo_fuente_A" "b=archivo_fuente_B" "c=lista,de,tablas,a,filtrar" "fmt=programa_generador" "out=archivo_salida"
```
* `a`: es la ruta y el nombre de un archivo json que representa la estructura como resultado de la generación de primer paso.
* `b`: es la ruta y el nombre de un archivo json que representa la estructura como resultado de la generación de primer paso.
* `c` (opcional): es una lista de nombres de tablas separadas por coma(,) las cuales seran comparadas.
* `fmt` (opcional): es el nombre del programa generador de segundo paso, si este parámetro se omite, se producirá como salida un objeto JSON que representa la estructura como resultado de la generación de primer paso.
* `out`: es el archivo en donde se escribirán los resultados de la generación, si se omite el resultado se imprimirá como salida de la consola.

Ejemplo de línea de comando para comparar 2 esquemas y generar nuevo código MySQL
```
dkl dbgen "a=esquema_a.json" "b=esquema_b.json" "fmt=gen_mysql.dkl" "out=script.sql"
```
Ejemplo de línea de comando para comparar 2 esquemas y generar nuevo código SQLServer
```
dkl dbgen "a=esquema_a.json" "b=esquema_b.json" "fmt=gen_sqlserver.dkl" "out=script.sql"
```

### Instalación de modelos
```
dkl install-model "db=mi_conexión" "model=mi_modelo"
```
* `db`: Cadena con el nombre cualificado de una conexión existente.
* `model`: Es la ruta y nombre del archivo (modelo) `.ddm` o `.json` que representa la estructura de una base de datos.

Ejemplo de línea de comando para instalar un modelo ddm
```
dkl install-model "db=mibasededatos@migrupodeapps" "model=mimodelonuevo.ddm"
```
Para la instalación del nuevo modelo la herramienta trabaja con 2 tablas sys_model_log y sys_model_ddl_log que serán creadas en la base de datos de la cadena de conexión si no existen.

Cuando el proceso de instalación ha finalizado podra ver en:

* sys_model_log: información del modelo instalado.
* sys_model_ddl_log: información de las acciones ejecutadas (nuevas tablas, nuevas columnas, modificación de columnas, etc.)

Campos incompatibles en true
* required
* primarykey

#### Funciones de install-model

```
dkl install-model "db=mi_conexion" "action=funcion"
```
* `db`: Cadena con el nombre cualificado de una conexión existente.
* `action`: Una de las siguientes funciones.
    - infer-model: deducir el modelo en base a una aplicación (db).
    - show-model: obtener el modelo de una aplicación (db).
    - show-last-error: obtener el último error de instalación ocurrido.

Ejemplo de linea de comando para obtener el modelo de una aplicación.
```
dkl install-modell "db=mibasededatos@migrupodeapps" "action=show-model"
```

---
Acerca del proceso de generación

1. EL modelo es validado sintácticamente de acuerdo a los patrones establecidos en ddm.dkh
2. El modelo es transformado a JSON (primer paso de generación)
3. El modelo descrito en JSON produce SQL o cualquier otra salida (segundo paso de generación)

# Utilidades
## dba
Utilidad de línea de comandos para administrar conexiones y crear bases de datos.

```
./dkl dba [opción] [parámetros]
```
Opciones:

* ```show all``` Muestra todas las conexiones en el repositorio
* ```show``` Muestra la información de la conexión indicada en el parámetro ```qn```
* ```delete``` Elimina la conexión indicada en el parámetro ```qn```
* ```create database``` Crea una base de datos y su registro de conexión con la información de los parámetros ```qn``` y ```settings```
* ```create connection``` Crea un registro de conexión con la información de los parámetros ```qn``` y ```settings```

Parámetros
* ```qn``` Indica un nombre cualificado de conexión (nombreconexion@grupodeaplicacion)
* ```settings``` Indica un conjunto de valores para establecer una conexión a la base de datos vea [Funciones de administración de conexiones](https://docs.induxsoft.net/es/devkron/Bibliotecas-de-funciones/dbr/dbr.md)

Ejemplos
```
# Mostrar todas las conexiones
./dkl dba show all

# Mostrar información de una conexión
./dkl dba show qn=miconexion@grupoapps

# Eliminar una conexión
./dkl dba delete qn=miconexion@grupoapps

# Crear una conexión a una base de datos existente
./dkl dba create connection qn=miconexion@grupoapps "settings=Engine:MY_SQL,Host:nombreoipdelhost,Port:3306,UserName:root, Password:12345,SslMode:0,DefaultSchema:nombrebasededatos"

# Crear una base de datos y su registro de conexión
./dkl dba create database qn=miconexion@grupoapps "settings=Engine:MY_SQL,Host:nombreoipdelhost,Port:3306,UserName:root, Password:12345,SslMode:0,DefaultSchema:nombrebasededatos"
```
