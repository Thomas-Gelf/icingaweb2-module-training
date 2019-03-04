# Extending Icinga Web 2

## Write your own Icinga Web 2 module

Welcome! Glad that you are here to write your first Icinga Web module. Icinga Web makes getting started as easy as possible. Over the next few hours we will discover how fun the whole thing is with a series of practical examples.

## Should I really? Why?

Absolutely, why not? It's simple, and Icinga is 100% free Open Source software with a great community. Icinga Web 2 is a stable, easy to understand and future-proof platform. So exactly what you want to build your own projects.

## Only for monitoring?

Not at all! Sure, monitoring is where Icinga Web originates. There it has its strengths, there it is at home. Since monitoring systems communicate with all sorts of systems in and outside of our data center anyway, we found it obvious to do so in the same way in the frontend as well.

Icinga Web aims to be a modular framework that wants to make integration of third-party software as easy as possible. At the same time, true to the Open Source concept, we also want to make it easy for third parties to use Icinga logic as conveniently as possible in their own projects.

Whether it is about the mere linking of third-party systems, the connection of a CMDB or the visualization of complex systems as a supplement to conventional check-plugins - imagination knows no bounds.

## But I'm not a PHP/JavaScript/HTML5 hacker

No problem. Of course, it does not hurt to have in-depth knowledge of web development. Icinga Web allows you to write your own modules without in-depth PHP/HTML/CSS knowledge.

# Preparation

We use a Debian base installation to demonstrate how few dependencies Icinga Web 2 has. Although there are packages for all major distributions, we will be working directly with the GIT source tree for a better learning experience.

To wake up our notebooks from their weekend sleep, we'll give then a little task:

    apt-get update

    # A few usefull tools for the workshop:
    apt-get install git vim wget

    # Dependencies for Icinga Web 2:
    apt-get install php5-cli php5-mysql php5-gd php5-intl php5-ldap

    # Current source code from Icinga Web 2 master:
    cd /usr/local
    git clone https://git.icinga.org/icingaweb2.git

In the meantime, we will dedicate ourselves to the introduction!

## Overview of the training

* Installing Icinga Web 2
* Creating your own module
  * Own CLI commands
  * Working with parameters
  * Colors and other tricks
* Extending the web frontends
  * Own images
  * Own stylesheets
  * Extending the menu
  * Providing dashboards
* Working with data
  * Providing data
  * Pack code in libraries
  * Working with parameters
  * Tricks to work comfortably
* Configuration
* Translations
* Integration in third-party software
* Free lab

## Icinga Web 2 achitecture

During the development of Icinga Web 2, three priorities were emphasized:

* Simplicity
* Speed
* Reliability

Although we have dedicated ourselves to the DevOps movement, our target audience with Icinga Web 2 is clearly the operator, the admin. Therefore, we try to have as few dependencies as possible on external components. We do without one or the other hip feature, but then it isn't as broken when the hipsters in the next version want to do everything differently again. The best example is this workshop: now one year old, written long before the first stable release of Icinga Web 2 - and yet almost all exercises still work without any changes to the code.

The web interface is designed to hang easily on the wall for weeks and even months on the same screen. We want to be able to rely on what we see there corresponds to the current state of our environment. If there are problems, these are visualized, even if they are in the application itself. If the problem is resolved, everything must continue as usual. And without anyone having to plug in a keyboard and intervene manually.

## Libraries used

* Zend Framework 1.x
* jQuery 1.11 and 2.1
* Smaller PHP libraries
  * HTMLPurifier
  * DOMPdf
  * lessphp
  * Parsedown
* Smaller JS libraries
  * jquery-...

## Anatomy of an Icinga Web 2 module

Icinga Web 2 follows the paradigm "convention before configuration". After the experience with Icinga Web 1, we came to the conclusion that one of the best tools for XML processing is on each disk: `/bin/rm`. Those who stick to a few simple conventions will save a lot of configuration work. Basically, in Icinga Web you only have to configure paths for special cases. It is usually enough to just save a file in the right place.

An extensive, mature module could have approximately the following structure

    .
    └── training                Basic directory of the module
        ├── application
        │   ├── clicommands     CLI Commands
        │   ├── controllers     Web Controller
        │   ├── forms           Forms
        │   ├── locale          Translations
        │   └── views
        │       ├── helpers     View Helper
        │       └── scripts     View Scripts
        ├── configuration.php   Deploy menu, dashlets, permissions
        ├── doc                 Documentation
        ├── library
        │   └── Training        Library Code, Module Namespace
        ├── module.info         Module Metadata
        ├── public
        │   ├── css             Own CSS Code
        │   ├── img             Own Images
        │   └── js              Own JavaScript
        ├── run.php             Registration of hooks and more
        └── test
            └── php             PHP Unit Tests

We will work on this step by step during this training and fill it with life.

## Source Tree preparation

To get started, we need Icinga Web 2 first. This can be checked out of the GIT Source Tree and used directly on the spot. If you then set `DocumentRoot` for an appropriately configured web server in the public directory, you can start already. For testing purposes, it's even easier:

    cd /usr/local
    # If not done
    git clone https://git.icinga.org/icingaweb2.git
    ./icingaweb2/bin/icingacli web serve

Finished. To use the installation wizard, a token is required for security reasons. This ensures that there is never a time between installation and setup when an attacker could take over an environment. For packagers this point is completely optional, the same applies to those who roll out Icinga Web with a CM tool like Puppet: if there is a configuration on the system, you will never see the Wizard.

  http://localhost

## Complete installation with web server

So far we have been running Icinga Web 2 without an external web server. That would be performant enough even for most production environments, yet most of us feel more comfortable with a "real" web server. So if you have not already done so, stop the PHP process and clean up first:

```sh
rm -rf /tmp/FileCache_icingaweb/ /var/lib/php5/sess_*
```

These files, likely created by root, would otherwise only cause us problems. Then we install our web server:

```
apt-get install libapache2-mod-php5
./icingaweb2/bin/icingacli setup config webserver apache \
  > /etc/apache2/conf.d/icingaweb2.conf
service apache2 restart
```

You see, Icinga Web 2 can generate its own configuration for Apache (2.x, also compatible with 2.4). This does not only apply to Apache, but also to Nginx.

## The configuration directory

If not configured differently, Icinga Web 2 looks for its configuration in `/etc/icingaweb`. This can be overridden at any time with the environment variable ICINGAWEB_CONFIGDIR. We can also use this in the web server:

    SetEnv ICINGAWEB_CONFIGDIR /another/directory

## Manage multiple module paths

Especially those who always work with the latest version or want to switch between GIT branches safely, usually do not want to have to change files in their working copy. Therefore, it is recommended to use several module paths in parallel from the start. This can be done in the system settings or in the configuration under `/etc/icingaweb/config.ini`:

    [global]
    module_path= "/usr/local/icingaweb-modules:/usr/local/icingaweb2/modules"

## Icinga CLI setup 

When installing from packages you do not have to worry about anything, for our GIT working copy we create a symbolic link for the sake of convenience:

    ln -s /usr/local/icingaweb2/bin/icingacli /usr/local/bin/

## Installation from packages

The Icinga project builds up-to-date snapshots for a wide range of operating systems daily, the package sources are available at [packages.icinga.org](https://packages.icinga.org/). The current build status can be viewed at [build.icinga.org](https://build.icinga.org/jenkins/view/Icinga%20Web%202/), and the latest development at any time at [git.icinga.org](https://git.icinga.org/?p=icingaweb2.git) or [GitHub](https://github.com/Icinga/icingaweb2/).

But for our training we use the git repository directly. And I do not hate to do that in production either. Checksums for everything, changed files never go undetected, version changes happen in a fraction of a second - which package management can offer that? In addition, this procedure shows nicely how simple Icinga Web 2 actually is. We did not have to change a single file in the source directory for installation. Automake, configure? What for?! The configuration is elsewhere, and WHERE it is is communicated to the runtime environment.

# Create your own module

## Where should I start?

Probably the most important question is usually what you really want to do with his module. In our training we will first experiment with the given possibilities and then implement a small practical example.

## How should I name my module?

Once you know what the module is about to do, the hardest task is often choosing a good name. Ideally, this will tell you what the module actually does. But the name should not be too complicated, because we'll use it in PHP namespaces, directory names, and URLs.

Your own (company) name is often the first step for your own. Our favorite module name for our first steps in training today is `training`.

## Create and activate a new module

    mkdir -p /usr/local/icingaweb-modules/training
    icingacli module list installed
    icingacli module enable training

Finished!

# Extending Icinga CLI

The Icinga CLI was designed to provide as much of the application logic in Icinga Web 2 and its modules as possible on the commandline. The project aims to make the creation of cronjobs, plugins, useful tools and possibly small services as easy as possible.

## Configure Vim

We work with VIM in the training and create some initial settings:

    echo 'syntax on
    set expandtab
    set tabstop=4
    set shiftwidth=4' > /root/.vimrc

## Own CLI Commands

Structure of the CLI commands:

    icingacli <module> <command> <action>

Creating a CLI command is very easy. The directory `application/clicommands` will create a file whose name corresponds to the desired command:

    cd /usr/local/icingaweb-modules/training
    mkdir -p application/clicommands
    vim application/clicommands/HelloCommand.php

Here, Hello corresponds to the desired command with a capital letter. The ending Command MUST always be set.

Example command:

```php
<?php

namespace Icinga\Module\Training\Clicommands;

use Icinga\Cli\Command;

class HelloCommand extends Command
{
}
```

## Namespaces

* Namespaces help to separate modules from each other
* Each module gets a namespace, which results from the module name:

```
Icinga\Module\<Modulname>
```

* The first letter MUST be capitalized
* For CLI commands, a dedicated namespace Clicommands is available

## Inheritance

All CLI commands MUST inherit the Command class in the namespace `Icinga\Cli`. This brings us a whole series of advantages, which we will discuss later. It is important that our class name corresponds to the name of the file. In our HelloCommand.php this would be class HelloCommand.

## Command Actions

Each command can provide multiple actions. Any new public method that ends with action automatically becomes a CLI command action:

```php
<?php
class HelloCommand extends Command
{
    public function worldAction()
    {
        echo "Hello World!\n";
    }
}
```

## Task 1

We create a CLI Command with an action, which is executed as follows and generates the following output:

    icingacli training say hello

## Bash Autocompletion

The Icinga CLI provides autocomplete for all modules, commands and actions. If you install Icinga Web 2 from packages, everything is already in the right place, for our test environment, we manually set:

## Bash completion

    apt-get install bash-completion
    cp etc/bash_completion.d/icingacli /etc/bash_completion.d/
    . /etc/bash_completion

If the input is ambiguous as in `icingacli mo`, then an appropriate help is displayed.

## Inline Documentation for CLI Commands

Commands and their actions can be easily documented via inline comments. The comment text is immediately available on the CLI as a help.

```php
<?php

/**
 * This is where we say hello
 *
 * The hello command allows us to be friendly to everyone
 * and his dog. That's how nice people behave!
 */
class HelloCommand extends Command
{
    /**
     * Use this to greet the world
     *
     * Greeting every single person would take some time,
     * so let's greet the whole world at once!
     */
    public function worldAction()
    {
        // ...
```

A few example combinations of how the help can be displayed:

    icingacli training
    icingacli training hello
    icingacli help training hello
    icingacli training hello world --help

The help command can be at the beginning or used at any point as a parameter with `--`.

## Task 2

Create and test documentation for a `something` action for the `say` command in the `training` module!

## Command line parameters

Of course we can completely check, use and control command line parameters ourselves. Thanks to inheritance, the corresponding instance of `Icinga\Cli\Params` is already available in `$this->params`. The object has a `get()` method, to which we can give the desired parameter and optionally a default value. Without default value we get `null` if the corresponding parameter is not given.

```php
<?php

// ...

    /**
     * Say hello as someone
     *
     * Usage: icingacli training hello from --from <someone>
     */
    public function fromAction()
    {
        $from = $this->params->get('from', 'Nowhere');
        echo "Hello from $from!\n";
    }
```

### Example call

    icingacli training hello from --from Nürnberg
    icingacli training hello from --from "Netways Training"
    icingacli training hello from --help
    icingacli training hello from

## Standalone parameters

It is not absolutely necessary to assign an identifier to each parameter. If you want, you can simply string parameters together. Most conveniently, these are accessible via the `shift()` method:

```php
<?php

// ...

/**
 * Say hello as someone
 *
 * Usage: icingacli training hello from <someone>
 */
public function fromAction()
{
    $from = $this->params->shift();
    echo "Hello from $from!\n";
}
```

### Example call

    icingacli training hello from Nürnberg

## Shifting is fun

The `shift()` method behaves in the same way as you would expect from common programming languages. The first parameter of the list is returned and removed from the list. If you call `shift()` several times in succession, all existing standalone parameters are returned until none exists. With `unshift()` you can undo such an action at any time.

A special case is `shift()` with an identifier (key) as a parameter. So `shift('to')` would not only return the value of the `--to` parameter, but also remove it from the params object, regardless of its position. Again, it is possible to specify a default value:

```php
<?php
// ...
$person = $this->params->shift('from', 'Nobody');
```

Of course, this also works for standalone parameters. Since we have already used the first parameter of `shift()` with the optional identifier (key), but still want to set something for the second (default value), we simply set the identifier to null here:

```php
<?php
// ...
public function fromAction()
{
    $from = $this->params->shift(null, 'Nowhere');
    echo "Hello from $from!\n";
}
```

### Example call

    icingacli training hello from Nürnberg
    icingacli training hello from
    icingacli training hello from --help

## API documentation

The Params class in the `Icinga\Cli` namespace documents other methods and their parameters. These are most conveniently accessible in the API documentation. This can be generated with phpDocumentor, in the near future there should also be a CLI command.

## Task 3

Extend the say command to support all of the following options:

    icingacli training say hello World
    icingacli training say hello --to World
    icingacli training say hello World --from "Icinga CLI"
    icingacli training say hello World "Icinga CLI"

## Exceptions

Icinga Web 2 wants to promote clean PHP code. This includes, among other things, that all Warnings generate errors. Errors are thrown for error handling. We can just try it:

```php
<?php
// ...
use Icinga\Exception\ProgrammingError;
// ...
/**
 * This action will always fail
 */
public function kaputtAction()
{
    throw new ProgrammingError('No way');
}
```

### Call

    icingacli training hello kaputt
    icingacli training hello kaputt --trace

## Exit codes

As we can see, the CLI catches all exceptions and outputs pleasant readable error messages along with a colored indication of the error. The exit code in this case is always 1:

    echo $?

This allows reliable evaluation of failed jobs. Only the exit code 0 stands for successful execution. Of course, everyone is free to use additional exit codes. This is done in PHP using `exit($code)`. Example:

```php
<?php
echo "CRITICAL\n";
exit(2);
```

Alternatively, Icinga Web provides the `fail()` function in the Command class. It is an abbreviation for a colored "ERROR", a status output and `exit(1)`:

```php
<?php
$this->fail('An error occurred');
```

## Colors?

As we have just seen, the Icinga CLI can create colored output. The Screen class in the `Icinga\Cli` namespace provides useful help functions. We access it in our Command classes via `$this->screen`. This way the output can be colored:

```php
<?php
echo $this->screen->colorize("Hello from $from!\n", 'lightblue');
```

As an optional third parameter, the `colorize()` function can be given a background color. For the representation of the colors ANSI escape codes are used. If Icinga CLI detects that the output is NOT in a terminal/TTY, no colors will be output. This ensures that e.g. when redirecting the output to a file, no disturbing special characters appear.

> To recognize the terminal, PHP uses the POSIX extension. If this is not available, as a precaution no ANSI codes are used.

Other useful features in the Screen class are:

* `clear()` to clear the screen (used by `--watch`)
* `underline()` to underline text
* `newlines($count = 1)` to output one or more newlines
* `strlen()` to determine the character width without ANSI codes
* `center($text)` to output text centered depending on the screen width
* `getRows()` and `getColumns()` where possible, to determine the usable space
* `hasUtf8()` to query UTF8 support of the terminal

Attention: Of course, it does not work to find out that someone is traveling in a UTF8 terminal with an ISO8859 Putty.

### Task

Our `hello` action in the `say` command should output the text in color and centered both horizontally and vertically. We use `--watch` to flash the output alternately in at least two colors.

# Your own module in the web frontend

Icinga Web would not carry __"Web"__ in its name if its true qualities did not show up there as well. As we'll see shortly, __convention before configuration__ applies. Of course, according to the classic __MVC concept__, there are controllers with all available actions and matching view scripts for output and display.

We have deliberately omitted the library/model separation, and each additional layer eventually increases the complexity. You could also look at the library code in many modules as a "model", but that's what the specialists should do after us. Anyway, we would like to have as many nice modules as possible, ideally with a lot of reusable code, which then also benefits other modules.

## A first Controller

Every `Action` in a `Controller` automatically becomes a `route` in our web frontend. It looks something like this:

    http(s)://<host>/icingaweb/<module>/<controller>/<action>

If we want to create our "Hello World" again for our training module, we first create the basic directory for our controllers:

    mkdir -p training/application/controllers

Afterwards we add our controller. As you suspect, this must be called HelloController.php and be in the Controller namespace of our module:

```php
<?php

namespace Icinga\Module\Training\Controllers;

use Icinga\Web\Controller;

class HelloController extends Controller
{
    public function worldAction()
    {
    }
}
```

If we call the URL (training/application/controllers) now, we get an error message:

    Server error: script 'hello/world.phtml' not found in path
    (/usr/local/icingaweb-modules/training/application/views/scripts/)

Practically, it immediately tells us what we need to do next.

## Create a view script

The corresponding base directory is still missing. Since we create a view script in a dedicated file per "action", there is one directory per "controller":

    mkdir -p training/application/views/scripts/hello

The view script is then just like the "action", so world.phtml:
    
```php
<h1>Hello World!</h1>
```

That's it, our new URL is now available. We could now use the full scope for our module and style it accordingly. But we can also use a few predefined elements. Two important classes are e.g. `controls` and `content` for header elements and the page content.

```php
<div class="controls">
<h1>Hello World!</h1>
</div>

<div class="content">
Here you go...
</div>
```

This automatically gives even spacing to the page margins, and also gives the effect that when scrolling down, the `controls` stop while the `content` scrolls. Of course, we will not notice that until we fill our module with more content.

## Menu entries

Menu entries in Icinga Web 2 can be personalized and/or specified by the administrator (*). Regardless, they can be provided by modules. This is a global configuration that can be made in the base directory of your own module in `configuration.php`:

```php
<?php

$this->menuSection('Training')
     ->add('Hello World')
     ->setUrl('training/hello/world');
```

### Icons for menu entries

So that our menu item looks better, we miss on this occasion just one more icon:

```php
<?php

$this->menuSection('Training')
     ->setIcon('thumbs-up')
     ->add('Hello World')
     ->setUrl('training/hello/world');
```

To find out which icons are available, we activate the `doc` module under `System`/`Module`. Then we find the icon list under `Documentation`/`Developer - Style`. These are icons that have been embedded in a font. This has the great advantage that much fewer requests have to be made via the connection - the icons are simply "always there".

Alternatively, you can still use classic icons (.png etc) if you wish. This is especially useful if you want to use a special icon (for example, a company logo) for your module, which can hardly be integrated into the official Icinga Icon font:

```php
<?php

$this->menuSection('Training')->setIcon('img/icons/success.png');
```

## Adding images

If you would like to use your own images in your module, you can simply provide them under `public/img`:

    mkdir -p public/img
    wget https://www.icinga.org/wp-content/uploads/2014/06/tgelf.jpg
    mv tgelf.jpg public/img/

Our images are immediately accessible via the web, the URL pattern is as follows:

    http(s)://<icingaweb>/img/<module>/<image>

For our specific case http://localhost/img/training/tgelf.jpg. This can also be wonderfully used in our View script. Instead of creating an img tag (which of course would be possible) we use one of the many practical view helpers:

```php
...
<div class="content">
<?= $this->img('img/training/tgelf.jpg', array('title' => 'Thomas Gelf')) ?> Here you go...
</div>
```

## Task

Create the URLs `training/hello/thomas` and `training/say/hello` and add an additional menu item. Also, look for a nicer icon for our training module on the Internet and set it accordingly.

## Dashboards

Before we take care of serious issues we will of course still provide our useful URL as a default dashboard. This can also be done in the `configuration.php`:

```php
<?php
$this->dashboard('Training')->add('Hello', 'training/hello/world');
```

# We need data!

Of course, with our web routes working so well, we want to do something useful with them. An application can be so beautiful, without useful content, it quickly gets boring. In an MVC environment, the `controllers` usually use the `models` to get their data and feed the `view`.

## Fill our view with data

The controller provides access to our view in `$this->view`. In this way it can be populated quite comfortably:

```php
<?php

public function worldAction()
{
    $this->view->application = 'Icinga Web 2';
    $this->view->moreData = array(
        'Work'   => 'done',
        'Result' => 'fantastic'
    );
}
```

We are now expanding our view script and presenting the submitted data accordingly:

```php
<h3>Some data...</h3>

This example is provided by <a href="http://www.netways.de">Netways</a> 
and based on <?= $this->application ?>.

<table>
<?php foreach ($this->moreData as $key => $val): ?>
    <tr><th><?= $key ?></th><td><?= $val ?></td></tr>
<?php endforeach ?>
</table> 
```

## Task

Under `training/list/files` the contents of our module directory should be listed in table form.
* Note: with `$this->Module()->getBaseDir()` we get our module directory
* More about opening directories at [http://en.php.net/opendir](http://en.php.net/opendir) 

# But please with style!

Although it is not directly related to our topic, but one thing stands out: our table is not very nice. Luckily we can easily put CSS in our module. We create a suitable directory, the name should be obvious:

    mkdir public/css

We then add our CSS instructions there in the file `module.less`. Less is a CSS extension to all sorts of functions, more can be found under [...](). Conventional CSS is definitely valid here. The nice thing about Icinga Web is that I do not have to worry about whether my CSS influences other modules or Icinga Web itself: that's not the case.

So we can easily define the following, without "breaking" foreign tables:

    table {
        width: 100%;
    }

    th {
        width: 20%;
        text-align: right;
        line-height: 2em;
        padding-right: 2em;
    }

When we watch the requests in our browser's developer tools, we see that Icinga Web loads css/icings.min.css as the only CSS file. We can also load css/icinga.css to conveniently view what Icinga Web has done with our CSS code:

    .icinga-module.module-training table {
      width: 100%;
    }
    .icinga-module.module-training th {
      width: 20%;
      text-align: right;
      line-height: 2em;
      padding-right: 2em;
    }

As we can see, prefixes ensure that our CSS only applies to those containers in which our module represents its contents.

## Useful CSS classes

Icinga Web 2 provides a set of CSS classes that make our job easier. Thus `common-table` is useful for the usual lists in tables, `name-value-table` for name/value pairs where the identifier is displayed as th on the left and the corresponding value in a td on the right. Also useful is `table-row-selectable` - this changes the behavior of the table. The whole line is highlighted when you hover over it. If you click somewhere, the first link of the line comes forth. In combination with `common-table`, the whole thing looks just as good without additional work.

# Real data cleaned up

As we saw earlier, such a module becomes really interesting with real data. What we have done wrong, however, is that our controller gets the data itself. That is unattractive and would cause us problems at the latest if we want to use this data on the CLI as well.

## Our own library

We create a new directory for our library in our module, following the scheme `library/<module name>`. In our case:

    mkdir -p library/Training

As already learned, we use the namespace `Icinga\Modules\<module name>` for our module. All of the namespaces underneath will search Icinga Web 2 automatically in the newly created directory. Exceptions are those seen earlier, such as `Clicommands` or `Controllers`.

A library that does the work for this exercise might be in `File.php` and look like this:

```php
<?php

namespace Icinga\Module\Training;

use DirectoryIterator;

class Directory
{
    public static function listFiles($path)
    {
        $result = array();

        foreach (new DirectoryIterator($path) as $file) {
            if ($file->isDot()) continue;

            $result[] = (object) array(
                'name' => $file->getFilename(),
                'path' => $file->getPath(),
                'size' => $file->getSize(),
                'type' => $file->getType()
            );
        }
        return $result;
    }
}
```

Our controller can now easily retrieve the data via our small library:

```php
<?php

// ...
use Icinga\Module\Training\Directory;

class FileController extends Controller
{
    public function listAction()
    {
        $this->view->files = Directory::listFiles($this->Module()->getBaseDir());
    }
}
```

## Task

Put this or a comparable library in your module. Make a view script available, which can list the individual files. Importantly, use `$this->escape()` in the view script to escape data whose source is unsafe (for example, filenames).

# Parameter handling

So far we have not given any parameters to our URLs. But that's easy too. As on the command line, Icinga Web provides us with simple access to Params. Access is as usual:

```php
<?php
$file = $this->params->get('file');
```

`shift()` and cohorts are of course available as well.

## Task

Under `training/file/show?file=<filename>` additional information about the desired file should be displayed. Owners, permissions, last change and mime-type - but it is also quite simply enough to sort name and size beautifully.

## Related links

In our file list we now want to link from each file to the corresponding detail area. In order to avoid problems with parameter escaping, we use a new helper, `qlink`:

```php
<td><?= $this->qlink(
    $file->name,
    'training/file/show',
    array('file' => $file->name)
) ?></td>
```

The first parameter is the text to be displayed, the second the link to be created, and third optional parameters for this link. The fourth parameter could be an array with any other HTML attributes.

If we now click on a file in our list, we end up with the corresponding details. But that is also more convenient. Just try putting `data-base-target="_next"` in the content-div:

    <div class="content" data-base-target="_next">

We control the multi-column layout of Icinga Web for the first time without great effort!

# URL handling

Anyone who has observed how the browser behaves, may have noticed that not every click reloads the page. Icinga Web 2 intercepts all requests and sends them independently via XHR request. On the server side, this is detected, and then only the respective HTML snippet is sent in response. This usually only corresponds to the output created by the corresponding view script.

Nevertheless, each link remains a link and can be e.g. open in a new tab. Here again it is recognized that this is not an XHR request, the complete layout is delivered.

Usually, links always end up in the same container, but you can influence the behavior with `data-base-target`. The attribute closest to the clicked element wins. If you want to cancel `_next` for a section of the page, simply set `data-base-target="_self"`.

# Data handling made easy

Icinga Web offers a lot of nice tools. One thing we still want to examine, the so-called DataSources. We integrate the ArrayDatasource and add another function to our library code:

```php
<?php

use Icinga\Data\DataArray\ArrayDatasource;

// ...

    public static function selectFiles($path)
    {
        $ds = new ArrayDatasource(self::listFiles($path));
        return $ds->select();
    }
```

Then we also change our controller very easily:

```php
<?php
$query = Directory::selectFiles(
    $this->Module()->getBaseDir()
)->order('type')->order('name');

$this->view->files = $query->fetchAll();
```

## Task 1

Rebuild the list so that you can sort it up or down by mouse click.

## Additional task

```php
<?php

$editor = Widget::create('filterEditor')->handleRequest($this->getRequest());
$query->applyFilter($editor->getFilter());
```

## Autorefresh

As a monitoring interface, it goes without saying that Icinga Web provides a reliable and stable Autorefresh function. This can be conveniently controlled from the controllers:

```php
<?php

$this->setAutorefreshInterval(10);
```

## Task 2

Our file list should be automatically updated, the detail information as well. Show the modification time of a file (`$file->getMtime()`) and use the `timeSince` helper to represent the time. Change a file on the hard drive and see what happens. How can that be explained?

# Configuration

Those who developed a module would like to configure this probably too. Configuration for a module is stored under `/etc/icingaweb/modules/<modulename>`. What is found there in a `config.ini` is accessible in the controller as follows:

```php
<?php
$config = $this->Config();

/*
[section]
entry = "value"
*/
echo $config->get('section', 'entry');

// Returns "default" because "noentry" does not exist:
echo $config->get('section', 'noentry', 'default');

// Reads from the special.ini instead of the config.ini:
$config = $this->Config('special');
```

## Task

The base path for the list controller of our training module should be configurable. If no path is configured, we continue to use our module directory.

# Translations

For a detailed description of the translation options, we open the documentation for the `translation` module. Here are the individual steps:

```php
<h1><?= $this->translate('My files') ?></h1>
```

    apt-get install gettext poedit
    icingacli module enable translation
    icingacli translation refresh module training de_DE
    # Translate with Poedit
    icingacli translation compile module training de_DE

# Using Icinga Web logic in third party software?

With Icinga Web 2 we want to make the integration of third party software as easy as possible. We also want it easy for others to use Icinga Web logic in their software.

Basically, the following call in any PHP file is enough:

```php
<?php

require_once 'Icinga/Application/EmbeddedWeb.php';
Icinga\Application\EmbeddedWeb::start();
```

Finished! No authentication, no bootstrapping of the full web interface. But anything that exists as library code can be used.

## Task

Create an additional PHP file that embeds Icinga Web 2. Then use the directory handling library from your training module.

# Free Lab

You really arrived here? In a single day? Respect. The speaker now has guaranteed an exciting final exercise to finish the day with a practical example and many new tricks.

Have fun with Icinga Web 2 !!!

