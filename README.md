# Devkron Data Model (DDM)
Dialecto y herramientas para diseñar y generar estructuras de bases de datos relacionales

## Contexto
Al definir modelos de bases de datos relacionales generalmente se emplean dos clases de herramientas:

a) CASE con una pesada interfaz de usuario y que requiere mucho trabajo para detallar a través de diagramas, adecuadamente los objetos y poder generar a partir de ellos el SQL DDL o bien,
b) directamente SQL DDL o una aplicación de DBA con interfaz de usuario que lo genere con la desventaja del fuerte acoplamiento al motor de bases de datos.

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

    entity "tuser" : "induxsoftTableModel"
    {
        @"userid" string required unique
        @"username" string(250) required
    }
    
    entity "agente" : "devkronTable"
    {
        @"codigo" string(15) required unique
        @"nombre" string(150) required unique
        @"iuser" int ref:"tuser" ["sys_pk"] unique
        index "indice1" unique
        {
            field "codigo" desc
            field "nombre"
        }
    }
    
    entity "sales_pipeline_stage" : "devkronTable"
    {
        @"ref_pipeline" ref:"sales_pipeline" required
        @"sequence" int required    //El número de secuencia de las etapas en el pipeline
        @"name" string(50) required
        @"probability" decimal(4,1)
        @"stuck_in_days" int
    }

    entity "sales_pipeline" : "devkronTable"
    {
        @"name" string(50)
        @"probability" bool
    }
```

### Definición de columnas
La sintaxis es: ```@"nombre_columna" tipo atributo1 atributo2 atributo3```

tipo es alguna de los siguientes:

* bool 
* int 
* datetime
* date
* time
* decimal(precisión, escala)
* string(tamaño)

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


## Generación de código




