---
typora-copy-images-to: ../../static/assets/
typora-root-url: ../../../
layout: post
slug: object-relational-mapping
conToc: true
title : Object Relational Mapping
date: 2022-09-03T19:50:07+01:00
---



## 3.1 Editar entidades

¿Qué pasa si, tras crear una entidad, queremos modificar su estructura? Podemos editar la clase de la entidad manualmente para añadir, modificar o borrar campos, pero también podemos volver a ejecutar el comando `make:entity`, indicar el mismo nombre de clase que queremos modificar, y especificar los nuevos campos que queramos añadir (en el caso de que lo que queramos sea añadir campos).
Después de definir los cambios en la(s) entidad(es) deseada(s), deberemos generar una nueva migración con los comandos vistos en el subapartado anterior.

## 3.2 Operaciones contra la base de datos

### 3.2.1 Insertar objetos

Si queremos añadir objetos nuevos a nuestra base de datos, basta con que creemos un objeto de la entidad correspondiente en el método oportuno, y llamemos al método `persist` y `flush` del `entity manager` de Doctrine.
Por ejemplo, para probar, vamos a crear un controlador en nuestra clase `ContactoController` asociado a una ruta `/contacto/insertar`, que de momento será de pruebas hasta que hagamos un formulario de inserción. Dentro de este método, creamos los objetos `Contacto` a partir del array que hemos creado anteriormente, obtenemos el `entity manager` de Doctrine y persistimos el objeto.

Por ejemplo, vamos a crear una ruta `/contacto/nuevo/manuel/99999/v@v.com`

![image-20260701125217553](/symfony-contactos-teoria/static/assets/image-20260701125217553.png)

### 3.2.2 Consultas más avanzadas

Con los métodos de consulta anteriores podemos realizar consultas que se limitan a comprobar si uno o varios campos de un objeto son iguales a unos criterios de búsqueda determinados. Pero, ¿cómo podríamos, por ejemplo, buscar los contactos cuyo nombre empiece por un cierto texto, o los libros de más de 100 páginas? Para este tipo de consultas, necesitamos ampliar el repositorio de nuestra entidad.

Por ejemplo, para nuestra entidad `Contacto`, imaginemos que queremos buscar los contactos cuyo nombre empieza un cierto texto. Para conseguir esto, necesitamos editar el repositorio de la entidad, que está en `src/Repository/ContactoRepository.php`. Este archivo contiene comentados un par de métodos de prueba que podríamos definir para ampliar las capacidades de la entidad.

En nuestro caso, vamos a añadir un método que se encargará de obtener los contactos cuyo nombre empiece por un texto determinado que le pasemos como parámetro:

Es **importante** recalcar que la llamada a `persist` por sí sola no actualiza la base de datos, sino que indica que se quiere persistir el objeto indicado. Es la llamada a `flush` la que hace efectiva esa persistencia.


```php
public function startsWith($value): array
{
    return $this->createQueryBuilder('c')
        ->andWhere('c.nombre LIKE :val')
        ->setParameter('val', $value . '%')
        ->orderBy('c.id', 'ASC')
        ->getQuery()
        ->getResult();
    // La consula en sql sería SELECT nombre FROM contactos WHERE nombre LIKE ('$value%')
}
```

Creamos el controlador:

donde lo único que varía es el nombre de la clase, `Contacto` en este caso.
```php
#[Route('/contacto/empieza/{letra}', name: 'empieza-por')]
public function empieza(ManagerRegistry $doctrine, Request $request, string $letra)
{
    $repositorio = $doctrine->getRepository(Contacto::class);
    $contactos = $repositorio->startsWith($letra);
    return $this->render('lista_contactos.html.twig', [
        'contactos' => $contactos,
        'letra' => $letra,
    ]);
}

Y la plantilla


```php
{% extends 'base.html.twig' %}
{% block body %}
	<h1>Contactos de la letra
		{{ letra }}</h1>
	{% for contacto in contactos %}
		{{ include ('partials/_contacto.html.twig', {'contacto': contacto})}}
	{% endfor %}
{% endblock %}
```

  ```php
  <?php
  $contacto = $repositorio->find(1);
  ```

* El método `findOneBy` localiza un objeto que cumpla los criterios de búsqueda pasados como parámetro. Así buscaríamos el contacto cuyo teléfono sea “900110011”:

  ```php
  <?php
  $contacto = $repositorio->findOneBy(["telefono" => "54565859"]);
  ```

  En el caso de querer definir más criterios de búsqueda, se pasarían uno tras otro en el array, separados por comas.

* El método `findBy` localiza todos los objetos que cumplan los criterios de búsqueda pasados como parámetro. Esta instrucción es como la anterior, pero devuelve un array de contactos con todos los resultados coincidentes:

  ```php
  <?php
  $contactos = $repositorio->findBy(["telefono" => "54565859"]);
  ```

* El método `findAll` (sin parámetros), obtiene todos los objetos de la colección.

  ```php
  <?php
  $contactos = $repositorio->findAll();
  ```
Todos estos métodos se obtienen a partir de un repositorio de la clase, que viene a ser algo así como un asistente que nos ayuda a obtener objetos que pertenezcan a esa clase.

Veamos un ejemplo con nuestra clase `ContactoController`: vamos a modificar nuestro método ficha para que, en lugar de buscar en la base de datos de prueba que hemos venido empleando en sesiones anteriores, busque por id en la base de datos real. Para ello, obtenemos el repositorio de nuestra clase `Contacto` y buscamos (`find`) el contacto con el id que hemos recibido como parámetro:

![image-20220109165736024](/symfony-contactos-teoria/assets/image-20220109165736024.png)

Empleamos el **query builder** de Doctrine para construir la consulta con esa sintaxis específica. En primer lugar, definimos un elemento (alias) que hemos llamado `c` (de `Contacto`) que usaremos para referenciar las propiedades de los contactos, por ejemplo, en la cláusula `where`. Lo que viene a hacer este código es buscar aquellos contactos `c` cuyo nombre sea como el parámetro `text`, y a continuación especifica que dicho parámetro `text` es igual al parámetro que recibimos en el método, encerrado entre símbolos `'%'`, para indicar que da igual lo que haya delante o detrás del texto.

Ahora, ya podríamos utilizar este método desde donde lo necesitemos. Por ejemplo, podemos modificar el método buscar de `ContactoController` para que busque contactos por nombre empleando este nuevo método:

Si, por ejemplo, quisiéramos buscar por una propiedad numérica (por ejemplo, personas cuya edad sea mayor que una dada), usaríamos una sintaxis como esta (también muy similar a SQL):
```php
<?php
$qb = $this->createQueryBuilder('p')
->andWhere('p.edad > :edad')
->setParameter('edad', $edad)
->getQuery();
```

Alternativamente, también podemos emplear un lenguaje llamado `DQL` (Doctrine Query Language) para realizar la consulta anterior:

```php
<?php
public function startsWith($text): array
{

    $entityManager = $this->getEntityManager();
    $query = $entityManager->createQuery(
        'SELECT c FROM App\Entity\Contacto c WHERE c.nombre LIKE :text'
    )->setParameter('text', $text . '%');

    return $query->execute();        
}
```

Y, como tercera vía, también podemos emplear SQL estándar, pero en este caso lo que obtendríamos ya no sería un array de objetos, sino un array de registros, como si empleáramos la librería mysqli de PHP para acceder a la base de datos.

Aquí tenéis enlaces para consultar información adicional tanto de [Query Builder](https://www.doctrine-project.org/projects/doctrine-orm/en/latest/reference/query-builder.html) como del lenguaje [DQL](https://www.doctrine-project.org/projects/doctrine-orm/en/latest/reference/dql-doctrine-query-language.html).

También se puede usar métodos con nombre de campos para buscar por campos de una entidad:

* `$contactos = $repositorio->findByEmail($email);` Encuentra  varios con ese correo
* `$contactos = $repositorio->findOneByEmail($email);` Encuentra sólo uno con ese correo
* Lo mismo se aplica para los campos `nombre` y `telefono`

### 3.2.3 Actualizar objetos

Para actualizar un objeto en una base de datos, debemos seguir tres pasos:

* Obtener el objeto de la base de datos (típicamente haciendo un `find` por su clave primaria)
* Modificar los datos necesarios con los respectivos `setters` del objeto
* Hacer un `flush` para actualizar los cambios en la base de datos.

Si, por ejemplo, quisiéramos actualizar el `nombre` de un contacto haríamos esto:


![image-20260702083620019](/symfony-contactos-teoria/static/assets/image-20260702083620019.png)

```php
// El valor por defecto del parámetro `codigo` es 1
#[Route('/contacto/update/{codigo?1}', name: 'update')]
public function update(ManagerRegistry $doctrine, $codigo): Response
{
    $entityManager = $doctrine->getManager();
    
    // Se coge el repositorio de la entidad Contacto o de la que se quiera
    $repositorio = $doctrine->getRepository(Contacto::class);
    
    // Se busca el contacto que tenga el id = $codigo
    // El método `find` siempre busca por la clave de la tabla, que suele ser `id`
    $contacto = $repositorio->find($codigo);
    
    // Cambiamos un dato, por ejemplo el nombre
    $contacto->setNombre("Nombre cambiado");
    
    // Guardamos de forma temporal
    $entityManager->persist($contacto);
    
    try{
        // y no nos olvidemos de guardar en la base de datos
        $entityManager->flush();
        
        // Mostramos la plantilla pasándole el contacto como parámetro
        return $this->render("ficha_contacto.html.twig", ["contacto" => $contacto]);
    }catch (\Exception $e){
        return new Response("Se ha producido un error: " . $e->getMessage());
    }
}
```

### 3.2.4 Borrar objetos

El borrado de objetos es similar a la actualización: debemos obtener el objeto también, pero después llamamos al método `remove` para borrarlo, y finalmente a `flush`. 

Por ejemplo:

![image-20260702084004000](/symfony-contactos-teoria/static/assets/image-20260702084004000.png)

Nuevamente, tanto en la actualización como en el borrado, el método `flush` puede provocar una **excepción** si la operación no ha podido llevarse a cabo. Debemos tenerlo en cuenta para capturarla y generar la respuesta oportuna.

## 3.3 Relaciones entre entidades

Hasta ahora las operaciones que hemos hecho se han centrado en una única tabla o entidad (la entidad/tabla de `contactos`). Veamos ahora cómo podemos trabajar con más de una `tabla/entidad` que estén relacionadas entre sí.

Existen dos tipos principales de relaciones entre entidades:

* **Muchos a uno**: en este tipo se englobarían las relaciones “uno a muchos”, “muchos a uno” y “uno a uno”, ya que en cualquiera de los tres casos, la relación se refleja añadiendo una clave ajena en una de las entidades que referencie a la otra.
* **Muchos a muchos**: en este tipo de relaciones, se necesita de una tabla adicional para reflejar la relación entre las entidades.

Vamos a definir una relación muchos a uno en nuestra base de datos de contactos. Para ello, vamos a crear primero una entidad llamada `Provincia`, que sólo contenga un `id` autogenerado y un `nombre` (string):

```bash
php bin/console make:entity
```

 Y definimos los campos:

<script id="asciicast-seD4fxqXUY9wSCkjJEUk5sqfj" src="https://asciinema.org/a/seD4fxqXUY9wSCkjJEUk5sqfj.js" async data-size="medium"></script>

Tras generar la nueva entidad, creamos la correspondiente tabla en la base de datos a través de la migración.

```bash
php bin/console make:migration
php bin/console doctrine:migration:migrate
```

Ahora, vamos a hacer que los contactos tengan una provincia asociada. Para ello, editamos la entidad `Contacto` y le añadimos un nuevo campo, llamado `provincia`, que será de tipo relación muchos a uno (un contacto pertenecerá a una provincia, y una provincia puede tener muchos contactos).

<script id="asciicast-EIgPamUamfQOm4N6xS2vIzXGP" src="https://asciinema.org/a/EIgPamUamfQOm4N6xS2vIzXGP.js" async></script>

Como puede verse, a la hora de elegir el tipo de campo, indicamos que es una relación (`relation`), en cuyo caso nos pide indicar a qué entidad está vinculada (`Provincia`, en este caso), y qué tipo de relación es (`ManyToOne` en nuestro caso, pero podemos elegir cualquiera de las otras tres opciones `OneToMany`, `OneToOne` o `ManyToMany`). También podemos comprobar que el asistente nos pregunta si queremos añadir un campo en la otra entidad para que la relación sea bidireccional (es decir, para que desde un objeto de cualquiera de las dos entidades podamos consultar el/los objeto(s) asociado(s) de la otra. En este caso indicamos que **no** para simplificar el código.

Una vez creada la relación vamos a realizar la migración:

```bash
php bin/console make:migration
php bin/console doctrine:migration:migrate
```

> -hint- Los comandos  se pueden **abreviar**. Por ejemplo, `doctrine:migration:migrate` se convierte en `d:m:m` , `make:migration` en `m:mi`. Solo se pueden abreviar hasta que no produzcan ambigüedad.
>
> Por ejemplo, si intentamos `m:m` salta la siguiente información:
>
> ![image-20260702084619368](/symfony-contactos-teoria/static/assets/image-20260702084619368.png)

Ya tendremos el nuevo campo añadido en nuestra entidad `Contacto` y a la tabla contacto de la base de datos:

![1549386995547](/symfony-contactos-teoria/static/assets/1549386995547-1782975215910-1.png)

```php
<?php
//src/Entity/Contacto
#[ORM\ManyToOne(inversedBy: 'contactos')]
private ?Provincia $provincia = null;
```

de tal forma que podemos obtener la provincia de un contacto y de ahí, cualquier campo:

```php
$contacto->getProvincia()->getNombre();
```

o en un plantilla

```twig
{{ contacto.provicia.nombre}}
```

y en `Provincia`

```php
//src/Entity/Provincia
/**
 * @var Collection<int, Contacto>
 */
#[ORM\OneToMany(targetEntity: Contacto::class, mappedBy: 'provincia')]
private Collection $contactos;
```

De tal forma que podemos acceder a todos los contactos de una provincia:

```php
provincia->getContactos()
```

 y en twig

```twig
{% for contacto in provincia.contactos %}
```



### 3.3.1 Modificar plantilla

Vamos a mostrar el campo `provincia` del `contacto`

> -warning-
>
> Aseguraos que tenéis registros en la tabla `provincias` y actualizado el campo `id_provincia` el `contacto`

```twig
<ul>
	<li>
		<strong>{{ contacto.nombre }}</strong>
	</li>
	<li>
		<strong>Teléfono</strong>:
		{{ contacto.telefono }}</li>
	<li>
		<strong>E-mail</strong>:
		{{ contacto.email }}</li>
	<li>
		<strong>Provincia</strong>:
		{{ contacto.provincia.nombre ?? 'Sin provincia' }}</li>
</ul>

```

### 3.3.2 Modificar formulario

Ahora nos falta añadir el campo `provincia` en el formulario de contactos.

```php
<?php

namespace App\Form;

use App\Entity\Contacto;
use App\Entity\Provincia;
use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\OptionsResolver\OptionsResolver;
use Symfony\Component\Form\Extension\Core\Type\SubmitType;
use Symfony\Component\Form\Extension\Core\Type\EmailType;
use Symfony\Bridge\Doctrine\Form\Type\EntityType;

class ContactoFormType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options): void
    {
        $builder
            ->add('nombre')
            ->add('telefono')
            ->add('email', EmailType::class, array('label' => 'Correo electrónico'))
            ->add('provincia', EntityType::class, [
                'class' => Provincia::class,
                'choice_label' => 'nombre',
                'label' => 'Provincia',
            ])
            ->add('save', SubmitType::class, array('label' => 'Enviar'));
    }

    public function configureOptions(OptionsResolver $resolver): void
    {
        $resolver->setDefaults([
            'data_class' => Contacto::class,
        ]);
    }
}

```

El campo provincia es de tipo `EntityType`, la entidad subyacente es `Provincia`, el texto que el usuario ve es `nombre` y la etiqueta que ve el usuario es `Provincia`

Ahora comprobad que los controladores funcionan igualmente sin haber cambiado una sola línea de código.

