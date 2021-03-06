[![License](https://img.shields.io/badge/license-MIT-blue.svg?style=flat-square)](LICENSE)
# Daniia
Se trata de un framework para realizar Mapeo de Objeto Racional (Object Relational Mapping ORM) para PHP basado en el patrón de diseño ORM ActiveRecord

## Instalación

### Composer

Para instalar con [Composer](https://getcomposer.org/), simplemente requiere la
última versión de este paquete.

```bash
composer require nozerrat/daniia dev-master
```

## Requerimientos
- PHP >= 5.3
- PDO

## Motor de Base de Datos soportado
Daniia ORM es compatible con:
* MySQL 5.1+
* Postgres 8+
* SQLite3
* SQLServer 2008+
* Oracle 

## Conección a la Base de Datos
Lo primero que necesitaremos es declarar las constantes para luego registrar los datos de conexión para poder usar el Framework Daniia, las constantes son USER, PASS, SCHEMA y [DSN](http://php.net/manual/es/pdo.drivers.php).

por ejemplo:
```php
// Conexión con la Base de Datos MySql
define("USER","root");
define("PASS","1234");
define("DSN","mysql:port=3306;host=localhost;dbname=test");
```
otro ejemplo:
```php
// Conexión con la Base de Datos PostgreSql
define("USER","postgres");
define("PASS","1234");
define("SCHEMA","public");
define("DSN","pgsql:port=5432;host=localhost;dbname=postgres");
```
## Instanciar Daniia ORM

```php
use Daniia\Daniia;
$daniia = new Daniia();
$daniia->table('personas')->get();// consultamos todos los datos
```
en el caso de hederar la clase Daniia hay que hederar la clase BaseDaniia
```php
use Daniia\Daniia;
use Daniia\BaseDaniia;
class Personas extends BaseDaniia {
   protected $table = "personas"; //Nombre de la tabla
   protected $primaryKey = "id"; //Clave primaria de la tabla
}
$personas = new Personas();
$personas->get();// consultamos todos los datos
```

También se puede cambiar la primaryKey desde una instancia Daniia:
```php

$daniia->primaryKey("id");

```

## Uso y Ejemplos
Para usar Daniia ORM es muy falcil, aquí aplicaremos algunos ejemplos del uso del Frameword:



### Insert
```php
// Insert simple
$daniia->table("personas")->insert(["ci"=>1,"nombre"=>"Carlos","apellido"=>"Garcia"]);

// Insert multiples
$daniia->table("personas")->insert([
   ["ci"=>1,"nombre"=>"Carlos","apellido"=>"Garcia"],
   ["ci"=>2,"nombre"=>"Carlos","apellido"=>"Garcia"],
   ["ci"=>3,"nombre"=>"Carlos","apellido"=>"Garcia"],
]);
```

### Update
```php
// Update simple
$daniia->table("personas")->primaryKey("id")->update(["id"=>1,"ci"=>"1111","nombre"=>"aa","apellido"=>"aa"]);

// o en caso de que la ID no este esablecida en los datos
$daniia->table("personas")->where("id",1)->update(["ci"=>"1111","nombre"=>"aa","apellido"=>"aa"], true/*Ignorar primaryKey*/);

// Update multiples
$daniia->table("personas")->primaryKey("id")->update([
   ["id"=>1,"ci"=>4,"nombre"=>"Petra","apellido"=>""],
   ["id"=>2,"ci"=>5,"nombre"=>"José","apellido"=>"Jill"],
   ["id"=>3,"ci"=>6,"nombre"=>"Jhon","apellido"=>"Peña"],
]);
```

### Delete
```php
$daniia->table("personas")->where("id",2)->delete();

$daniia->table("personas")->delete(3);
$daniia->table("personas")->delete([3]);

$daniia->primaryKey('id')->table("personas")->delete(6,7);
$daniia->primaryKey('id')->table("personas")->delete([6,7]);

$daniia->primaryKey('id')->table("personas")->find(8)->delete();
$daniia->primaryKey('id')->table("personas")->find([8])->delete();
```


### Select
```php
$daniia->table('personas')->select()->get();

$daniia->table('personas')->select('COUNT(*)')->first();

$daniia->table('personas')->select('ci','nombre')->first();
$daniia->table('personas')->select(['ci','nombre'])->first();
```

### Operadores validos
Los operadores que soporta la Framework Daniia son: =, <, >, <=, >=, <>, !=, like, not like, in, is, is not, ilike, between, not between. 

Por ejemplo:
```php
/**
 * OPERADORES VALIDOS
 **/
$daniia->table("personas")->where("id","4")->first();// por defecto es '='

$daniia->table("personas")->where("id",'=',"4")->first();

$daniia->table("personas")->where("id",'like',"4")->first();

$daniia->table("personas")->where("id",["4"])->first();// si es un array por default es 'IN'

$daniia->table("personas")->where("id",'in',["4"])->first();

$daniia->table("personas")->where("id",'is',"true",false)->first();//el parametro false indica no escapar valor
// o puede usar en este caso 
$daniia->table("personas")->where("id is true")->first();
```

### Table
El método table se encarga de asignar el nombre a la tabla que se desee consultar
```php
$daniia->table('personas')->first();
$daniia->table(['personas'])->first();

// realizamos un JOIN
$daniia->table('personas','oficina')->first();
$daniia->table(['personas','oficina'])->first();
```

### From
El método from es similar al método table, se usa para asignar los nombres de las tablas a consultar, pero si en su argumento especificamos un closure indicará un sub-quey contenido en la clausula from.

```php
$daniia->from('personas')->first();
$daniia->from(['personas'])->first();

$daniia->from('personas','oficina')->first();
$daniia->from(['personas','oficina'])->first();

// especificamos un sub-query
$daniia->from(function (Daniia $daniia) {
   $daniia->table("personas");
})->first();

// con alias para el sub-query contenido en el método from
$daniia->from(function (Daniia $daniia) {
   $daniia->table("personas");
}, "myAlias")->first();

// otro ejemplo de sub-query anidado contenido en el clausula from
$daniia->from(function (Daniia $daniia) {
   $daniia->from(function (Daniia $daniia) {
      $daniia->from(function (Daniia $daniia) {
         $daniia->table("personas");
      }, "C");
   }, "B");
}, "A")->first();

// otro ejemplo 
$daniia
->select("P.id","P.nombre","P.apellido")
->from(function (Daniia $daniia) {
   $daniia
   ->table("personas")
   ->select("personas.id","personas.nombre","personas.apellido");
}, 'P')->first();
```

### Join, LeftJoin, RightJoin
```php
// Join alternativa
$daniia->table('personas','oficina')->where('personas.id','oficina.id_personas',false)->first();

$daniia->table('personas')->join("oficina","personas.id","oficina.id_personas")->first();

$daniia->table('personas')->innerJoin("oficina","personas.id","=","oficina.id_personas")->first();

$daniia->table('personas')->leftJoin("oficina","personas.id","oficina.id_personas")->first();

$daniia->table('personas')->rightJoin("oficina","personas.id","oficina.id_personas")->first();

$daniia->table('personas')->rightJoin("oficina",['personas.id'=>[4,5,6,7]])->first();

$daniia->table("personas")->join('oficina',"personas.id",function(Daniia $query) {
   $query->select("personas.id")->from("personas")->where("personas.id","4")->limit(1);
})->first();


$daniia->table('personas')->join("oficina",function(Daniia $daniia) {
   $daniia
      ->on("personas.id",TRUE)
      ->on("personas.id",FALSE)
      ->on("personas.id",NULL)
      ->on("personas.id",[TRUE,FALSE,NULL])
      ->on("personas.id",[1,2,3,4])
      ->on("personas.id","oficina.id_personas")

      ->on( ["personas.id"=>"oficina.id_personas"] )
      ->on( ["personas.id <>"=>"oficina.id_personas"] )

      ->orOn("personas.id",'<>',"oficina.id_personas")
      ->andOn("personas.id","holas",true) //Escapamos el valor
})->first();

 $daniia->table('personas')->join("oficina alias_a",function(Daniia $daniia) {
   $daniia->on(function(Daniia $daniia) {
      $daniia->on("personas.id","alias_a.id_personas");
   });
})->join("oficina alias_b",function(Daniia $daniia) {
   $daniia->on("personas.id",'=',function(Daniia $daniia) {
      $daniia->table('oficina')->select('id_personas')->where('id_personas',4);
   });
})->first();

$daniia->table('personas')->join("oficina",function(Daniia $daniia) {
   $daniia->on("personas.id",function(Daniia $daniia) {
      $daniia->select("personas.id")->from("personas")->where("personas.id","4")->limit(1);
   });
})->first();
```

### Get, GetArray
```php
$daniia->table('personas')->get();

$daniia->table('personas')->getArray();

// en caso de que queramos realizar filtros en los datos consultados y queramos contituar consultando solo pasamos un Closure al método get o getArray y lo retornamos
$daniia->table('personas')->get(function( $data, Daniia $daniia ){
   // realizar algún filtro
   return $data;
});

$daniia->table('personas')->getArray(function( $data, Daniia $daniia ){
   // realizar algún filtro
   return $data;
});
```

### List
```php
$daniia->table('personas')->lists('nombre');

$daniia->table('personas')->lists('nombre','id');

$daniia->table('personas')->lists('nombre',function( $data, Daniia $daniia ){
   // realizar algún filtro
   return $data;
});

$daniia->table('personas')->lists('nombre','id',function( $data, Daniia $daniia ){
   // realizar algún filtro
   return $data;
});
```

###  Find
```php
$daniia->table('personas')->find(); // Consulta Todos

$daniia->table('personas')->where('id',1)->find();

$daniia->table('personas')->find(1);
$daniia->table('personas')->find([1]);

$daniia->table('personas')->find(1,2);
$daniia->table('personas')->find([1,2]);

$daniia->table('personas')->find(2);
echo $daniia->id;
echo $daniia->ci;
echo $daniia->nombre;
echo $daniia->apellido;

$daniia->table('personas')->find(1,2,function($data, Daniia $daniia) {
   // realizar algún filtro
   return $data;
});

$daniia->table('personas')->find([1,2],function($data, Daniia $daniia) {
   // realizar algún filtro
   return $data;
});
```

### First, FirstArray
```php
$daniia->table('personas')->first();
$daniia->table('personas')->firstArray();

$daniia->table('personas')->where('id',1)->first();
$daniia->table('personas')->where('id',1)->firstArray();

$daniia->table('personas')->primaryKey('id')->find([1,2])->first();
$daniia->table('personas')->primaryKey('id')->find([1,2])->firstArray();

$daniia->table('personas')->first(function($data, Daniia $daniia) {
   // realizar algún filtro
   return $data;
});

$daniia->table('personas')->firstArray(function($data, Daniia $daniia) {
   // realizar algún filtro
   return $data;
});

$daniia->table('personas')->primaryKey('id')->find([1,2],function($data, Daniia $daniia) {
   // realizar algún filtro
   return $data;
})->first(function($data, Daniia $daniia) {
   // realizar algún filtro
   return $data;
});
```

### Save
```php
$daniia->table('personas')->find(2);
$daniia->nombre = "yyyyyyyy";
$daniia->save();//UPDATE

$daniia->table('personas')->find(1,2)->first();
$daniia->nombre = "yyyyyyyy";
$daniia->save();//UPDATE

// Cuendo la función find esta sin argumento es obligatorio
// usar la función first para extraer los datos del Array
// consultado
$daniia->table('personas')->where('id',1)->find()->first();
$daniia->nombre = "yyyyyyyy";
$daniia->save();//UPDATE

$daniia->table('personas')->find('00')->first(); // registro no existe..
$daniia->ci       = "123456789";
$daniia->nombre   = "Carlos";
$daniia->apellido = "Garcia";
$daniia->save();//INSERT

$daniia->table('personas');
$daniia->ci       = "123456789";
$daniia->nombre   = "Carlos";
$daniia->apellido = "Garcia";
$daniia->save();//INSERT

$daniia->table('personas')->find()->first();
$daniia->ci       = "123456789";
$daniia->nombre   = "Carlos";
$daniia->apellido = "Garcia";
$daniia->save(function($data, Daniia $daniia) {
   // Si la actualización es exitosa devuelve los datos actualizado
   // sino, devuelve los datos insertado
   return $data;
});//UPDATE
```

### Where
```php
$daniia->table("personas")->where("id","1")->first();

$daniia->table("personas")->where("id",'=',"1")->first();

$daniia->table("personas")->where("id","4")->andWhere("id","4")->first();

$daniia->table("personas")->where("id",4)->orWhere("id",4)->first();

$daniia->table("personas")->where("id",[4,5,6,7])->first();

$daniia->table("personas")->where("id",'in',[4,5,6,7])->first();

$daniia->table("personas")->where(function (Daniia $daniia) {
   $daniia->where("id",4)->andWhere("apellido","LIKE","%garcia%");
})->first();

$daniia->table("personas")->where("id",FALSE)->orWhere("id",TRUE)->orWhere("id",NULL)->first();

$daniia->table("personas")->where("id",'=',FALSE)->orWhere("id",'=',TRUE)->orWhere("id",'=',NULL)->first();

$daniia->table("personas")->where("id",[TRUE,FALSE,NULL])->first();

$daniia->table("personas")->where(function (Daniia $daniia) {
   $daniia->where("id",4);
})->orWhere(function (Daniia $daniia){
   $daniia->where("id",4);
})->first();

$daniia->table("personas")->where(function (Daniia $daniia) {
   $daniia->where("id","1")->orWhere(function (Daniia $daniia) {
      $daniia->where("id","2")->andWhere(function (Daniia $daniia) {
         $daniia->where("id","3");
      });
   });
})->first();

$daniia->table("personas")->where("personas.id",function($query) {
   $query->table("personas")->select("id")->where("id",4)->limit(1);
})->first();

$daniia->table("personas")->where("personas.id","=",function($query) {
   $query->table("personas")->select("id")->where("id",4)->limit(1);
})->first();

$daniia->table("personas")->where("id",'in',function(Daniia $daniia){
   $daniia->table('personas')->select('id');
})->first();




$daniia->table("personas")->where(['nombre'=>'carlos'])->first();

$daniia->table("personas")->where(['nombre !='=>'carlos'])->first();

$daniia->table("personas")->where(['nombre'=>[4,5,6,7]])->first();

$daniia->table("personas")->where(['nombre in'=>[4,5,6,7]])->first();

$daniia->table("personas")->where(["personas.id"=>function(Daniia $daniia) {
   $daniia->table("personas")->select()->where("id",4)->limit(1);
}])->first();

$daniia->table("personas")->where(["personas.id !="=>function(Daniia $daniia) {
   $daniia->table("personas")->select()->where("id",4)->limit(1);
}])->first();
```

### Having 
```php
$daniia->table("personas")->having("id","1")->first();

$daniia->table("personas")->having("id",'=',"1")->first();

$daniia->table("personas")->having("id","4")->andHaving("id","4")->first();

$daniia->table("personas")->having("id",4)->orHaving("id",4)->first();

$daniia->table("personas")->having("id",[4,5,6,7])->first();

$daniia->table("personas")->having("id",'in',[4,5,6,7])->first();

$daniia->table("personas")->having(function (Daniia $daniia) {
   $daniia->having("id",4)->andHaving("apellido","LIKE","%garcia%");
})->first();

$daniia->table("personas")->groupBy( "id" )->having("id",false)->having("id",true)->having("id",null)->first();

$daniia->table("personas")->groupBy( "id" )->having("id",'=',false)->having("id",'=',true)->having("id",'=',null)->first();

$daniia->table("personas")->groupBy( "id" )->having("id",[true,false,null])->first();

$daniia->table("personas")->having(function (Daniia $daniia) {
   $daniia->having("id",4);
})->orHaving(function (Daniia $daniia){
   $daniia->having("id",4);
})->first();

$daniia->table("personas")->having(function (Daniia $daniia) {
   $daniia->having("id","1")->orHaving(function (Daniia $daniia) {
      $daniia->having("id","2")->andHaving(function (Daniia $daniia) {
         $daniia->having("id","3");
      });
   });
})->first();

$daniia->table("personas")->having("personas.id",function(Daniia $daniia) {
   $daniia->table("personas")->select("id")->having("id",4)->limit(1);
})->first();

$daniia->table("personas")->having("personas.id","=",function(Daniia $daniia) {
   $daniia->table("personas")->select("id")->having("id",4)->limit(1);
})->first();

$daniia->table("personas")->having("id",'in',function(Daniia $daniia){
   $daniia->table('personas')->select('id');
})->first();
```

### Union
```php
$daniia->table("personas")->where('id',1)->union(function (Daniia $daniia) {
   $daniia->table("personas")->where('id',1);
})->get();

$daniia->table("personas")->select('id')->union(function (Daniia $daniia) {
   $daniia->table("oficina")->select('id_personas AS id');
})->get();

$daniia->table("personas")->select('id')->where('id',1)->union(function (Daniia $daniia) {
   $daniia->table("oficina")->select('id_personas AS id')->where('id_personas',4);
})->get();

$daniia->table("personas")->where('id',1)->union(function (Daniia $daniia) {
   $daniia->table("personas")->where('id',1);
})->union(function (Daniia $daniia) {
   $daniia->table("personas")->where('id',1);
})->limit(1)->get();
```

### Order By
```php
$daniia->table("personas")->orderBy("ci")->get();
$daniia->table("personas")->orderBy(["ci"])->get();

$daniia->table("personas")->orderBy("apellido","nombre")->get();
$daniia->table("personas")->orderBy(["apellido","nombre"])->get();

// al final se indica el tipo de orden 
$daniia->table("personas")->orderBy("apellido","nombre",'asc')->get();
$daniia->table("personas")->orderBy(["apellido","nombre",'desc'])->get();
```

### Group By
```php
$daniia->table("personas")->groupBy("ci")->first();
$daniia->table("personas")->groupBy(["ci"])->get();

$daniia->table("personas")->groupBy("apellido","nombre")->get();
$daniia->table("personas")->groupBy(["apellido","nombre"])->get();
```

### Limit
```php
$daniia->table("personas")->limit("1")->get();
$daniia->table("personas")->limit(["1"])->get();

$daniia->table("personas")->limit("1","0")->get();
$daniia->table("personas")->limit(["1","0"])->get();
```

### Offset
```php
$daniia->table("personas")->limit("1")->offset("0")->get();
```

### Otros
```php
// Obtiene la última ID insertada en la base de datos
$daniia->lastId();

// Obtiene el última Query ejecutado
$daniia->lastQuery();

// Obtiene los últimos datos consultados
$daniia->getData();
```



## API
```php
Daniia {
   // Obtener datos resultante despues de la ejecución 
   public integer lastId( void );
   public mixed lastQuery( void );
   public mixed getData( void );
   public array error( void );

   // Escape de cadenas de caracteres
   public string quote( void );

   // Sentencia para transación
   public Daniia begin( void );
   public Daniia commit( void );
   public Daniia rollback( void );

   // ejecutar sentencias SQL
   public object query( string $sql [, Closure $closure] );
   public array queryArray( string $sql [, Closure $closure] );

   // Obtiener los campos de la tabla especificada
   public array columns( string $table = null );

   // ejecutar Query Builder
   public array get( [Closure $closure] );
   public array getArray( [Closure $closure] );
   public array all( [Closure $closure] );
   public array allArray( [Closure $closure] );
   public object first( [Closure $closure] );
   public array firstArray( [Closure $closure] );
   public array lists( string $column [, string $index = null | Closure $closure ] [, Closure $closure] );
   public bool save( [Closure $closure] );
   public Daniia find( [string $ids [, ...] | array $ids] [, Closure $closure] );

   public bool truncate( void );
   public int insertGetId( array $datas );
   public bool insert( array $datas [, $returning_id = false]  ); // $returning_id solo para PostgresSql
   public bool update( [array $datas] );
   public bool delete( [string $ids [, ...] | array $ids = []] );
   
   // Query Builder
   public Daniia primaryKey( string $primaryKey );
   public Daniia table( string $table [, ...] | array $table );
   public Daniia select( string $select [, ...] | array $select );
   public Daniia from( string $table [, ...] | array $table | Closure $table [, string $aliasFrom = ""] );
   public Daniia join( string $table, string $column | Closure $column [, string $operator = null | Closure $operator [, string $value = null | Closure $value | bool $value [, bool $scape_quote = false ]]] );
   public Daniia innerJoin( string $table, string $column | Closure $column [, string $operator = null | Closure $operator [, string $value = null | Closure $value | bool $value [, bool $scape_quote = false ]]] );
   public Daniia leftJoin( string $table, string $column | Closure $column [, string $operator = null | Closure $operator [, string $value = null | Closure $value | bool $value [, bool $scape_quote = false ]]] );
   public Daniia rightJoin( string $table, string $column | Closure $column [, string $operator = null | Closure $operator [, string $value = null | Closure $value | bool $value [, bool $scape_quote = false ]]] );
   public Daniia on( string $column | Closure $column [, string $operator = null | Closure $operator [, string $value = null | Closure $value | bool $value [, bool $scape_quote = false ]]] );
   public Daniia orOn( string $column | Closure $column [, string $operator = null | Closure $operator [, string $value = null | Closure $value | bool $value [, bool $scape_quote = false ]]] );
   public Daniia andOn( string $column | Closure $column [, string $operator = null | Closure $operator [, string $value = null | Closure $value | bool $value [, bool $scape_quote = false ]]] );
   public Daniia where( string $column | Closure $column [, string $operator = null | Closure $operator [, string $value = null | Closure $value | bool $value [, bool $scape_quote = false ]]] );
   public Daniia orWhere( string $column | Closure $column [, string $operator = null | Closure $operator [, string $value = null | Closure $value | bool $value [, bool $scape_quote = false ]]] );
   public Daniia andWhere( string $column | Closure $column [, string $operator = null | Closure $operator [, string $value = null | Closure $value | bool $value [, bool $scape_quote = false ]]] );
   public Daniia union( Closure $closure );
   public Daniia having( string $column | Closure $column [, string $operator = null | Closure $operator [, string $value = null | Closure $value | bool $value [, bool $scape_quote = false ]]] );
   public Daniia orHaving( string $column | Closure $column [, string $operator = null | Closure $operator [, string $value = null | Closure $value | bool $value [, bool $scape_quote = false ]]] );
   public Daniia andHaving( string $column | Closure $column [, string $operator = null | Closure $operator [, string $value = null | Closure $value | bool $value [, bool $scape_quote = false ]]] );
   public Daniia groupBy( string $fields | array $fields );
   public Daniia orderBy( string $fields | array $fields );
   public Daniia limit( int $limit [, int $offset = null ] | array $limit );
   public Daniia offset( int $offset);
}
```
