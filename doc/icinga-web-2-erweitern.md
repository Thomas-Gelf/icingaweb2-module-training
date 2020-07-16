# Icinga Web 2 erweitern

## Eigene Icinga Web 2 Module schreiben

Herzlich willkommen! Schön dass du hier bist, um deine ersten eigenen Icinga Web Module zu schreiben. Icinga Web macht uns den Einstieg möglichst einfach. In den nächsten Stunden werden wir anhand einer Reihe praktischer Beispiele entdecken, wie angenehm das Ganze ist.

## Soll ich wirklich? Warum?

Unbedingt, warum nicht? Es ist herrlich einfach, und Icinga ist 100% freie Open Source Software mit einer großartigen Community. Icinga Web 2 stellt eine stabile, einfach verständliche und zukunftssichere Plattform dar. Also eigentlich genau das, worauf man eigene Projekte aufbauen möchte.

## Nur für's Monitoring?

Überhaupt nicht! Klar, Monitoring ist da wo Icinga Web herkommt. Dort hat es seine Stärken, dort ist es zu Hause. Nachdem Monitoring-Systeme ohnehin mit allen möglichen Systemen in- und außerhalb des eigenen Rechenzentrums kommunizieren fanden wir es naheliegend, dies jetzt auch im Frontend auf ähnliche Weise zu tun.

Icinga Web will ein modulares Framework sein, welches die Integration von Drittsoftware möglichst einfach gestalten will. Gleichzeitig wollen wir es, getreu dem Open Source Gedanken, auch Dritten einfach machen, Logik von Icinga möglichst bequem in deren eigenen Projekten zu nutzen.

Ob es jetzt um die reine Verlinkung von Drittsystemen, die Anbindung einer CMDB oder die Visualisierung von komplexen Systemen als Ergänzung zu herkömmlichen Check-Plugins geht - der Fantasie sind hier keine Grenzen gesetzt.

## Ich bin aber kein PHP/JavaScript/HTML5 Hacker

Kein Problem. Freilich schadet es nicht, über fundierte Kenntnisse der Webentwicklung zu verfügen. Icinga Web erlaubt es aber, auch ohne tiefgehende PHP/HTML/CSS-Kenntnisse eigene Module schreiben zu können.

# Vorbereitung

Zum warm werden geben wir unseren Noteboooks schon mal eine kleine Aufgabe und installieren Icinga Web 2:

    https://icinga.com/docs/icingaweb2/latest/doc/02-Installation/

Bevor es dann so richtig los geht widmen wir uns erstmal der Einführung!

## Aufbau des Trainings

* Über Icinga Web 2
* Erstellung eines eigenen Moduls
  * Eigene CLI-Commands
  * Arbeiten mit Parametern
  * Farben und andere Gimmicks
* Erweiterung des Web-Frontends
  * Eigene Bilder
  * Eigene Stylesheets
  * Erweiterung des Menüs
  * Bereitstellung von Dashboards
* Das Arbeiten mit Daten
  * Bereitstellen von Daten
  * Code in Bibliotheken packen
  * Arbeiten mit Parametern
  * Tricks zum bequemen Arbeiten
* Konfiguration
* Übersetzungen
* Integration in Dritt-Software
* Abschließende Anmerkungen


## Icinga Web 2 Architektur

Bei der Entwicklung von Icinga Web 2 wurde auf drei Schwerpunkte Wert gelegt:

* Einfachheit
* Geschwindigkeit
* Zuverlässigkeit

Wir haben uns zwar der DevOps-Bewegung verschrieben, unsere Zielgruppe ist mit Icinga Web 2 aber ganz klar der Operator - der Admin. Wir versuchen darum möglichst wenig Abhängigkeiten von externen Komponenten zu haben. Wir verzichten deshalb auf das ein oder andere hippe Feature, dafür geht dann aber auch weniger kaputt, wenn man auf die neuesten Versionen updaten möchte.

Die Web-Oberfläche wurde entworfen, um problemlos Wochen- und Monatelang auf demselben Bildschirm an der Wand hängen zu können. Wir wollen uns darauf verlassen können, dass was wir dort sehen, dem aktuellen Stand unserer Umgebung entspricht. Gibt es Probleme, werden diese visualiert - auch wenn sie in der Anwendung selbst liegen. Wird das Problem behoben, muss alles weiterlaufen wie gehabt. Und das ohne, dass jemand eine Tastatur anstöpseln und manuell eingreifen muss.


## Benutzte Bibliotheken

* Zend Framework 1.x
* jQuery 3
* Kleinere PHP-Libraries
  * HTMLPurifier
  * DOMPdf
  * lessphp
  * Parsedown
* Kleinere JS-Bibliotheken
  * jquery-...

## Anatomie eines Icinga Web 2 Moduls

Icinga Web 2 folgt dem Paradigma "Konvention vor Konfiguration". Nach den Erfahrungen mit Icinga Web 1 kamen wir zu dem Ergebnis, dass eines der besten Tools zur XML-Verarbeitung auf jeder Platte liegt: `/bin/rm`. Wer sich an ein paar einfache Konventionen hält, spart sich eine Menge Konfigurationsarbeit. Grundsätzlich gilt, dass man in Icinga Web nur für ganz spezielle Fälle Pfade konfigurieren muss. Meist reicht es, eine Datei einfach an die richtige Stelle zu speichern.

Ein umfangreiches, erwachsenes Modul könnte in etwa folgende Struktur aufweisen:

    .
    └── training                Basis-Verzeichnis des Moduls
        ├── application
        │   ├── clicommands     CLI Befehle
        │   ├── controllers     Web Controller
        │   ├── forms           Formulare
        │   ├── locale          Übersetzungen
        │   └── views
        │       ├── helpers     View Helper
        │       └── scripts     View Skripte
        ├── configuration.php   Bereitstellen von Menü, Dashlets, Berechtigungen
        ├── doc                 Dokumentation
        ├── library
        │   └── Training        Library-Code, Modul-Namensraum
        ├── module.info         Metadaten zum Modul
        ├── public
        │   ├── css             Eigener CSS-Code
        │   ├── img             Eigene Bilder
        │   └── js              Eigenes JavaScript
        ├── run.php             Registrierung von Hooks und mehr
        └── test
            └── php             PHP Unit-Tests

Wir werden uns eine solche im Rahmen dieses Trainings Schritt für Schritt erarbeiten und mit Leben befüllen.

## Source Tree vorbereiten

Um loslegen zu können benötigen wir zuallererst Icinga Web 2. Dieses lässt sich aus dem GIT Source Tree auschecken und direkt an Ort und Stelle benutzen. Setzt man anschließend `DocumentRoot` eines entsprechend konfigurierten Webservers in das `public` Verzeichnis, kann man auch schon loslegen. Zu Testzwecken geht es aber auch noch einfacher:

    cd /usr/local
    # Wenn noch nicht erledigt:
    git clone https://github.com/Icinga/icingaweb2.git
    ./icingaweb2/bin/icingacli web serve

Fertig. Um den Installationswizard benutzen zu dürfen, ist aus Sicherheitsgründen ein Token erforderlich. Man wird von der Weboberfläche dazu aufgefordert, ein auf der CLI generiertes Token einzugeben. Damit stellen wir sicher, dass es zwischen Installation und Einrichtung, nie einen Zeitpunkt gibt, zu welchem ein Angreifer eine Umgebung übernehmen könnte. Für Packager ist dieser Punkt vollkommen opional, selbiges gilt für jene, die Icinga Web mit einem CM-Tool wie Puppet ausrollen: liegt eine Konfiguration auf dem System, so bekommt man den Wizard nie zu Gesicht.

```
  http://localhost
```

## Verwalten mehrerer Modul-Pfade

Gerade wer immer mit dem aktuellsten Versionsstand arbeiten oder gefahrlos zwischen GIT-Branches hin- und herwechseln möchte, der will für gewöhnlich ungern Dateien in seiner Arbeitskopie ändern müssen. Darum empfiehlt es sich, von Beginn an parallel mehrere Modul-Pfade zu benutzen. Dies kann in den System-Einstellungen oder in der Konfiguration unter `/etc/icingaweb2/config.ini` vorgenommen werden:

    [global]
    module_path= "/usr/local/icingaweb-modules:/usr/local/icingaweb2/modules"

## Installation aus Paketen

Das Icinga Projekt baut täglich aktuelle Snapshots für verschiedenste Betriebssysteme, die Paketquellen gibt es unter [packages.icinga.org](https://packages.icinga.org/). Die aktuelle Entwicklung lässt sich jederzeit auf [GitHub](https://github.com/Icinga/icingaweb2/) verfolgen.

Für unser Training nutzen wir aber direkt das Git-Repository. Und auch in Produktion mache ich das nicht ungern. Checksummen für alles, geänderte Dateien bleiben nie unentdeckt, Versionswechsel passieren in einem Sekundenbruchteil - welches Paketmanagement kann das schon bieten? Außerdem zeigt dieses Vorgehen schön, wie simpel Icinga Web 2 eigentlich ist. Wir mussten zur Installation keine einzige Datei im Quellverzeichnis ändern. Automake, configure? Wozu denn?! Die Konfiguration liegt woanders, und WO sie liegt wird der Laufzeitumgebung mitgeteilt.

# Ein eigenes Modul erstellen

## Womit soll ich anfangen?

Die vermutlich wichtigste Frage ist meist, was man mit seinem Modul eigentlich anstellen möchte. In unserem Training werden wir erst mit den gegebenen Möglichkeiten experimentieren und anschließend ein kleines Praxis-Beispiel umsetzen.

## Wie soll ich mein Modul nennen?

Sobald man weiß, was das Modul in etwa machen soll ist die schwierigste Aufgabe häufig die Wahl eines guten Namens. Im Idealfall geht daraus schon hervor, was das Modul eigentlich macht. Zu kompliziert soll der Name aber auch nicht sein, schließlich werden wir ihn in PHP-Namespaces, Verzeichnisnamen und in URLs verwenden.

Für erste eigene Gehversuche bietet sich häufig der eigene (Unternehmens-)Name an. Unser favorisierte Modulename für unsere ersten Gehversuche in der Schulung heute ist `training`.

## Erstellen und Aktivieren eines neuen Modules

    mkdir -p /usr/local/icingaweb-modules/training
    icingacli module list installed
    icingacli module enable training

Fertig!

# Erweitern der Icinga CLI

Die Icinga CLI wurde entworfen, um möglichst alles von dem was an Applikationslogik in Icinga Web 2 und dessen Modulen zur Verfügung steht, auch auf der Kommandozeile bereitzustellen. Damit will das Projekt die Erstellung von Cronjobs, Plugins, nützlichen Tools und eventuell kleinen Diensten möglichst einfach gestalten.

## Eigene CLI-Commands

Struktur der CLI Befehle:

    icingacli <modul> <command> <action>


Das Erstellen eines CLI-Kommandos ist denkbar einfach. Im Verzeichnis `application/clicommands` wird eine Datei erstellt, deren Name dem gewünschten Befehl entspricht:

    cd /usr/local/icingaweb-modules/training
    mkdir -p application/clicommands
    vim application/clicommands/HelloCommand.php

Hierbei entspricht `Hello` dem gewünschten Kommando mit großem Anfangsbuchstaben. Die Endung `Command` muss IMMER drangesetzt werden.

Beispiel-Command:

```php
<?php

namespace Icinga\Module\Training\Clicommands;

use Icinga\Cli\Command;

class HelloCommand extends Command
{
}
```

## Namespaces

* Namespaces helfen, um Module sauber gegeneinander abzugrenzen
* Jedes Modul erhält einene Namespace, der sich aus dem Module-Namen ergibt:

```
Icinga\Module\<Modulname>
```

* Der Anfangsbuchstabe MUSS hierbei jeweils groß geschrieben werden
* Für CLI-Befehle steht ein dedizierter Namensraum Clicommands zur Verfügung

## Vererbung

Sämtliche CLI-Commands MÜSSEN die Command-Klasse im Namesraum `Icinga\Cli` beerben. Dies bringt uns eine ganze Reihe von Vorteilen, auf die wir später noch eingehen werden. Wichtig ist, dass unser Klassenname dem Namen der Datei entspricht. In unsererer `HelloCommand.php` wäre dies also die Klasse `HelloCommand`.

## Command-Actions

Jedes Command kann mehrere Actions bereitstellen. Jede neue öffentliche Methode, welche mit `Action` endet wird hierbei automatisch zu einer CLI command action:

```php
<?php

namespace Icinga\Module\Training\Clicommands;

use Icinga\Cli\Command;

class HelloCommand extends Command
{
    public function worldAction()
    {
        echo "Hello World!\n";
    }
}
```

## Aufgabe 1

Wir erstellen ein CLI Command mit einer Action, welche folgendermaßen bedient wird und nachstehenden Output generiert:

    icingacli training hello world


## Bash Autocompletion

Die Icinga CLI stellt Autovervollständigung für alle Module, Kommandos und Aktionen bereit. Installiert man Icinga Web 2 per Paket ist alles schon an der richtigen Stelle, für unsere Test-Umgebung legen wir manuell Hand an:

## Bash completion

    apt-get install bash-completion
    
    cp /etc/bash_completion.d/icingacli /etc/bash_completion.d/ . /etc/bash_completion

Ist die Eingabe mehrdeutig wie bei `icingacli mo`, dann wird eine entsprechende Hilfe angezeigt.


## Inline-Dokumentation für CLI-Befehle

Befehle und deren Aktionen können einfach über Inline-Kommentare dokumentiert werden. Der Kommentar-Text steht sofort auf der CLI als Hilfe zur Verfügung.

```php
<?php

/**
 * This is where we say hello
 *
 * The hello command allows us to be friendly to everyone
 * and their dog. That's how nice people behave!
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

Ein paar Beispiel-Kombinationen, wie die Hilfe angezeigt werden kann:

    icingacli training
    icingacli training hello
    icingacli help training hello
    icingacli training hello world --help

Das Help-Kommando kann zu Beginn stehen oder an jeder beliebigen Stelle als Parameter mit `--` genutzt werden.

## Aufgabe 2

Erstelle und teste Dokumentation für eine `something` Aktion für den `say` Befehl im `training` Modul!


## Kommandozeilenparameter

Wir können Kommandozeilen-Parameter natürlich vollumfänglich selbst kontrollieren, nutzen und steuern. Dank Vererbung steht die entsprechende Instanz von `Icinga\Cli\Params` schon in `$this->params` bereit. Das Objekt verfügt über eine `get()`-Methode, welcher wir den gewünschten Parameter und optional einen Default-Wert mitgeben können. Ohne Default-Wert erhalten wir `null`, falls der entsprechende Parameter nicht mitgegeben wird.

```php
<?php

namespace Icinga\Module\Training\Clicommands;

use Icinga\Cli\Command;

class SayCommand extends Command
{
    /**
     * Say hello as someone
     *
     * Usage: icingacli training say from --from <someone>
     */
    public function fromAction()
    {
        $from = $this->params->get('from', 'Nowhere');
        echo "Hello from $from!\n";
    }
}
```

### Beispiel-Aufruf:

    icingacli training hello from --from Nürnberg
    icingacli training hello from --from "Icinga Training"
    icingacli training hello from --help
    icingacli training hello from

## Standalone-Parameter

Es ist nicht zwingend erforderlich, jedem Parameter einen Bezeichner zuzuordnen. Wer möchte, kann auch einfach Parameter aneinanderreihen. Am bequemsten sind diese über die `shift()`-Methode zugänglich:

```php
<?php

namespace Icinga\Module\Training\Clicommands;

use Icinga\Cli\Command;

class SayCommand extends Command
{

    /**
     * Say hello as someone
     *
     * Usage: icingacli training say from <someone>
     */

    public function fromAction()
    {
        $from = $this->params->shift();
        echo "Hello from $from!\n";
    }
}
```

### Beispiel-Aufruf

    icingacli training say from Nürnberg

## Shiften macht Freude

Die Methode `shift()` verhält sich so, wie man das von gängigen Programmiersprachen gewohnt ist. Der erste Parameter der Liste wird zurückgeliefert und von der Liste entfernt. Ruft man `shift()` mehrmals hintereinander auf, werden alle vorhandenen Standalone-Parameter zurückgeliefert, bis keiner mehr vorhanden ist. Mit `unshift()` kann man so eine Aktion jederzeit wieder rückgängig machen.

Ein Spezialfall ist `shift()` mit einem Bezeichner (key) als Parameter. So würde `shift('to')` nicht nur den Wert des Parameters `--to` zurückliefern, sondern diesen auch unabhängig von seiner Position aus dem Params-Objekt entfernen. Auch hier ist es möglich, einen Standardwert mitzugeben:

```php
<?php
// ...
$from = $this->params->shift('from', 'Nobody');
```

Das geht natürlich auch für Standalone-Parameter. Da wir durch den optionalen Bezeichner (key) den ersten Parameter von `shift()` schon belegt haben, jetzt aber für den zweiten (Standardwert) dennoch etwas setzen möchten, setzen wir den Bezeichner hier einfach auf null:

```php
<?php
// ...
public function fromAction()
{
    $from = $this->params->shift(null, 'Nowhere');
    echo "Hello from $from!\n";
}
```

### Beispiel-Aufruf

    icingacli training hello from Nürnberg
    icingacli training hello from
    icingacli training hello from --help

## API-Dokumentation

Die Params-Klasse im `Icinga\Cli` namespace dokumentiert noch weitere Methoden und deren Parameter. Diese sind am bequemsten in der API-Dokumentation zugänglich. Diese lässt sich mit phpDocumentor generieren.

## Aufgabe 3

Erweitere den `say` Befehl, um alle folgenden Varianten zu unterstützen:

    icingacli training say hello World
    icingacli training say hello --to World
    icingacli training say hello World --from "Icinga CLI"
    icingacli training say hello World "Icinga CLI"


## Exceptions

Icinga Web 2 will sauberen PHP-Code fördern. Dazu gehört, nebst anderem, dass sämtliche Warnungen Fehler generieren. Zum Error-Handling werden Fehler geworfen. Wir können das einfach ausprobieren:

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

### Aufruf

    icingacli training hello kaputt
    icingacli training hello kaputt --trace

## Exit-Codes

Wie wir sehen, fängt die CLI sämtliche Exceptions und gibt angenehm lesbare Fehlermeldungen, nebst farbigem Hinweis auf den Fehler, aus. Der Exit-Code ist in diesem Fall immer 1:

    echo $?

Damit lassen sich fehlgeschlagene Jobs zuverlässig auswerten. Nur der Exit-Code 0 steht für erfolgreiche Ausführung. Natürlich steht es jedem frei, zusätzlich weitere Exit-Codes zu benutzen. Dies erledigt man in PHP mittels `exit($code)`. Beispiel:

```php
<?php
echo "CRITICAL\n";
exit(2);
```

Alternativ stellt Icinga Web in der Command-Klasse die `fail()`-Funktion bereit. Sie ist eine Abkürzung für ein farbiges "ERROR", eine Status-Ausgabe und `exit(1)`:

```php
<?php
$this->fail('Ein Fehler ist passiert');
```

## Farben?

Wie wir eben gesehen haben, kann die Icinga CLI farbigen Output erstellen. Über die Screen-Klasse im `Icinga\Cli` Namensraum stehen nützliche Hilfsfunktionen hierzu bereit. Wir greifen in unseren Command-Klassen über `$this->screen` darauf zu. So lässt sich die Ausgabe farbig gestalten:

```php
<?php
echo $this->screen->colorize("Hello from $from!\n", 'lightblue');
```

Als optionalen dritten Parameter kann man der `colorize()`-Funktion eine Hintergrundfarbe mitgeben. Für die Darstellung der Farben werden ANSI escape codes benutzt. Erkennt Icinga CLI, dass die Ausgabe NICHT auf in ein Terminal/TTY erfolgt, werden keine Farben ausgegeben. Damit wird sichergestellt, dass z.B. beim Umleiten der Ausgabe in eine Datei keine störenden Sonderzeichen aufscheinen.

> Um das Terminal zu erkennen, wird die POSIX-Erweiterung von PHP genutzt. Ist diese nicht vorhanden, werden ebenfalls vorsichtshalber keine ANSI-Codes verwendet.

Weiter nützliche Funktionen in der Screen-Klasse sind:

* `clear()` um den Bildschirm zu löschen (wird von `--watch` benutzt)
* `underline()` um Text zu unterstreichen
* `newlines($count = 1)` um einen oder mehrere Zeilenumbrüche auszugeben
* `strlen()` um die Zeichenbreite ohne ANSI-Codes zu ermitteln
* `center($text)` um Text abhängig von der Bildschirmbreite zentriert auszugeben
* `getRows()` und `getColumns()` um wo möglich den verwendbaren Platz zu ermitteln
* `hasUtf8()` um UTF8-Unterstützung des Terminals abzufragen

Vorsicht: klappt natürlich nicht um herauszufinden, dass jemand in einem UTF8-Terminal mit einem ISO8859 Putty unterwegs ist. 

### Aufgabe

Unsere `hello`-Aktion im `say`-Befehl soll den Text in Farbe und sowohl horizontal als auch vertikal zentriert ausgeben. Wir nutzen `--watch`, um die Ausgabe abwechselnd in mindestens zwei Farben blinken zu lassen.

# Das eigene Modul im Web-Frontend

Icinga Web würde aber nicht __"Web"__ im Namen tragen, wenn seine wahren Qualitäten nicht auch dort zum Vorschein kämen. Wie wir gleich sehen werden gilt auch hier __Konvention vor Konfiguration__. Nach dem klassischen __MVC-Konzept__ gibt es natürlich Controller mit allen verfügbaren Aktionen und passende View-Skripte für die Ausgabe und Darstellung.

Wir haben bewusst auf die Trennung Library/Model verzichtet, jede zusätzliche Schicht erhöht schließlich auch die Komplexität. Man könnte auch den Library-Code in vielen Modulen als "Model" betrachten, das sollen aber die Spezialisten nach uns klären. Wir wünschen uns jedenfalls möglichst viele schicke Module, idealerweise mit viel wiederverwendbarem Code, der dann auch anderen Modulen zu Gute kommt.

## Ein erster Controller

Jede `Action` in einem `Controller` wird automatisch zu einer `Route` in unserem Web-Frontend. Das sieht in etwa wie folgt aus:

    http(s)://<host>/icingaweb2/<modul>/<controller>/<action>

Wenn wir für unser Training-Modul jetzt wieder unser "Hello World" erstellen möchten, erstellen wir erst mal das Basis-Verzeichnis für unsere Controller:

    mkdir -p training/application/controllers

Anschließend legen wir unseren Controller an. Wie du schon richtig vermutest, muss dieser HelloController.php heißen und im Controllers-Namespace unseres Moduls liegen:

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

Wenn wir die Url ( training/hello/world ) jetzt aufrufen, erhalten wir eine Fehlermeldung:


    Server error: script 'hello/world.phtml' not found in path
    (/usr/local/icingaweb-modules/training/application/views/scripts/)

Praktischerweise erzählt sie uns gleich schon, was wir als nächstes machen müssen.

## Ein View-Script anlegen

Das entsprechende Basis-Verzeichnis fehlt noch. Da wir pro "action" ein View-Skript in einer dedizierten Datei anlegen, gibt es ein Verzeichnis pro "controller":

    mkdir -p training/application/views/scripts/hello
    
Das View-Skript heißt dann einfach so wie die "action", also world.phtml:

```php
<h1>Hello World!</h1>
```

Das war's auch schon, unsere neue URL ist jetzt verfügbar. Wir könnten jetzt den vollen Bereich für unser Modul nutzen und es entsprechend stylen. Wir können aber auch auf ein paar vordefinierte Elemente zurückgreifen. Zwei wichtige Klassen sind z.B. `controls` und `content` für Header-Elemente und den Seiteninhalt.

```php
<div class="controls">
<h1>Hello World!</h1>
</div>

<div class="content">
Here you go...
</div>
```

Damit erhält man automatisch gleichmäßige Abstände zu den Seitenrändern und erzielt zudem den Effekt, dass beim Scrollen nach unten die `controls` stehen bleiben, während der `content` scrollt. Das werden wir natürlich erst dann bemerken, wenn wir unser Modul mit mehr Inhalt befüllen.

## Menü-Einträge

Menü-Einträge in Icinga Web 2 können einerseits personalisiert und / oder vom Administrator vorgegeben werden (*). Unabhängig davon können sie aber von Modulen bereitgestellt werden. Hierbei handelt es sich um globale Konfiguration, die im Basis-Verzeichnis des eigenen Moduls in der `configuration.php` vorgenommen werden kann:

```php
<?php

$this->menuSection('Training')
     ->add('Hello World')
     ->setUrl('training/hello/world');
```

### Icons für Menü-Einträge

Damit unser Menüpunkt besser aussieht, verpassen wir ihm bei dieser Gelegenheit gleich noch ein Icon:

```php
<?php

$this->menuSection('Training')
     ->setIcon('thumbs-up')
     ->add('Hello World')
     ->setUrl('training/hello/world');
```

Um herauszufinden, welche Icons zur Verfügung stehen, aktivieren wir unter `Konfiguration` / `Module` das `doc`-Modul. Anschließend finden wir die Icon-Liste unter `Dokumentation` / `Entwickler - Style`. Es handelt sich hierbei um Icons welche in eine Schriftart eingebettet wurden. Das hat den großen Vorteil, dass sehr viel weniger Requests über die Leitung müssen - die Icons sind einfach "immer da".

Alternativ lassen sich auf Wunsch aber immer noch klassische Icons (.png etc) benutzen. Das ist vor allem dann nützlich, wenn man für sein Modul ein spezielles Icon (z.B. ein Firmenlogo) nutzen möchte, welches sich schlecht in den offiziellen Icinga Icon-Font integrieren lässt:

```php
<?php

$this->menuSection('Training')->setIcon('img/icons/success.png');
```

## Bilder hinzufügen

Wenn man in seinem Modul eigene Bilder nutzen möchte, stellt man diese einfach unter `public/img` bereit:

    mkdir -p public/img
    wget https://icinga.com/wp-content/uploads/2016/02/icinga_icon.png
    mv icinga_icon.png public/img/

Unsere Bilder sind sofort via Web erreichbar, das URL-Muster ist wie folgt:

    http(s)://<icingaweb>/img/<module>/<bild>

Für unseren konkreten fall also `http://localhost/img/training/icinga_icon.png`. Das lässt sich so auch wunderbar gleich in unserem View-Skript nutzen. Anstatt einen img-Tag anzulegen (was natürlich möglich wäre) nutzen wir einen der vielen praktischen View-Helper:

```php
...
<div class="content">
<?= $this->img('img/training/icinga_icon.png', array('title' => 'Icinga Icon')) ?> Here you go...
</div>
```

## Aufgabe

Erstelle die URLs `training/hello/test` und `training/say/hello` und füge jeweils einen zusätzlichen Menüpunkt hinzu. Suche zudem für unser Training-Modul ein schöneres Icon aus dem Internet und richte es entsprechend ein.

## Dashboards

Bevor wir uns um ernsthafte Themen kümmern, wollen wir unsere nützliche URL natürlich noch als Default-Dashboard bereitstellen. Auch das lässt sich in der `configuration.php` erledigen:

```php
<?php
$this->dashboard('Training')->add('Hello', 'training/hello/world');
```

# Wir brauchen Daten!

Nachdem unsere Web-Routen jetzt so wunderbar funktionieren, wollen wir natürlich etwas Sinnvolles damit anstellen. Eine Anwendung kann noch so schön sein, ohne nützliche Inhalte wird sie schnell langweilig. In einer MVC-Umgebung besorgen sich für gewöhnlich die `Controller` mit Hilfe der `Models` ihre Daten und befüttern damit die `View`.

## Unsere View mit Daten befüllen

Der Controller stellt in `$this->view` einen Zugriff auf unsere View bereit. Auf diesem Wege lässt sie sich ganz bequem betanken:

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

Wir erweitern jetzt unser View-Skript und stellen die übermittelten Daten entsprechend dar:

```php
<h3>Some data...</h3>

This example is provided by <a href="http://www.icinga.com">Icinga
</a> 
and based on <?= $this->application ?>.

<table>
<?php foreach ($this->moreData as $key => $val): ?>
    <tr><th><?= $key ?></th><td><?= $val ?></td></tr>
<?php endforeach ?>
</table> 
```

## Aufgabe

Unter `training/list/files` soll der Inhalt unseres Modul-Verzeichnisses in Tabellen-Form aufgelistet werden.
* Hinweis: mit `$this->Module()->getBaseDir()` ermitteln wir unser Modul-Verzeichnis
* Mehr zum öffnen von Verzeichnissen unter [http://de.php.net/opendir](http://de.php.net/opendir)

# Aber bitte mit Stil!

Das hat jetzt zwar nicht direkt mit unserem Thema zu tun, aber eins fällt auf: unsere Tabelle ist nicht gerade besonders hübsch. Zum Glück können wir in unser Modul ganz bequem auch CSS packen. Wir erstellen dazu ein passendes Verzeichnis, der Name dürfte naheliegend sein:

    mkdir public/css

Unsere CSS-Anweisungen legen wir anschließend dort in der Datei `module.less` ab. Less ist eine CSS-Erweiterung um allerhand Funktionen, mehr dazu findet sich unter [der offiziellen Website](http://lesscss.org/functions/). Herkömmliches CSS ist hier aber auf jeden Fall gültig. Das Schöne an Icinga Web ist nun, dass ich mir keine Gedanken darüber machen muss, ob mein CSS andere Module oder Icinga Web selbst beeinflusst: das ist nicht der Fall.

So können wir problemlos folgendes definieren, ohne fremde Tabellen "kaputt" zu machen:

    table {
        width: 100%;
    }

    th {
        width: 20%;
        text-align: right;
        line-height: 2em;
        padding-right: 2em;
    }

Wenn wir in den Entwickler-Tools unseres Browsers die Requests beobachten sehen wir, dass Icinga Web als einzige CSS-Datei css/icings.min.css lädt. Wir können auch css/icinga.css laden um uns bequem anschauen zu können, was Icinga Web aus unserem CSS-Code gemacht hat:

    .icinga-module.module-training table {
      width: 100%;
    }
    .icinga-module.module-training th {
      width: 20%;
      text-align: right;
      line-height: 2em;
      padding-right: 2em;
    }

Wie wir sehen wird durch entsprechende Präfixe sichergestellt, dass unser CSS immer nur in jenen Containern gilt, in denen unser Modul seine Inhalte darstellt.

## Nützliche CSS-Klassen

Icinga Web 2 stellt eine Reihe von CSS-Klassen bereit, die uns die Arbeit einfacher machen. So ist `common-table` nützlich für die üblichen Listen in Tabellen, `name-value-table` Für Name/Wert-Paare bei denen links der Bezeichner als th und rechts der entsprechende Wert in einem td dargestellt wird. Hilfreich ist auch `table-row-selectable` - damit verändert sich das Verhalten der Tabelle. Die ganze Zeile wird hervorgehoben, wenn man mit der Maus darüber fährt. Wenn man irgendwo klickt, kommt der erste Link der Zeile zum Zug. In Kombination mit `common-table` sieht das Ganze ohne zusätzliche Arbeit dann auch schon gleich gut aus.

# Echte Daten aufgeräumt

Wie wir vorhin gesehen haben, wird so ein Modul erst mit echten Daten so richtig interessant. Was wir allerdings falsch gemacht haben ist, dass unser Controller die Daten selbst besorgt. Das ist unschön und würde uns spätestens wenn wir diese Daten auch auf der CLI nutzen möchten Probleme bereiten.

## Unsere eigene Bibliothek

Wir erstellen für unsere Bibliothek wieder ein neues Verzeichnis in unserem Module und folgen dabei dem Schema `library/<Modulname>`. In unserem Fall also:

    mkdir -p library/Training

Für unser Modul nutzen wir wie schon gelernt den Namensraum `Icinga\Module\<Modulname>`. Alle darunter befindlichen Namespaces sucht Icinga Web 2 automatisch im eben erstellten Verzeichnis. Ausnahmen sind:  

* `Clicommands`  
* `Controllers`  
* `Forms` 

Eine Biblithek, welche die in der Übung umgesetzte Aufgabe erledigt, könnte in `File.php` liegen und wie folgt aussehen:

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

Unser Controller kann die Daten jetzt ganz bequem über unsere kleine Library abholen:

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

## Aufgabe

Setze diese oder eine vergleichbare Library in dein Modul. Stelle ein View-Skript bereit, welches passend dazu die einzelnen Files auflisten kann. Wichtig dabei: benutze `$this->escape()` im View-Skript, um Daten deren Herkunft unsicher ist (z.B. Dateinamen) entsprechend zu escapen.

# Parameter-Handling

Bisher haben wir an unsere URLs noch keine Parameter mitgegeben. Auch das ist aber ganz einfach. Wie auf der Kommandozeile steht uns in Icinga Web ein simpler Zugriff auf Params zur Verfügung. Der Zugriff darauf erfolgt wie gewohnt:

```php
<?php
$file = $this->params->get('file');
```

Auch `shift()` und Konsorten sind hier natürlich wieder verfügbar.

## Aufgabe

Unter `training/file/show?file=<filename>` sollen nach Belieben zusätzliche Infos zur gewünschten Datei angezeigt werden. Fleißige zeigen Eigentümer, Berechtigungen, letzte Änderung und Mime-Type an - es reicht aber auch völlig einfach nur erneut Dateiname und Größe "in schön" darzustellen.

## Weiterführende Links

In unserer Dateiliste wollen wir jetzt von jeder Datei zum entsprechenden Detailbereich verlinken. Um keine Probleme mit dem Escaping von Parametern zu bekommen, nutzen wir einen neuen Helper, `qlink`:

```php
<td><?= $this->qlink(
    $file->name,
    'training/file/show',
    array('file' => $file->name)
) ?></td>
```

Der erste Parameter ist hierbei der anzuzeigende Text, der zweite der zu erstellende Link und an dritter Stelle optionale Paramteter für diesen Link. Als vierten Parameter könnte man noch ein Array mit beliebigen weiteren HTML-Attributen mitgeben.

Wenn wir jetzt in unserer Liste eine Datei anklicken, landen wir bei den entsprechenden Details dazu. Doch das geht auch bequemer. Probiere einfach mal, `data-base-target="_next"` in das content-div zu setzen:

    <div class="content" data-base-target="_next">

Damit steuern wir erstmals ohne großen Aufwand das mehrspaltige Layout von Icinga Web!

# URL-Handling

Wer beobachtet hat wie der Browser sich verhält, der hat vielleicht bemerkt, dass hier nicht bei jedem Klick die Seite neu geladen wird. Icinga Web 2 fängt sämtliche Requests ab und versendet sie eigenständig per XHR-Request. Serverseitig wird dies erkannt, und als Antwort dann lediglich das jeweilige HTML-Schnipsel versendet. Das entspricht meist lediglich dem vom entsprechenden View-Script erstellten Output.

Trotzdem bleibt jeder Link ein Link und lässt sich z.B. in einem neuen Tab öffnen. Hier wiederum wird erkannt, dass es sich um keinen XHR-Request handelt, das vollständige Layout wird ausgeliefert.

Für gewöhnlich landen Links immer im selben Container, man kann das Verhalten mit `data-base-target` aber beeinflussen. Das am nächsten am angeklickten Element liegende Attribut gewinnt dabei. Will man `_next` für einen Teilbereich der Seite wieder aufheben, setzt man dort einfach `data-base-target="_self"`.

# Daten-Handling leichtgemacht

Icinga Web bietet noch einiges an netten Hilfmitteln. Eines wollen wir noch begutachten, die sogenannten DataSources. Wir binden die ArrayDatasource ein und erweitern unseren Library-Code um eine weitere Funktion:

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

Anschließend ändern wir auch unseren Controller ganz leicht ab:


```php
<?php
$query = Directory::selectFiles(
    $this->Module()->getBaseDir()
)->order('type')->order('name');

$this->view->files = $query->fetchAll();
```

## Aufgabe 1
Baue die Liste so um, dass man sie per Mausklick auf- oder absteigend sortieren kann.

## Zusatzaufgabe

```php
<?php

$editor = Widget::create('filterEditor')->handleRequest($this->getRequest());
$query->applyFilter($editor->getFilter());
```

## Autorefresh

Als Monitoring-Oberfläche ist es selbstverständlich, dass Icinga Web eine zuverlässig und stabile Autorefresh-Funktion mitliefert. Diese lässt sich bequem aus den Controllern steuern:

```php
<?php

$this->setAutorefreshInterval(10);
```

## Aufgabe 2

Unsere Datei-Liste soll automatisch aktualisiert werden, die Detail-Infos ebenfalls. Zeige die Änderungszeit einer Datei an (`$file->getMtime()`) und benutze den Helper `timeSince` um die Zeit darzustellen. Ändere eine Datei auf der Festplatte und beobachte, was passiert. Wie kann man das erklären?

# Konfiguration

Wer ein Modul entwickelt möchte dieses vermutlich auch konfigurieren können. Konfiguration für ein Module legt man unter `/etc/icingaweb/modules/<modulename>` ab. Was sich dort in einer `config.ini` findet ist im Controller wie folgt zugänglich:

```php
<?php
$config = $this->Config();

/*
[abschnitt]
eintrag = "wert"
*/
echo $config->get('abschnitt', 'eintrag');

// Ergibt "standardwert", da "keineintrag" nicht existiert:
echo $config->get('abschnitt', 'keineintrag', 'standardwert');

// Liest aus der special.ini statt aus der config.ini:
$config = $this->Config('special');
```

## Aufgabe
Der Basis-Pfad für den List-Controller unseres Training-Moduls soll konfigurierbar sein. Ist kein Pfad konfiguriert, benutzen wir weiterhin unser Modulverzeichnis.

# Übersetzungen
Für eine detaillierte Beschreibung der Übersetzungsmöglichkeiten öffen wir die Dokumentation zum `translation` Modul. Hier kurz die einzelnen Schritte:

```php
<h1><?= $this->translate('My files') ?></h1>
```

    apt-get install gettext poedit
    icingacli module enable translation
    icingacli translation refresh module training de_DE
    # Übersetzen mit Poedit
    icingacli translation compile module training de_DE


# Icinga Web Logik in Drittsoftware nutzen?

Wir wollen mit Icinga Web 2 nicht nur das Einbinden von Drittsoftware möglichst einfach gestalten. Wir wollen auch, dass es anderen einfach fällt, Icinga Web Logik in deren Software zu nutzen.

Dazu reicht im Grunde folgender Aufruf in einer beliebigen PHP-Datei:

```php
<?php

require_once 'Icinga/Application/EmbeddedWeb.php';
Icinga\Application\EmbeddedWeb::start();
```

Fertig! Keine Authentifizierung, kein Bootstrapping der vollen Web-Oberfläche. Aber alles was an Library-Code vorhanden ist, kann genutzt werden.

## Aufgabe
Erstelle eine zusätzliche PHP-Datei, welche Icinga Web 2 einbettet. Benutze anschließend die Bibliothek zum Verzeichnis-Handling aus deinem Trainingsmodul.

# Abschließend

Hiermit hast du es geschafft, die Grundlagen der Modulentwicklung für Icinga Web 2 hast du nun gelernt - für alles Weitere gilt es: ausprobieren! Für weitere Hilfestellungen gibt es auf [Icinga Exchange](https://exchange.icinga.com/) und auf unseren [Icinga Events](https://icinga.com/events/) mit Sicherheit allerhand Inspirationen für dich!

Viel Freude und fröhliches Hacken mit Icinga Web 2!!!

