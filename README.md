# PsyshBundle

[![Package version](https://img.shields.io/packagist/v/theofidry/psysh-bundle.svg?style=flat-square)](https://packagist.org/packages/theofidry/psysh-bundle)
[![Build Status](https://img.shields.io/github/actions/workflow/status/theofidry/PsyshBundle/tests.yaml?branch=master&style=flat-square)](https://github.com/theofidry/PsyshBundle/actions)
[![License](https://img.shields.io/badge/license-MIT-red.svg?style=flat-square)](LICENSE)

A bundle to use the PHP REPL [PsySH][1] with [Symfony][2]. Learn more at [psysh.org][1].

**Requirements:** PHP 8.1+, Symfony 6.4+

What does it do?
- Loads [PsySH][1] with the full application container
- Exposes the following variables out of the box:

| Variable       | Description                                      |
|----------------|--------------------------------------------------|
| `$container`   | The Symfony service container (TestContainer)    |
| `$kernel`      | The application kernel                           |
| `$parameters`  | All container parameters                         |
| `$self`        | The PsySH shell instance itself                  |

You can also [add your own variables](#adding-custom-variables) via configuration.


## Documentation

1. [Install](#install)
1. [Usage](#usage)
    1. [PsySH as a debugger](doc/debugger.md)
    1. [Reflect like a boss](doc/reflect.md)
    1. [PsySH for breakpoints](doc/breakpoint.md)
1. [Customize PsySH](#customize-psysh)
1. [Credits](#credits)


## Install

Install via [Composer](https://getcomposer.org/):

```bash
composer require --dev theofidry/psysh-bundle
```

With [Symfony Flex](https://github.com/symfony/flex), the bundle is registered automatically in `config/bundles.php`. If you manage bundles manually, add it only for `dev`/`test` environments:

```php
// config/bundles.php
return [
    // ...
    Fidry\PsyshBundle\PsyshBundle::class => ['dev' => true, 'test' => true],
];
```


## Usage

### Interactive shell

```bash
bin/console psysh
```

Once inside the shell, you have immediate access to `$container`, `$kernel`, `$parameters` and `$self`.

![PsySH Shell](doc/images/shell.png)

### Inline breakpoints

Place a `psysh()` call anywhere in your code to drop into an interactive shell at that point:

```php
use function Fidry\PsyshBundle\psysh;

class OrderService
{
    public function process(Order $order): void
    {
        // Drop into a shell with $order available and the current object bound
        psysh(['order' => $order], $this);
    }
}
```

[Go further with the docs](#documentation).


## Customize PsySH

### Adding a custom command

Tag any class extending `Psy\Command\Command` with `psysh.command`. With autoconfigure enabled (the default in Symfony 6.4), no explicit tag is needed — the bundle detects these classes automatically:

```yaml
# config/services.yaml
services:
    _defaults:
        autoconfigure: true
        autowire: true

    Acme\Shell\MyCommand: ~
```

To add the tag explicitly:

```yaml
services:
    Acme\Shell\MyCommand:
        tags:
            - { name: psysh.command }
```

> The bundle autoconfigures any service that inherits from `Psy\Command\Command` or `Psy\Command\ReflectingCommand`.

### Adding custom variables

Declare extra shell variables in `config/packages/dev/psysh.yaml`:

```yaml
# config/packages/dev/psysh.yaml
psysh:
    variables:
        foo: bar
        router: "@router"
        some: [thing, else]
        debug: "%kernel.debug%"
```

Variables can be:
- scalar values
- container parameter references (e.g. `%kernel.debug%`)
- service references (prefixed with `@`, e.g. `"@router"`)
- arrays

After running `bin/console psysh`, inspect available variables with `ls`:

```
>>> ls
Variables: $container, $kernel, $parameters, $self, $foo, $router, $some, $debug
```


## Credits

This bundle is developed by [Théo FIDRY](https://github.com/theofidry). This project has been made possible thanks to:

- [Justin Hileman](https://github.com/bobthecow): author of [PsySH][1] and [all the contributors](https://github.com/bobthecow/psysh/graphs/contributors)
- [Adrian Palmer](https://github.com/navitronic): gave the lead for porting [PsySH][1] to [Symfony][2]


[1]: https://psysh.org/
[2]: https://symfony.com/
