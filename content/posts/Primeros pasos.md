---
typora-copy-images-to: ../../static/assets/
typora-root-url: ../../../
layout: post
slug: primeros-pasos
conToc: true
title : Primeros pasos
date: 2022-09-04T19:50:07+01:00
---
Para crear un proyecto, se utiliza el comando

```
composer create-project symfony/skeleton:"6.4.*" symfony-contactos
```

Que creará un proyecto con el nombre indicado en la carpeta actual, conteniendo la estructura mínima, sin librerías de terceros. Será nuestra responsabilidad añadirlas más tarde. Esta funcionalidad ha sido añadida en la versión 4 de Symfony, para permitir que se instale como microframework y no dejar un proyecto demasiado pesado para nuestras necesidades.

`composer` es un gestor de paquetes o dependencias de php.

Este comando generará un proyecto con la siguiente estructura:

![Estructura proyecto](/symfony-contactos-teoria/assets/image-20220103120234751-1697018612811.png)

## 1.1 Inicio

> -info-**Controladores**
> Un controlador es una clase PHP que conecta rutas con métodos, de tal forma que, cuando el usuario escribe una ruta en el navegador, Symfony ejecuta el método que responde a la misma.
>
> En cualquier framework de desarrollo web, se definen, de alguna forma, las rutas y los métodos que responden a ellas.

Primero instalamos la dependencia necesaria para crear controladores para desarrollo:

```
composer require symfony/maker-bundle --dev
```

Vamos a crear nuestro primer controlador.

```
php bin/console make:controller PageController
```

Este comando genera un controlador `PageController` en la carpeta `src/Controller`. El código que genera es el siguiente:

```php
<?php

namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

class PageController extends AbstractController
{
    #[Route('/page', name: 'app_page')]
    public function index(): JsonResponse
    {
        return $this->json([
            'message' => 'Welcome to your new controller!',
            'path' => 'src/Controller/PageController.php',
        ]);
    }
}
```

Lo que hace es asociar una ruta a un método. En este ejemplo, asocia el método `index` con la ruta `/page/`,  a esta ruta la nombra `app_page`y muestra la respuesta `JsonResponse` en formato JSON.

Vamos a levantar un servidor de desarrollo mediante el comando `php -S 127.0.0.1:8080` dentro de la carpeta `public`

Si ahora visitamos la página [http://127.0.0.1:8080/page](http://127.0.0.1:8080/page) este será el resultado:

![image-20250918092739108](/symfony-contactos-teoria/static/assets/image-20250918092739108.png)



Vemos que el código renderiza la plantilla `page/index.html.twig` pasándole como parámetro `controller_name`.

Ahora crea un método para la ruta para `/` llamado `inicio` con el siguiente código:

![image-20231011115643513](/symfony-contactos-teoria/assets/image-20231011115643513.png)

Al visitar la url [http://127.0.0.1:8080/](http://127.0.0.1:8080/) este será el resultado:

![image-20220103121930288](/symfony-contactos-teoria/assets/image-20220103121930288-1697018822174.png)

## 1.2 Ficha

Vamos a crear la ficha del contacto. Para ello, crea otro controlador llamado `ContactoController` y una ruta para que muestre el contacto con código igual al `$codigo`

![image-20260630115432185](/symfony-contactos-teoria/static/assets/image-20260630115432185.png)

En este caso vemos el uso de los parámetros en los controladores. El parámetro `codigo` va entre llaves y será automáticamente inyectado por Symfony,

Por ejemplo, si visitas la página [http://127.0.0.1:8080/contacto/2](http://127.0.0.1:8080/contacto/2) se mostrará la siguiente ventana:

![image-20220103165623881](/symfony-contactos-teoria/assets/image-20220103165623881-1697018905697.png)

### 1.2.1 Base de datos

> -info-Un **ORM (Object Relational Mapping)** es un framework encargado de tratar con una base de datos relacional (conectar con ella, realizar operaciones de consulta, inserción, etc.), de forma que, de cara a la aplicación, se convierten a objetos todos los elementos que se extraigan de la base de datos, y viceversa (los objetos de la aplicación se transforman en registros de la base de datos, llegado el caso).
>
> De esta forma, el ORM se encargará de realizar esta conversión o mapeo automáticamente por nosotros. Definiendo una serie de reglas, indicaremos qué tablas de la base de datos relacional se corresponden con qué clases de nuestro modelo, y qué campos de cada tabla se corresponden con qué atributos de cada clase. A partir de ahí, el ORM se encargará de extraer la información de la base de datos y crear los objetos correspondientes, o de convertir los objetos con sus atributos en registros de la base de datos, con sus correspondientes columnas.
>
> La principal ventaja de utilizar un ORM como Doctrine es aislar la aplicación del gestor de base de datos que hayamos elegido (MySQL, Oracle, PostgreSQL...) ya que a nivel de aplicación trabajaremos con objetos, y será Doctrine quien se encargue de conectar con la base de datos elegida, y transformar los objetos para adaptarlos a la misma.

El primer paso va a ser instalar el componte ORM

```
composer require symfony/orm-pack
```

> -info-**Archivos de variables de entorno**
> Para poder utilizar Doctrine, tenemos que indicar cómo conectar al servidor de base de datos que vayamos a utilizar. Estos parámetros de conexión se pueden configurar en el archivo `.env` de nuestro proyecto. Este es un archivo donde se definen ciertas variables propias de entorno, que luego se procesan y se convierten en variables reales. En nuestro caso, definimos una llamada `DATABASE_URL`, con una `URL` donde se especifican tanto la dirección y puerto de conexión a la base de datos, como el `login` y `password` necesarios para acceder, y el nombre de la base de datos a la que conectar. Por ejemplo, para una base de datos MySQL, la estructura general será ésta:
>
> ```properties
> DATABASE_URL=mysql://root:sa@127.0.0.1:3306/contactos
> ```

Usaremos Doctrine para crear la base de datos, aunque también se puede hacer directamente con [phpmyadmin](http://127.0.0.1/phpmyadmin/)

```
php bin/console doctrine:database:create
```

El siguiente paso es crear las entidades

> -info-**Entidad**
>
> Una entidad es una clase que se mapea con un tabla de la base de datos

### 1.2.2 Creación de entidades

Las entidades son las clases que van a componer el modelo de datos de nuestra aplicación. Por ejemplo, para nuestra aplicación de contactos, necesitaremos una entidad/clase llamada `Contacto` que almacene los datos concretos de cada contacto (código, nombre, teléfono y e­mail).
Para crear una entidad, empleamos el siguiente comando desde el terminal (dentro de la carpeta principal de nuestro proyecto Symfony):

```bash
php bin/console make:entity
```

Se iniciará un asistente que nos irá pidiendo información para construir la entidad:

* Nombre de la clase o entidad
* Propiedades o atributos de la clase, para cada uno, pedirá el nombre (si directamente pulsamos Intro dejará de pedirnos más datos), el tipo de dato, la longitud o tamaño del campo, si admite nulos...

<script id="asciicast-bc8O8iC56qHQ33JnAuvjC1EBT" src="https://asciinema.org/a/bc8O8iC56qHQ33JnAuvjC1EBT.js" async data-size="medium"></script>

Como resultado, se generará una clase `Contacto` dentro de la carpeta `src/Entity`. El código queda como sigue:

```php
<?php

namespace App\Entity;

use App\Repository\ContactoRepository;
use Doctrine\ORM\Mapping as ORM;

#[ORM\Entity(repositoryClass: ContactoRepository::class)]
class Contacto
{
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column]
    private ?int $id = null;

    #[ORM\Column(length: 255)]
    private ?string $nombre = null;

    #[ORM\Column(length: 255)]
    private ?string $telefono = null;

    #[ORM\Column(length: 255)]
    private ?string $email = null;

    public function getId(): ?int
    {
        return $this->id;
    }

    public function getNombre(): ?string
    {
        return $this->nombre;
    }

    public function setNombre(string $nombre): static
    {
        $this->nombre = $nombre;

        return $this;
    }

    public function getTelefono(): ?string
    {
        return $this->telefono;
    }

    public function setTelefono(string $telefono): static
    {
        $this->telefono = $telefono;

        return $this;
    }

    public function getEmail(): ?string
    {
        return $this->email;
    }

    public function setEmail(string $email): static
    {
        $this->email = $email;

        return $this;
    }
}
```

En cuanto a los tipos de datos que podemos especificar, si pulsamos `?` e `Intro` cuando vayamos a especificar el tipo de dato, veremos un listado completo de los tipos disponibles (también lo podéis consultar [aquí](https://www.doctrine-project.org/projects/doctrine-dbal/en/4.4/reference/types.html)). Lo habitual será trabajar con cadenas de texto de una longitud determinada (`string`), textos ilimitados (`text`), enteros (`integer`), booleanos (`boolean`), reales (`float`), fechas (`date`, `time` o `datetime`, dependiendo de lo que queramos almacenar)...

También nos ha creado un **repositorio** que es una clase que nos permite realizar consultas sobre la entidad subyaciente.

```php
<?php

namespace App\Repository;

use App\Entity\Contacto;
use Doctrine\Bundle\DoctrineBundle\Repository\ServiceEntityRepository;
use Doctrine\Persistence\ManagerRegistry;

/**
 * @extends ServiceEntityRepository<Contacto>
 */
class ContactoRepository extends ServiceEntityRepository
{
    public function __construct(ManagerRegistry $registry)
    {
        parent::__construct($registry, Contacto::class);
    }
}
```



### 1.2.3 Generación del esquema

Una vez hemos definida la entidad, podemos generar la correspondiente tabla en la base de datos. Para ello, escribimos este comando:

```
php bin/console make:migration
```

Lo que hace este comando es cotejar los cambios entre nuestro modelo de entidades y el esquema de la base de datos, y generar un archivo PHP que se encargará de volcar esos cambios a la base de datos. Por consola se nos informará de dónde está este archivo para que lo comprobemos (estará en la carpeta `src/migrations`), y si todo es correcto, ejecutando este otro comando se reflejarán los cambios en la base de datos:

```bash
php bin/console doctrine:migration:migrate
```

![1549382168090](/symfony-contactos-teoria/static/assets/1549382168090-1782814460374-1.png)

### 1.2.4 Inserción de datos de forma manual

Vamos a insertar unos contactos en phpmyadmin:

```sql
INSERT INTO `contacto` (`id`, `nombre`, `telefono`, `email`) VALUES
(1, 'Pedro', '9586425982289', 'pedroparamo@micuenta.com'),
(2, 'Juan', '8866599', 'juanito265@gserver.com'),
(3, 'María', 'j8987(ext 10)', 'maria2022@gserver.com'),
(4, 'Elena', 'y8557cckk', 'elena89854@gserver.com');
```

### 1.2.5 Obtener objetos

Y modificamos el controlador, para que ahora nos muestre los datos del contacto en la base de datos pasado como parámetro:

```php
// Si queremos validar un parámetro, su usa 'requeriments' que es una expresión regular. En este caso, solo permite números de longitud variable
#[Route('/contacto/{codigo}', name: 'contacto', requirements: ['codigo' => '[0-9]+'])]
// Symfony inyecta la dependencia ManagerRegistry automáticamente
// Le pasa la variable $codigo con el valor en {codigo}. Si no se le pasa, coge 1 por defecto, en otro caso, daría not found
public function ficha(ManagerRegistry $doctrine, int $codigo = i): Response
{
    // La primera instrucción suele ser esta, ya que cogemos el repositorio de la entidad asociada
    $repositorio = $doctrine->getRepository(Contacto::class);
    // Ahora usamos uno de los métodos del repositorio
    $contacto = $repositorio->find($codigo);
    // Y creamos la vista HTML
    $html = "
    <h1>Detalle del contacto</h1>
    <p>Nombre: " . $contacto->getNombre() . "</p>
    <p>Teléfono: " . $contacto->getTelefono() . "</p>
    <p>Email: " . $contacto->getEmail() . "</p>
    ";
    // Devolvemos como respuesta el html
    return new Response($html);
}
```

Los objetos siempre se obtienen de un repositorio:

```php
<?php
 $repositorio = $doctrine->getRepository(Contacto::class);
```

donde lo único que varía es el nombre de la `Entidad`,

A la hora de obtener objetos de una tabla, existen diferentes métodos que podemos emplear. Por ejemplo:

* El método `find` localiza el objeto por la clave primaria (normalmente el id) que se le pasa como parámetro. Así buscaríamos el contacto con id 1:

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

Una vez recogidos los datos, generamos la salida:

```php
$html = "
    <h1>Detalle del contacto</h1>
    <p>Nombre: " . $contacto->getNombre() . "</p>
    <p>Teléfono: " . $contacto->getTelefono() . "</p>
    <p>Email: " . $contacto->getEmail() . "</p>
    ";
return new Response($html);
```

### 1.2.6 Añadir valores predeterminados

En algunas ocasiones, también nos puede interesar dar un valor por defecto a una wildcard para que, si en la ruta no se especifica nada, tenga dicho valor por defecto. Esto se consigue asignando un valor por defecto al parámetro asociado en el controlador. En el caso de la ficha del contacto anterior, si quisiéramos que cuando se introduzca la ruta `/contacto` (sin código), se mostrara por defecto el contacto con código 1, haríamos esto:

```php
<?php
#[Route('/contacto/{codigo?1}', name: 'contacto', requirements: ['codigo' => '[0-9]+'])]
```

## 1.3 Plantillas

> -info-**Plantillas**
>
> Las plantillas (templates) son archivos de texto, creados en un lenguaje propio del sistema de plantillas, que permiten mezclar fácilmente los datos de la base de datos y html. En cualquier framework, existe un sistema de plantillas. En el caso de Symfony, es Twig

El primer paso va a ser instalar el componente de plantilla `twig`:

```
composer require symfony/twig-bundle
```

Y creamos nuestra primera plantilla, `inicio.html.twig` en la carpeta `templates`.

```html
<!doctype html>
<html>
<meta charset="utf-8">
<body>
	<h1>Contactos</h1>
	<h2>Bienvenido a la web de contactos.</h2>
	<p>Página de inicio</p>
</body>
</html>
```

y modificamos también el método `inicio` del controlador `PageController` para que, en lugar de mostrar una respuesta de texto plano, renderice la vista `inicio.html.twig` que acabamos de hacer. Para ello, el código será el siguiente:

![image-20231011122855181](/symfony-contactos-teoria/assets/image-20231011122855181.png)

Observa que se utiliza `$this`. Esto es así porque el controlador hereda de `AbstractController` y este es uno de los métodos que posee.

### 1.3.1 Plantillas con partes variables

La plantilla anterior no es algo habitual, ya que únicamente contiene texto estático. Lo normal es que haya alguna parte que provenga de la base de datos, y que le sea proporcionada desde el controlador.

Vamos a ver un ejemplo. Para ello creamos `ficha_contacto.html.twig` con el siguiente contenido:

```html
<!doctype html>
<html>
<meta charset="utf-8">
<body>
    <h1>Ficha del contacto</h1>
    <ul>
        <li><strong>{{contacto.nombre}}</strong></li>
        <li><strong>Teléfono: </strong>{{contacto.telefono}}</li>
        <li><strong>Correo: </strong>{{contacto.email}}</li>
    </ul>
</body>
</html>
```

Empleamos la notación de la doble llave `{{ ... }}` para ubicar variables, que normalmente son datos que esperamos recibir de fuera (del controlador, en este caso). Nos faltaría, en el método `ficha` de `ContactoController`, obtener el contacto deseado (eso ya lo tenemos hecho) y pasárselo a la vista, de este modo:

![image-20260701090726562](/symfony-contactos-teoria/static/assets/image-20260701090726562.png)

Al llamar al método `render` le pasamos a la plantilla la variable `$contacto` y en la plantilla, accedemos a ella mediante `{{contacto.nombre-del-campo}}`

### 1.3.2 Estructuras de control en plantillas

La plantilla anterior es un ejemplo para añadir partes dinámicas en el contenido de la misma, pero está algo *coja*: ¿qué pasa si no encontramos el contacto en la lista?. Si no adoptamos ninguna solución, se mostrará un error 500

![image-20260701091828534](/symfony-contactos-teoria/static/assets/image-20260701091828534.png)

Para solucionarlo, creamos un condicional:

```twig
<!doctype html>
<html>
	<meta charset="utf-8">
	<body>
		{%if contacto%}
			<h1>Ficha del contacto</h1>
			<ul>
				<li>
					<strong>{{contacto.nombre}}</strong>
				</li>
				<li>
					<strong>Teléfono:
					</strong>
					{{contacto.telefono}}</li>
				<li>
					<strong>Correo:
					</strong>
					{{contacto.email}}</li>
			</ul>
			{%else%}
			<p>No se ha encontrado el contacto</p>
			{%endif%}
	</body>
</html>
```

y la vista distinguirá si hay o no contacto, para mostrar una u otra información

Observa cómo hemos incluido un bloque `{% ... %}`, que son **bloques de acción**, empleados para definir ciertas sentencias de control (condiciones, bucles) e incluir dentro el código asociado a dicha sentencia.

Resumiendo:

* `{{variable.dato}}` para escribir
* `{% %}` para los bloques de acción
* `{# comentarios #}` para comentarios

### 1.3.3 Herencia de plantillas

La herencia de plantillas nos permite reaprovechar el código de unas en otras. En realidad, esto es algo muy habitual en el diseño web: que todas las páginas (o varias) de una web compartan la misma cabecera y pie, por ejemplo. Así, podemos definir una estructura o layout base en una plantilla, y hacer que otra(s) hereden de ella para rellenar ciertos huecos. Veamos un ejemplo con nuestra web de contactos.

En primer lugar, definiremos la plantilla base. Tenéis un ejemplo en que basaros ya hecho, en el archivo `templates/base.html.twig`, que proporciona un esqueleto parecido a este que podríamos aprovechar para muchas aplicaciones:

```twig
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <title>{% block title %}Welcome!{% endblock %}</title>
        <link rel="icon" href="data:image/svg+xml,<svg xmlns=%22http://www.w3.org/2000/svg%22 viewBox=%220 0 128 128%22><text y=%221.2em%22 font-size=%2296%22>⚫️</text><text y=%221.3em%22 x=%220.2em%22 font-size=%2276%22 fill=%22%23fff%22>sf</text></svg>">
        {% block stylesheets %}
        {% endblock %}

        {% block javascripts %}
        {% endblock %}
    </head>
    <body>
        {% block body %}{% endblock %}
    </body>
</html>
```

Como podemos observar, la parte *rellenable* de la plantilla se define mediante bloques (`blocks`), de forma que en las diferentes plantillas hija podemos indicar qué bloques de la plantilla padre queremos rellenar. Por ejemplo, vamos a definir una plantilla hija para la página de inicio. Retocamos nuestra plantilla `inicio.html.twig` y la dejamos así:

![image-20260701092452955](/symfony-contactos-teoria/static/assets/image-20260701092452955.png)

Es importante que, si una plantilla hereda de otra, el primer código que haya en esa plantilla (sin contar comentarios previos) sea una instrucción `{% extends ... %}` para indicar que es una herencia. Después, basta con rellenar los bloques cuyo contenido queramos modificar o establecer: en este ejemplo, los bloques `title` y `body`, definidos en la plantilla base.

Del mismo modo, definiríamos la plantilla `ficha_contacto.html.twig`

![image-20260701092616330](/symfony-contactos-teoria/static/assets/image-20260701092616330.png)

### 1.3.4 Incluir plantillas dentro de otras

Otra opción interesante, aparte de la herencia, es la de poder incluir una plantilla como parte del contenido de otra. Basta con utilizar la instrucción `include`, seguida del nombre de la plantilla y, si los necesita, sus parámetros asociados. Por ejemplo, podríamos sacar la lista de datos de un contacto a una plantilla llamada `partials/_contacto.html.twig`:

```php
<ul>
    <li><strong>{{ contacto.nombre }}</strong></li>
    <li><strong>Teléfono</strong>: {{ contacto.telefono }}</li>
    <li><strong>E-mail</strong>: {{ contacto.email }}</li>
</ul>
```

E incluirla tanto en `ficha_contacto.html.twig`

![image-20260701094159051](/symfony-contactos-teoria/static/assets/image-20260701094159051.png)

### 1.3.5 Añadir contenido estático en plantillas

Es necesario instalar el componente `assets` mediante el comando:

```
composer require symfony/asset
```

Para ilustrar cómo añadir contenido estático en plantillas (archivos de estilo, javascript e imágenes), vamos a definir en nuestra carpeta `public` de la web de contactos una subcarpeta `css`, y dentro un archivo `estilos.css` (que quedará, por tanto, en `public/css/estilos.css`). Definimos dentro un estilo básico para probar. Por ejemplo:

```css
body
{
    background-color: #99ccff;
}

h1
{
    border-bottom: 1px solid black;
}
```

Ahora, vamos a añadir este estilo a nuestra web. Como tenemos un bloque `stylesheets` en nuestra plantilla `base.html.twig`, podemos aprovecharlo e incluir el CSS dentro de dicho bloque, para que lo utilicen todas las subplantillas:

```twig
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1">
        <title>{% block title %}Welcome!{% endblock %}</title>
        {% block stylesheets %}
            <link href="{{ asset('css/estilos.css') }}" rel="stylesheet" />
        {% endblock %}
    </head>
    <body>
        {% block body %}{% endblock %}
        {% block javascripts %}{% endblock %}
    </body>
</html>
```

## 1.4 El patrón Modelo-Vista-Controlador (MVC)

MVC es una forma de organizar el código de una aplicación separando responsabilidades en **tres partes** que trabajan juntas pero están desacopladas entre sí. El objetivo es que el código sea más fácil de mantener, testear y ampliar.

### Las tres piezas

#### 1. Modelo (Model)

Es la parte que se encarga de los **datos y la lógica de negocio**.

- Representa las entidades de tu aplicación (por ejemplo, `Usuario`, `Producto`, `Pedido`).
- Se comunica con la base de datos (consultas, inserciones, actualizaciones...).
- Contiene las reglas de negocio: validaciones, cálculos, restricciones.
- **No sabe nada de cómo se muestran los datos.** No tiene ni idea de si el resultado va a salir en HTML, en JSON o en una app móvil.

#### 2. Vista (View)

Es la parte encargada de la **presentación**.

- Muestra los datos al usuario (una plantilla HTML, por ejemplo).
- No contiene lógica de negocio, solo lógica de presentación (bucles para pintar una lista, condicionales para mostrar u ocultar algo...).
- Recibe los datos ya procesados, normalmente desde el controlador.

#### 3. Controlador (Controller)

Es el **intermediario** entre el Modelo y la Vista.

- Recibe las peticiones del usuario (por ejemplo, cuando entra a una URL o envía un formulario).
- Decide qué hacer: pide datos al Modelo, los procesa si hace falta.
- Envía esos datos a la Vista para que los muestre.
- No accede directamente a la base de datos ni genera HTML él mismo.

```
Usuario → Controlador → Modelo (consulta BD)
                ↓
              Vista (genera HTML) → Usuario
```

### ¿Por qué se usa esto?

- **Separación de responsabilidades**: cada parte hace una cosa y la hace bien. Si cambias el diseño (CSS/HTML), no tocas la lógica de negocio.
- **Reutilización**: el mismo Modelo puede servir a varias Vistas (una web, una API, una app).
- **Mantenimiento**: es más fácil encontrar y arreglar errores cuando el código está organizado.
- **Trabajo en equipo**: un compañero puede maquetar las vistas mientras otro programa la lógica del modelo, sin pisarse el trabajo.

## 1.5 Recuperar múltiples objetos.

Vamos a modificar la página de portada para que muestre una lista con todos los contactos:

```php
#[Route('/', name: 'inicio')]
public function inicio(ManagerRegistry $doctrine): Response
{
    $repositorio = $doctrine->getRepository(Contacto::class);
    // findAll es un método que se encuentra en el repositorio
    $contactos = $repositorio->findAll();
    //Mostramos la plantilla pasándole los contactos
    return $this->render("inicio.html.twig", ["contactos" => $contactos]);
}
```

Y modificamos la plantilla `inicio.html.twig` para listar los contactos

```twig
{% extends 'base.html.twig' %}
{% block body %}
	<h1>Contactos</h1>
	<h2>Bienvenido a la web de contactos.</h2>
	<p>Página de inicio</p>
	{% for contacto in contactos %}
		 {{ include ('partials/_contacto.html.twig', {'contacto': contacto})}}
	{% endfor %}
{% endblock %}
```

## 1.6 Errores más comunes

### 1.6.1 Ruta no encontrada

![image-20260701094815811](/symfony-contactos-teoria/static/assets/image-20260701094815811.png)

Pues eso. La ruta no coincide con ningún controlador.

Posible causa:

* la ruta está mal escrita. En el ejemplo, `http://localhost:8080/contato/1` en vez de `http://localhost:8080/contacto/1`

:ok_hand: **Solución**: **revisa bien la url**

Para comprobar qué rutas hay, usa el siguiente comando:

```
php bin/console debug:route --show-controllers
```

![image-20260701111508603](/symfony-contactos-teoria/static/assets/image-20260701111508603.png)

Si quieres comprobar qué ruta coincide con un path, usa

```
php bin/console router:match ruta-a-comprobar
```

Por ejemplo:

```
php bin/console router:match /contacto/1
```

![image-20260701111132425](/symfony-contactos-teoria/static/assets/image-20260701111132425.png)

Si no encuentra ninguna:

![image-20260701111209305](/symfony-contactos-teoria/static/assets/image-20260701111209305.png)

### 1.6.2 Un campo no existe

![image-20260701111706662](/symfony-contactos-teoria/static/assets/image-20260701111706662.png)

:ok_hand: **Solución**: **revisa bien el nombre del campo en la entidad**

En este caso, el campo se llama `email`

### 1.6.3 Una variable no existe

![image-20260701111917334](/symfony-contactos-teoria/static/assets/image-20260701111917334.png)

:ok_hand: **Solución**: **revisa bien el nombre de la variable en la plantilla y también revisa el nombre de la variable en el controlador**

![image-20260701112104691](/symfony-contactos-teoria/static/assets/image-20260701112104691.png)

Fíjate que dice `contato` no `contacto`

### 1.6.4  No encuentra la plantilla 

![image-20260701112501052](/symfony-contactos-teoria/static/assets/image-20260701112501052.png)

:ok_hand: **Revisa el nombre tanto de la plantilla en el controlador como del archivo físico de la misma**

En este caso, lo correcto es `ficha_contacto.html.twig` y no `ficha_contato.html.twig`

![image-20260701112907891](/symfony-contactos-teoria/static/assets/image-20260701112907891.png)

También puede ser que el nombre del archivo de la plantilla tenga algún gazapo:

:ok_hand: **Lee uno a uno los caracteres hasta encontrar el gazapo**

![image-20260701113158073](/symfony-contactos-teoria/static/assets/image-20260701113158073.png)

### 1.6.5 Qué no hacer

Cuando os da un error, muestra la línea que lo ha producido. Para solucionarlo, solo mirar en aquellos archivos que habéis tocado y dejad a una lado los del propio framework, pues seguro que el error no está ahí. Por ejemplo,

![image-20260701114404877](/symfony-contactos-teoria/static/assets/image-20260701114404877.png)

El error no estará en el archivo `vendor/autoload_runtime.php`,  buscad  en vuestro código.





