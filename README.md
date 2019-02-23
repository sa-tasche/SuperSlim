# Nyholm's SuperSlim framework

The quickest and best framework you can start building on. 

## The idea

The idea of this framework is to give you a good foundation to start building your
application. **No file is sacred** in this framework. It is *you* as a developer
that are responsible for code and the dependencies you use. 

If you eventually outgrow this framework, just replace `Kernel.php` with Symfony's
Kernel and you are running a Symfony 4 application.  

## The architecture

The framework is quite simple, it consists of less than 5 classes. It follows the
Middleware pattern and supports Symfony's HTTP Foundation. 

### index.php

Frontend controller, its job is to create a Request and give it to the `Kernel`.

### Kernel

It creates a container from cache or from config then starts the `Runner`.

### Runner

Runs the chain of middleware one by one in the order they are defined in the service
declaration of `App\Runner` in `services.yaml`. The last middleware should be the 
router that calls one of your controller. The router will return a Response. 

When the router middleware as returned a response the middleware will run again but
backwards. 

### Router

The frameworks ships with two routers: `App\Middleware\Router`  and `App\Middleware\RouterForComplexRoutes`.
The former is using just simple if-statements to match the route with a controller.
This is by far the quickest way if you only got a few routes. If you have more complex
routing or a great number of them, you might be better of with `RouterForComplexRoutes`. 
It uses the [Symfony 4 router](https://symfony.com/doc/current/components/routing.html) 
which is the fastest generic router written in PHP. 

Please make sure you profile your application to see which routes that fits you better. 

### Controller

Here is your normal PHP classes and your normal code. Your controller should always
return a Response. 

### Services

You are free to create how many services, value objects, database entities as you
want. You can use `config/services.yaml` to register your services. By default they
are autowired with [Symfony Dependency Injection](https://symfony.com/doc/current/components/dependency_injection.html)
container.

## Configuration

It is simple to configure the application. You can use environment variables or
the `.env` files for host specific configuration. If you want to register services
or modify behavior then check the `config/` folder. 

If you know your way around Symfony configuration then you wont have any problem 
configure SuperSlim. 

### Database

Your application may want to use a database. Just pick your favorite way to connect
to your database and register it as a service. Here is an example using [Doctrine](https://www.doctrine-project.org).

```bash
composer require doctrine/orm
```

```yaml
# config/packages/doctrine.yaml

services:
  doctrine.config:
    class: Doctrine\ORM\Configuration
    factory: Doctrine\ORM\Tools\Setup::createAnnotationMetadataConfiguration
    arguments:
      - ['%kernel.project_dir%/src']
      - 'kernel.debug'
      - null
      - null
      - false

  Doctrine\ORM\EntityManager:
    factory: Doctrine\ORM\EntityManager::create
    public: true
    arguments:
      - { driver: pdo_mysql, url: '%env(resolve:DATABASE_URL)%' }
      - '@doctrine.config'

```

```
# .env.local

DATABASE_URL=mysql://db_user:db_password@127.0.0.1:3306/db_name

```

```php

namespace App\Entity;

use Doctrine\ORM\Mapping as ORM;

/**
 * @ORM\Entity()
 * @ORM\Table(name="products")
 **/
class Product
{
    /**
     * @ORM\Id()
     * @ORM\Column(type="integer")
     * @ORM\GeneratedValue()
     */
    protected $id;

    /**
     * @ORM\Column(type="string")
     */
    protected $name;

    public function getId()
    {
        return $this->id;
    }

    public function getName()
    {
        return $this->name;
    }

    public function setName($name)
    {
        $this->name = $name;
    }
}
```

If you want to enable CLI support: 
```php
// cli-config.php

use App\Kernel;
use Doctrine\ORM\Tools\Console\ConsoleRunner;

require __DIR__.'/config/bootstrap.php';

$kernel = new Kernel($_SERVER['APP_ENV'], (bool) $_SERVER['APP_DEBUG']);
$kernel->boot();

return ConsoleRunner::createHelperSet($kernel->getContainer()->get(\Doctrine\ORM\EntityManagerInterface::class));

```

```bash
vendor/bin/doctrine orm:schema-tool:update --force --dump-sql
```

### Templating

Returning `new Response('Hello world');` is not very fun. You probably want to use
some templating. Pick you favorite tool and just register it as a service. Here is
an example using [Twig](https://twig.symfony.com/).

```bash
composer require twig/twig
```

```yaml
# config/packages/twig.yaml

services:
  Twig\Loader\FilesystemLoader:
    arguments: ['%kernel.project_dir%/templates']

  Twig\Environment:
    arguments:
      - '@Twig\Loader\FilesystemLoader'
      - { cache: '%kernel.cache_dir%/templates' }

```

```php
namespace App\Controller;

use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Twig\Environment;

class MyTwigController
{
    private $twig;

    public function __construct(Environment $twig)
    {
        $this->twig = $twig;
    }

    public function index()
    {
        return new Response($this->twig->render('index.html.twig', ['name' => 'Foobar']));
    }
}
```

```twig
{# templates/index.html.twig #}

Hello {{ name }}!

```

## The future of this framework

I will treat this as a hobby project. If you like it, give it a star and fork it
to turn it into something you like. 

Or, you could read these great articles and [build your own framework](https://symfony.com/doc/current/create_framework/index.html).