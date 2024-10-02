---
typora-copy-images-to: ../symfony-contactos-teoria/assets
typora-root-url: ../../
layout: post
slug: generacion-de-formularios
conToc: true
title : Generación de formularios
date: 2022-09-02T19:50:07+01:00
---

Hasta este apartado hemos aprendido algunos conceptos útiles de Symfony y algunos de sus bundles más destacados, como por ejemplo la generación de vistas con el motor de plantillas Twig y  la comunicación con la base de datos a través del ORM Doctrine. Hemos hecho algunos controladores de ejemplo para buscar datos, o para insertar. Pero, en este último caso, al no disponer aún de un mecanismo para que se envíen datos de inserción desde el cliente, hemos optado por ahora por insertar unos datos prefijados o dummy data, es decir, un contacto con unos datos ya predefinidos en el código.

Para el funcionamiento de un formulario nos hace falta:
* Un formulario definido en su propia clase
* Un método en el controlador
* Una plantilla que muestre el formulario

## 3.1 Creación de la clase para el formulario
Para crear el formulario usaremos el `maker bundle`:
```bash
php bin/console make:form ContactoForm Contacto
```
Donde `ContactoForm` es el nombre de la clase a crear y `Contacto` es el nombre de la entidad.

Este comando nos generará un formulario por defecto en la carpeta `forms` con el siguiente contenido:

```php
<?php

namespace App\Form;

use App\Entity\Contacto;
use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\OptionsResolver\OptionsResolver;
use Symfony\Component\Form\Extension\Core\Type\SubmitType;

class ContactoType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options): void
    {
        $builder
            ->add('nombre')
            ->add('telefono')
            ->add('email')
            ->add('provincia')
            ->add('save', SubmitType::class, array('label' => 'Enviar'));
        ;
    }

    public function configureOptions(OptionsResolver $resolver): void
    {
        $resolver->setDefaults([
            'data_class' => Contacto::class,
        ]);
    }
}
```

Por defecto, cada campo lo crea de tipo `TextType`, es decir, un `<input>` de tipo `text`. Además le hemos añadido un botón para enviar el formulario.

## 3.2 Creación del controlador

Vamos a crear un método para renderizar el formulario:

```php
#[Route('/contacto/nuevo', name: 'nuevo')]
public function nuevo(ManagerRegistry $doctrine, Request $request) {
        $contacto = new Contacto();
        $formulario = $this->createForm(ContactoType::class, $contacto);
        $formulario->handleRequest($request);

        if ($formulario->isSubmitted() && $formulario->isValid()) {
            $contacto = $formulario->getData();
            
            $entityManager = $doctrine->getManager();
            $entityManager->persist($contacto);
            $entityManager->flush();
            return $this->redirectToRoute('ficha_contacto', ["codigo" => $contacto->getId()]);
        }
        return $this->render('nuevo.html.twig', array(
            'formulario' => $formulario->createView()
        ));
    }
```

* la línea <span style=color:red>3</span> crea un objecto de la clase `Contacto`
* la línea <span style=color:red>4</span> crea el formulario mediante la clase que define el formulario y la entidad base del mismo
* la línea <span style=color:red>6</span> comprueba si el formulario ha sido enviado y también comprueba si es válido, que veremos más adelante.
* la línea <span style=color:red>7</span> fija los datos de la entidad con los datos del formulario
* las lineas <span style=color:red>9-11</span> guardan los datos en la BD.
* las líneas <span style=color:red>14-17</span> permiten renderizar la plantilla, pasándole un parámetro llamado `formulario`

## Plantilla

Esta es la plantilla

```twig
{% extends 'base.html.twig' %}

{% block title %}Contactos{% endblock %}
{% block body %}
    <h1>Nuevo contacto</h1>
    {{ form(formulario) }}
{% endblock %}
```

la línea <span style=color:red>6</span> renderiza el plantilla

Si ahora accedemos a [http://127.0.0.1:8080/contacto/nuevo](http://127.0.0.1:8080/contacto/nuevo) podremos ver el formulario:

![image-20220109174250630](/symfony-contactos-teoria/assets/image-20220109174250630.png)

Existen multitud de tipos de campo, entre los que están lo siguientes:

* `TextType` (cuadros de texto de una sola línea, como el ejemplo anterior. Son el control por defecto)
* `TextareaType` (cuadros de texto de varias líneas)
* `EmailType` (cuadros de texto de tipo e­mail)
* `IntegerType` (cuadros de texto para números enteros)
* `NumberType` (cuadros de texto para números en general)
* `PasswordType` (cuadros enmascarados para passwords)
* `EntityType` (desplegables para elegir valores vinculados a otra entidad)
* `DateType` (para fechas)
* `CheckboxType` (para checkboxes)
* `RadioType` (para botones de radio o radio buttons)
* `HiddenType` (para controles ocultos)
* ... etc.

Puedes acceder a todos los tipos de campos [aquí](https://symfony.com/doc/current/reference/forms/types.html)

### 3.2.1 Etiquetas personalizadas

Como podemos ver para el caso del botón de `submit`, podemos especificar un tercer parámetro en el método `add` que es un array de propiedades del control en cuestión. Una de ellas es la propiedad `label`, que nos permite especificar qué texto tendrá asociado el control. Por defecto, se asocia el nombre del atributo correspondiente en la entidad, pero podemos cambiarlo por un texto personalizado. Para el `e­mail`, por ejemplo, podríamos poner:

```php
<?php
->add('email', EmailType::class, array('label' => 'Correo electrónico'))
```

![image-20220109174433735](/symfony-contactos-teoria/assets/image-20220109174433735.png)

### 3.2.2 Mejorando nuestro formulario

Aprovechando la variedad de tipos de campos que ofrece Symfony, vamos a mejorar un poco nuestro formulario añadiendo el dato Provincia que vimos en el apartado de `Doctrine`. Para ello podemos emplear un `EntityType` que tome sus datos de la entidad `Provincia`.

Con esto, el formulario quedaría así:

```php
<?php

namespace App\Form;

use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\Form\Extension\Core\Type\TextType;
use Symfony\Component\Form\Extension\Core\Type\HiddenType;
use Symfony\Component\Form\Extension\Core\Type\EmailType;
use Symfony\Component\Form\Extension\Core\Type\SubmitType;
use Symfony\Bridge\Doctrine\Form\Type\EntityType;
use Symfony\Component\Form\Extension\Core\Type\TextType;
use Symfony\Component\Form\Extension\Core\Type\EmailType;

use App\Entity\Provincia;
class ContactoType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder
            ->add('nombre', TextType::class)
            ->add('telefono', TextType::class)
            ->add('email', EmailType::class, array('label' => 'Correo electrónico'))
            ->add('provincia', EntityType::class, array(
                'class' => Provincia::class,
                'choice_label' => 'nombre',))
            ->add('save', SubmitType::class, array('label' => 'Enviar'));
    }
}
```

![image-20220109174804617](/symfony-contactos-teoria/assets/image-20220109174804617.png)

**Automáticamente obtiene los datos de la provincia de la base de datos**



### 3.2.3 Modificación de datos

Lo que hemos hecho en el ejemplo anterior es una inserción de un nuevo contacto, pero... ¿cómo sería hacer una modificación de contacto existente?. El funcionamiento sería muy similar, pero con un pequeño cambio: la ruta del controlador recibirá como parámetro el código del contacto a modificar, y a partir de ahí, buscaríamos el contacto y lo cargaríamos en el formulario, incluyendo su `id`. De esta forma, al hacer `persist` se modificaría el contacto existente.

Podemos probarlo con este controlador:

```php
#[Route('/contacto/editar/{codigo}', name: 'editar', requirements:["codigo"=>"\d+"])]
public function editar(ManagerRegistry $doctrine, Request $request, int $codigo) {
    $repositorio = $doctrine->getRepository(Contacto::class);

    $contacto = $repositorio->find($codigo);
    if ($contacto){
        $formulario = $this->createForm(ContactoType::class, $contacto);

        $formulario->handleRequest($request);

        if ($formulario->isSubmitted() && $formulario->isValid()) {
            $contacto = $formulario->getData();
            $entityManager = $doctrine->getManager();
            $entityManager->persist($contacto);
            $entityManager->flush();
            return $this->redirectToRoute('ficha_contacto', ["codigo" => $contacto->getId()]);
        }
        return $this->render('nuevo.html.twig', array(
            'formulario' => $formulario->createView()
        ));
    }else{
        return $this->render('ficha_contacto.html.twig', [
            'contacto' => NULL
        ]);
    }
```

Ahora, si accedemos a [http://127.0.0.1:8080/contacto/editar/1](http://127.0.0.1:8080/contacto/editar/1), por ejemplo (suponiendo que tengamos un contacto con `id = 1` en la base de datos), se cargará el formulario con sus datos, y al enviarlo, se modificarán los campos que hayamos cambiado, y se cargará la página de inicio.

## 3.3 Validación de formularios

<iframe width="960" height="540" src="https://www.youtube.com/embed/qIHk_P2kE88" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Ahora que ya sabemos crear, enviar y gestionar formularios, veamos un último paso, que sería la validación de datos de dichos formularios previa a su envío. En el caso de Symfony, la validación no se aplica al formulario, sino a la entidad subyacente (es decir, a la clase `Contacto`, por ejemplo).

Por lo tanto, la validación la obtendremos añadiendo una serie de restricciones o comprobaciones a estas clases. Por ejemplo, para indicar que el `nombre`, `teléfono` y `e­-mail` del contacto no pueden estar vacíos, añadimos estas anotaciones en los atributos de la clase `Contacto`:

```php
<?php
namespace App\Entity;

use App\Repository\ContactoRepository;
use Doctrine\ORM\Mapping as ORM;
use Symfony\Component\Validator\Constraints as Assert;
use Symfony\Component\Validator\Constraints\Email;
#[ORM\Entity(repositoryClass: ContactoRepository::class)]
class Contacto
{
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column]
    private ?int $id = null;

    #[ORM\Column(length: 255)]
    #[Assert\NotBlank(message: 'El nombre es obligatorio')]
    private ?string $nombre = null;

    #[ORM\Column(length: 15)]
    #[Assert\NotBlank(message: 'El teléfono es obligatorio')]
    private ?string $telefono = null;

    #[ORM\Column(length: 255)]
    #[Assert\NotBlank(message: 'El correo es obligatorio')]
    #[Assert\Email(message: 'Correo no válido')]
    private ?string $email = null;

    #[ORM\ManyToOne(inversedBy: 'contactos')]
    #[Assert\NotBlank(message: 'La provincia es obligatoria')]
    private ?Provincia $provincia = null;

    public function getId(): ?int
    {
        return $this->id;
    }

    public function getNombre(): ?string
    {
        return $this->nombre;
    }

    public function setNombre(?string $nombre): self
    {
        $this->nombre = $nombre;

        return $this;
    }

    public function getTelefono(): ?string
    {
        return $this->telefono;
    }

    public function setTelefono(?string $telefono): self
    {
        $this->telefono = $telefono;

        return $this;
    }

    public function getEmail(): ?string
    {
        return $this->email;
    }

    public function setEmail(?string $email): self
    {
        $this->email = $email;

        return $this;
    }

    public function getProvincia(): ?Provincia
    {
        return $this->provincia;
    }

    public function setProvincia(?Provincia $provincia): self
    {
        $this->provincia = $provincia;

        return $this;
    }
}

```

Estas aserciones repercuten directamente sobre el código HTML del formulario, donde se añadirá el atributo `required` para que se validen los datos en el cliente. Para probarlo, hay que modificar el atributo `required` mediante Firebug.

Además, en todos los setters hemos de modificado el valor del atributo para que este sea nulo. Por ejemplo:

```php
<?php
public function setTelefono(?string $telefono): self
```

Si no hacemos el campo `nullable`  dará el siguiente error al intentar almacenar un dato vacío:

![image-20220117183231785](/symfony-contactos-teoria/assets/image-20220117183231785.png)

En el caso del e­mail, además, podemos especificar que queremos que sea un e­mail válido, lo que se consigue con esta otra anotación:

```php
<?php
/**
 * @ORM\Column(type="string", length=255)
 * @Assert\NotBlank()
 * @Assert\Email()
 */
private $email;
```

Estas funciones de validación admiten una serie de parámetros útiles. Uno de los más útiles es `message`, que se emplea para determinar el mensaje de error que mostrar al usuario en caso de que el dato no sea válido. Por ejemplo, para el e­mail, podemos especificar este mensaje de error:

```php
<?php
/**
 * @ORM\Column(type="string", length=255)
 * @Assert\NotBlank()
 * @Assert\Email(message="El email {{ value }} no es válido")
 */
private $email;
```

Y se disparará cuando no escribamos un e­mail válido e intentemos enviar el formulario:

![1549870910746](/symfony-contactos-teoria/assets/1549870910746.png)

>-info- Recordad que para probar que funciona la validación en el lado del servidor debéis cambiar con las herramientas de desarrollador del navegador el tipo de campo a `text`

Puedes consultar más información [aquí](https://symfony.com/doc/current/validation.html)

## 3.4 Otras consideraciones finales

<iframe width="960" height="540" src="https://www.youtube.com/embed/NDLkZJ6yr_A" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Para finalizar este apartado, veamos algunas cuestiones que hemos dejado en el tintero y no dejan de ser igualmente importantes.

### 3.4.1 Añadiendo estilo a los formularios

Los formularios que hemos generado en esta sesión son muy funcionales, pero poco vistosos, ya que carecen de estilos CSS propios. Si quisiéramos añadir CSS a estos formularios, tenemos varias opciones.

Una opción rudimentaria consiste en añadir clases (atributo class) a los controles del formulario para dar estilos personalizados. Después, en nuestro CSS bastaría con indicar el estilo para la clase en cuestión.

Pero Symfony ofrece diversos temas (`themes`) que podemos aplicar a los formularios (y webs en general) para darles una apariencia general tomada de algún framework conocido, como Bootstrap o Foundation. Si queremos optar por la apariencia de Bootstrap, debemos hacer lo siguiente:
Incluir la hoja de estilos CSS y el archivo Javascript de Bootstrap en nuestras plantillas. 

1. Una práctica habitual es hacerlo en la plantilla `base.html.twig` para que cualquiera que herede de ella adopte este estilo. Para ello, en la sección `stylesheets` debemos añadir el código [HTML](https://getbootstrap.com/docs/4.0/getting-started/introduction/#css) que hay en la documentación oficial de Bootstrap para incluir su hoja de estilo, y en la sección javascripts los [enlaces](https://getbootstrap.com/docs/4.0/getting-started/introduction/#js) a las diferentes librerías que se indican en la documentación de Bootstrap también.
   Al final tendremos algo como esto:

   ```twig
   <!DOCTYPE html>
   <html>
       <head>
           <meta charset="UTF-8">
           <title>{% block title %}Welcome!{% endblock %}</title>
           <link rel="icon" href="data:image/svg+xml,<svg xmlns=%22http://www.w3.org/2000/svg%22 viewBox=%220 0 128 128%22><text y=%221.2em%22 font-size=%2296%22>⚫️</text></svg>">
           {# Run `composer require symfony/webpack-encore-bundle` to start using Symfony UX #}
           {% block stylesheets %}
               <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.0.2/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-EVSTQN3/azprG1Anm3QDgpJLIm9Nao0Yz1ztcQTwFspd3yD65VohhpuuCOmLASjC" crossorigin="anonymous">
               <link href="{{ asset('css/estilos.css') }}" rel="stylesheet" />
           {% endblock %}
   
           {% block javascripts %}
               {{ encore_entry_script_tags('app') }}
               <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.0.2/dist/js/bootstrap.bundle.min.js" integrity="sha384-MrcW6ZMFYlzcLA8Nl+NtUVF0sA7MsXsP1UyJoMp4YLEuNSfAP+JcXn/tWtIaxVXM" crossorigin="anonymous"></script>
           {% endblock %}
       </head>
       <body>
           {% block body %}{% endblock %}
       </body>
   </html>
   
   ```

2. Editar el archivo de configuración `config/packages/twig.yaml` e indicar que los formularios usarán el tema de Bootstrap (en este caso, Bootstrap 5):

   ```yaml
   twig:
       # ....
       form_themes: ['bootstrap_5_horizontal_layout.html.twig']
   ```

Con estos dos cambios, la apariencia de nuestro formulario de contactos queda así:

![1549872641935](/symfony-contactos-teoria/assets/1549872641935.png)

Se tienen otras alternativas, como por ejemplo no indicar esta configuración general en `config/packages/twig.yaml` e indicar formulario por formulario el tema que va a tener:

```twig
{% form_theme formulario 'bootstrap_5_horizontal_layout.html.twig' %}
{{ form_start(formulario) }}
...
```

Existen también otros temas disponibles que utilizar. Podéis consultar más información [aquí](https://symfony.com/doc/current/form/form_customization.html#what-are-form-themes).

### 3.4.1 Añadir estilos para las validaciones

En el caso de las validaciones de datos del formulario, también podemos definir estilos para que los mensajes que de error que se muestran (parámetro `message` o similares en las anotaciones de la entidad) tengan un estilo determinado. Esto se consigue fácilmente eligiendo alguno de los temas predefinidos de Symfony. Por ejemplo, eligiendo Bootstrap, la apariencia de los errores de validación queda así automáticamente:

![1549872771901](/symfony-contactos-teoria/assets/1549872771901.png)

Estamos hablando de las validaciones en el servidor, ya que las que se efectúan desde el cliente por parte del propio HTML5 no tienen un estilo controlable desde Symfony.

Podríamos desactivar esta validación para que todo corra a cargo del servidor, si fuera el caso. Para ello, basta con añadir un atributo `novalidate` en el formulario al renderizarlo:

```twig
{{ form_start(formulario, {'attr': {'novalidate': 'novalidate'}}) }}
...
```

