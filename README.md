

# Alvine

Das Alvine-JS-Framework kann über das [CDN][cdn.alvine.io/libs/alvine/framework/alvine.framework-0.9.1.min.js] geladen werden.
Die Integration erfolgt über einen Script-Tag in der HTML-Seite

    <script src="//cdn.alvine.io/libs/alvine/framework/alvine.framework-0.9.1.min.js"></script>

## Core

### Exception

Das zentrale Exception-Objekt kapselt die Nachricht beim 
Wurf einer Exception. Die Exception ist in keinem Namespace
definiert und global im Framework verfügbar.

    throw new Exception

### Namespace

Der Namespace des Alvine-Framework besteht aus verschachtelten
Namespace-Objekten. Mit Hilfe der Funktion Alvine.assignToNamespace()
können interne Funktionen veröffentlicht werden. Hierbei muss der
Name vor dem Verschleiern (obfuscation) mit einem [Hint][minify] geschützt 
werden.

Die internen Funktionen myFkt1 - myFkt3 sind erstmal nur innerhalb 
des Funktionskörpers sichbar und können von außen nicht aufgerifen werden.

Durch die Verwendung von 

    Alvine.assignToNamespace("Alvine.Demo.My", myFkt1, myFkt2); // , ... (beliebig viele Methoden)
    
werden die Funktionen myFkt1 und myFkt2 im Namespace Alvine.Demo.My registriert und sind
ab diesem Zeitpunkt über Alvine.Demo.My.myFkt1() aufrufbar. Im folgenden noch das gesamte
Beispiel.

    (function(){
       Alvine.assignToNamespace("Alvine.Types.My", myFkt1, myFkt2); // , ... (beliebig viele Methoden)
    
       function myFkt1() {console.log('OK-1');}
       function myFkt2() {console.log('OK-2');}
       function myFkt3() {console.log('OK-3');}
    }());
    
    Alvine.Types.My.myFkt1()
    // -> OK-1
    Alvine.Types.My.myFkt2()
    // -> OK-2
    Alvine.Types.My.myFkt3()
    // -> Uncaught TypeError: Alvine.Demo.My.myFkt3 is not a function(…)

Die dritte Funktion myFkt3 wurde nicht veröffentlicht und führt folglich beim Aufruf zu einem Fehler.

## Math

Sammlung von verschiedenen Mathematikfunktionen wie zur Erstellung von Zufallszahlen. Die Standardmethode
Math.rand() erzeugt keine echte Zufallszahl. Diese Lücke soll mit dem Random-Objekt geschlossen werden. 

    Alvine.Math.Random.create()
    // -> 0.113176439 
    
Mittels der Funktion  kann auch eine maximale und eine minimale Grenze definiert werden.

    // Zufallszahl zwischen 100 und 1000
    Alvine.Math.Random.createInteger(100,1000)
    // -> 845


## Types

Alle im Framework definierten generellen Typen, Objekte, Strukturen sind
im Namespace Alvine.Types zusammengefasst.

### Version

Mit Hilfe des Versionsobjektes lässt sich die Version des Frameworks
abfragen und gegen eine Mindestanforderung prüfen.

    // Neue Version aus Zeichenkette erstellen
    version = new Alvine.Types.Version('3.2.1');
    
    // Neue Version aus integer erstellen
    version = new Alvine.Types.Version(3,2,1);
    
    //Nur Hautversionsnummer (ergibt 2.0.0)
    version = new Alvine.Types.Version('2');
    
Eine Version kann gegen eine Andere mit der Funktion Version.compareTo() geprüft werden.
Diese Methode gibt 1 zurück, wenn das Objekt größer als der übergebene Wert ist und -1 im 
umgekehrten Fall. 0 wird zurück gegeben, wenn die beiden Versionen identisch sind.
      
    var version = new Alvine.Types.Version(4,3,18);
    if(version.compareTo(new Alvine.Types.Version(3,3,2))>0) {
       // ... code
    };    
    
### Observer

Das Observer-Objekt nimmt neben einer Callback-Funktion noch weitere Argumente auf. Dieses Objekt
dient als Übergabe-Objekt für die verschiedenen Observer Pattern.

    observer = new Alvine.Types.Observer(function() {
                            // ...
                        }, 'arg1', 'arg2', ...); // arg1, arg2, ... wird an update als Parameter übergeben
    observer.update(subject);

### Map

Eine Map optimiert den Zugriff auf Objektfunktionen mittels Map.set() und Map.get()

    var map = new Alvine.Types.Map()
    // Key/Value setzen
    map.set('marke','bmw')
    // Weiteren Wert
    map.set('farbe','rot')
    
    // Werte holen
    map.get('farbe');
    // -> rot
    
Über einen zeiten Parameter kann ein Default-Wert definiert werden. Ist in der Map der
entsprechenden Schlüssel nicht definiert wird der Defaultwert zurückgegeben.

    var map = new Alvine.Types.Map()
    // Werte holen
    map.get('leistung','700');
    // -> 700 (leistung ist kein Schlüssel in der Map)
    
    
Die Map.toString() Funktion gibt alle Key/Values als Zeichenkette aus. Über die Eigenschaften
Map.keyValueSeparator (Standard :) und Map.entrySeparator (Standard ,) können die Trennzeichen 
definiert werden.

    map.toString()
    // -> "marke:bmw,farbe:rot"
    
Möchte man Änderungen an einer Map überwachen, so kann man mittels eines Observers-Objektes
eine Funktion an die Map anhängen.

    observer = new Alvine.Types.Observer(function(map) {
                            console.log('updated');
                        });
        // -> updated

    // Observer einhängen
    map.attachObserver(observer);
    
    // ruft die im Observer definierte Callback-Funktion auf.
    map.set('key','value')
    // -> updated
    
__ACHTUNG:__ Die Observer werden nur bei Änderungen an der Map
aufgerufen und nicht bei Änderungen von Objekten die in der Map
eingetragen sind.

    obj = {}
    
    // update() wird nicht aufgerufen
    map.set('key', obj);
    
    // update() wird nicht aufgerufen
    obj.new='3';


### Collection

Eine Collection ist im wesentlichen ein erweitertes Array und kann beliebige Objekte und Werte aufnehmen.

    var collection = new Alvine.Types.Collection();
    collection.append(new Alvine.Util.UUID());
    collection.append('Hans Mustermann');
    collection.append('Berlin');
    collection.append(1425);
    collection.append(4.5);
    collection.toString();
    // -> 7e571823-2c11-4237-84e6-82374cb54878,Hans Mustermann,Berlin,1425,4.5
    
Das Trennzeichen bei der Ausgabe ist im Standard ein Komma, dies kann aber über die Eigenschaft *separator*
geändert werden. 

Eine Collection kann auch auf bestimmte Objekte eingeschränkt werden. So kann über den Konstruktor der
Name von Objekten die erlaubt sind angegeben werden. Wird ein anderer Wert übergeben, so wird eine
Exception geschmissen.

    var collection = new Alvine.Types.Collection('Alvine.Util.UUID');
    collection.append(new Alvine.Util.UUID());
    // -> OK
    collection.append('string value');
    // -> exception

Objekte die man mit dieser Einschränkung benutzen will, müssen die Eigenschaft *objectName* besitzen.

    obj = {
      objectName: 'myName';
    }
    var collection = new Alvine.Types.Collection('myName');

Die beiden Funktionen *Collection.checkLimitation* und *Collection.isLimitToObject* zeichnen für die
Überprüfung verantwortlich.

Um über eine Collection zu iterieren, kann die Methode *Collection.each* verwendet werden. Die Methode
erwartet eine Callbackfuntion die mit den in der Collection gespeicherten Werten und Objekten aufgrufen 
wird.

    collection.each(function(value) {
        // 
    });
    
Eine weitere wichtige Methode ist *Collection.contains*. Diese Methode prüft ob ein Wert in der Collection
enthalten ist. Die Prüfung, ob ein Wert enthalten ist, erfolgt über den identisch Operator ===.

    version = new Alvine.Types.Version('1.2.3');
    collection = new Alvine.Types.Collection();
    collection.append('Hans Mustermann');
    collection.append('Berlin');
    collection.append(version);
    collection.append(1425);
    collection.append(4.5);

    collection.contains(1425);
    // -> true
    collection.contains(4.5);
    // -> true
    collection.contains(4.9);
    // -> false
    collection.contains('München');
    // -> false
    collection.contains('Berlin');
    // -> true
    collection.contains(version);
    // -> true
    collection.contains(new Alvine.Types.Version('1.2.3'));    
    // -> false
    
Einträge aus der Collection entfernt man mit der Methode Collection.remove().    

Möchte man Änderungen an einer Collection überwachen, so kann man mittels 
eines Observers-Objektes eine Funktion an die Map anhängen.

    observer = new Alvine.Types.Observer(function(collection) {
                            console.log('updated');
                        });
        // -> updated

    // Observer einhängen
    collection.attachObserver(observer);
    
    // ruft die im Observer definierte Callback-Funktion auf.
    collection.append('value')
    // -> updated
    
__ACHTUNG:__ Die Observer werden nur bei Änderungen an der Collection
aufgerufen und nicht bei Änderungen von Objekten die in der Collection
hängen.   

### Type

In Type sind Hilfsfunktinen wie Alvine.Types.Type.getType() gebündelt. Mit der Funktion 
Alvine.Types.Type.getType() kann der Type einer Variable genauer als mit typeof bestimmt werden.

    Alvine.Types.Type.getType(4)
    // -> "number"
    Alvine.Types.Type.getType(new Alvine.Types.Version(1))
    // -> "object"
    Alvine.Types.Type.getType(2)
    // -> "number"
    Alvine.Types.Type.getType()
    // -> "undefined"
    Alvine.Types.Type.getType(1)
    // -> "number"
    Alvine.Types.Type.getType([1,2])
    // -> "array"
    Alvine.Types.Type.getType({})
    // -> "object"
    Alvine.Types.Type.getType(this)
    // -> "global"
    Alvine.Types.Type.getType(window)
    // -> "global"
    
### ID

Das ID-Objekt wird mit einem eindeutigen Wert initialisiert und ist besonders für zufällige ID
für DOM-Objekte sinnvoll. Die ID ist nur innerhalb einer Webseite eindeutig und nicht kryptografisch
abgesichert. Für eine eindeutigere ID sollte man auf die [UUID](#uuid) zurückgreifen.

     id = new ID();
     id.toString()
     // -> Mzn6
    
## I18N

Unter Lokalisierung versteht man die Zuordnung von Merkmalen wir Sprache, Bilder, Logos zu einer Webseite.

### Lokalisierung

Bei der Anzeige einer Sprachversion gibt es einige Herausforderungen. Welche Sprache soll man anzeigen? Die bestimmung der Sprache sollte
auf Serverseite erfolgen, da hier alle Daten zur Verfügung stehen und genügend Resourcen für die Auswertung gegeben sind. Das Frontend sollte
nicht mit eine umfangreichen Lokalisierung aufgebläht werden.

#### Dokumentensprache auslesen

Die wichtigste Funktion ist in diesem Zusammenhang die Funktion:

    Alvine.i18n.getLocaleOfDocument();
    
Mit Hilfe dieser Funktion kann die Locale des Dokuments ermittelt werden, die der Server festgelegt hat. Will man zusätzlich eine
andere Defaultsprache als englisch festlegen, so muss man noch den gewünchten Parameter übergeben.

    Alvine.i18n.getLocaleOfDocument('de');
    
Ist im Dokument selber kein Locale definiert, dafür aber in der URI, so kann man die
Option scannen der URI aktivieren. Dazu muss die Position der Locale in der URI übergeben werden. Ist die URL zum 
Beispiel http://www.example.com/part/de/page so kann man mit dem ersten Parameter das de aus der URL übernehmen:

    Alvine.i18n.getLocaleOfDocument(1);
    
Das Ergebnis dieser Methode ist ein Objekt vom Typ Locale, zum Beispiel:

    Locale {
       language: "en", 
       region: "CN", 
       script: "Hans", 
       variants: "rozaj", 
       extensions: undefined
       }
           
#### Dokumentenlokale setzen           
           
Nach dem ersten Abfragen der Sprache wird der Wert zwischengespeichert und nicht erneut aus dem Dokument ermittelt. Möchte man
die Sprache des Dokuments ändern, so muss man die Funktion 

    Alvine.i18n.setDocumentLocale(locale);
    
aufrufen. Diese Funktion schreibt den Wert der Locale in den HTML-Tag und setzt die interne Variable auf den neuen Wert. Der
Parameter locale kann etweder eine Zeichenkette oder ein Objekt vom Typ Locale sein.    
    
#### Lokale aus Zeichenkette erstellen    
    
Um ein Lokale-Objekt zu erstellen, kann auf die Funktion Alvine.i18n.getLocale() zurückgegriffen werden. Diese Funktion zerlegt
eine Sprach-Zeichenkette und gibt ein Locale-Objekt mit den entsprechenden Eigenschaften zurück.

    Alvine.i18n.getLocale('en_Hans_CN_nedis_rozaj_x_prv1_prv2');    

    Locale {
       language: "en", 
       region: "CN", 
       script: "Hans", 
       variants: "rozaj", 
       extensions: undefined
       }
       
Diese Methode hat einige Einschränkungen bzg der Erkennung mehrerer Varianten und privaten Erweiterungen, die aber nicht
für den normalen Betrieb einer Webseite ins Gewicht fallen dürfte.

    Alvine.i18n.getLocale('zh-cmn-Hans-CN');
    
    Locale {
       language: "zh", 
       region: "CN", 
       script: "Hans", 
       variants: undefined, 
       extensions: "cmn"
       }
       
    Alvine.i18n.getLocale('de-AT');
        
    Locale {
       language: "de", 
       region: "AT", 
       script: undefined, 
       variants: undefined, 
       extensions: undefined
       }     
       
#### Umwandungin Zeichenkette       
       
Bei der Umwandlung der Locale in eine Zeichenkette wird die Locale::toString() aufgerufen. Diese Funktion
besitzt als Parameter einen Filter. Mit Hilfe dieses Filters lassen sich beliebige Variationen der Eigenschaften 
erstellen.       

    locale = Alvine.i18n.getLocale('zh-cmn-Hans-CN');
    
    // Ausgabe der Locale als Zeichenkette
    console.log(locale.toString());
    // -> zh-cmn-Hans-CN

    // Nur die Region
    console.log(locale.toString(['region']));
    // -> CN
    
    // Die Region gefolgt von der Sprache
    console.log(locale.toString(['region','language']));
    // -> CN-zh    

#### Resourcendatei laden

Lokalisierte Zeichenketten können aus einer Resourcendatei nachgeladen werden. Wird in einem Dokument
mit der Lokalen zh-cmn-Hans-CN die Methode Alvine.i18n.loadLocaleResource() aufgerufen, 

    Alvine.i18n.loadLocaleResource()
    
so versucht die Funktion der Reihe nach folgende Dateien zu laden:

* /resource/locale/zh-cmn-Hans-CN.json
* /resource/locale/zh-cmn-Hans.json
* /resource/locale/zh-cmn.json
* /resource/locale/zh.json

Um Resourcen-Dateien von einer anderen URL zu laden, muss der Funktion der Parameter url übergeben werden.

    Alvine.i18n.loadLocaleResource('mylocale')

Jetzt werden die Dateien im Verzeichnis mylocale/ gesucht:

* /mylocale/zh-cmn-Hans-CN.json
* /mylocale/zh-cmn-Hans.json
* /mylocale/zh-cmn.json
* /mylocale/zh.json

Wird eine Datei gefunden und vom Server mit Status 200 zurückgeliefert, so werden weitere Versuche abgebrochen.
Um an die geladene Sprachdatei zu kommen, muss das Event *loadresource.DONE* abonniert werden.

    jQuery(window).on('loadresource.DONE', function(event, resource) {
       // code
    })
    
Fehler beim Laden einer Datei können über das Event *loadresource.FAIL* abonniert werden.

    jQuery(window).on('loadresource.FAIL', function(event, resource) {
       // code
    })
    
Das Ergebnis ist eine Objekt vom Typ Resouren. Mit der Methode Resource::get() können die
einzelnen Texte geholt werden. Resource ist von Map abgeleitet ([siehe Map](#map))

    resource.get('key')
    // -> mein text 
    
    
## Markup

Funktionen und Objekte für die Manipulation von Markups wie HTML.

### HTML

Die Funktionen und Objekte in diesem Namespace dienen der Manipulation der HTML-Seite.

#### Dataset

Das Dataset-Objekt ist eine Erweiterung des Map-Objektes und für die Verwendung mit dem Template-Objekt
optimiert.

    dataset = new Alvine.Markup.Html.Dataset();
    dataset.set('h1', 'My Headline');
    
Datasets lassen sich auch beliebig verschachteln und mit anderen Objekten füllen.

    dataset = new Alvine.Markup.Html.Dataset();
    inner = new Alvine.Markup.Html.Dataset();
    dataset.set('index0', inner);
    
Datasets lassen sich auch durch einfache Objekte initialisieren. Dazu muss als Wert im Konstruktor 
lediglich ein Objekt übergeben werden. Der Konstruktor erstellt aus diesem Objekt eine Datenstruktur aus
*Alvine.Types.Map* für Objekte und *Alvine.Types.Collection* für echte Arrays

    obj = {
        "headline": "Staedte",
        "cities": [
            "Muenchen",
            "Frankfurt",
            "Berlin"
        ]
    };

    dataset = new Alvine.Markup.Html.Dataset(obj);    
    
    dataset.get('cities')
    // ist vom Typ Alvine.Types.Collection
    
    dataset.get('cities').toString
    // -> Muenchen,Frankfurt, Berlin


#### Template

Das Templateobjekt erlaubt es mit dem neuen HTML5 Template-Tag zu arbeiten. Wie in dem
folgenden Beispiel zu sehen ist der neue [Template-Tag](http://www.html5rocks.com/en/tutorials/webcomponents/template/) 
eine gute Möglichkeit HTML-Templates direkt im HTML zu definieren. Der Vorteil der Templates
ist, das diese zur Ladezeit nicht gerendert werden und somit keinen nenswerten Speicher
und Rechenleitung verbrauchen.

    <template id="mytemplate" class="templates">
    <div>
        <h1>Template 2</h1>
        <p>Platzhalter</p>
    </div>
    </template>
    
Auf dieses Template kann nun einfach über das Template-Objekt zugegriffen werden.

    template = new Alvine.Markup.Html.Template('#mytemplate');
    
Werden keine Templates mit dem Namen gefunden so wird eine Exception geworfen. Werden mehrere 
Templates gefunden, so wird mit get(0) das erste aus der Liste der gefundenen genommen. Um 
das Template in eine andere Struktur einfügen zu können, kann man mit der Methode 
*Template.getjQueryObject()* ein jQuery-Objekt erstellen und dieses per jQuery.append()
einfügen. Alternativ kann man sich per *Template.getHtmlFragment()* ein echtes DOM-Objekt
holen.

    obj = template.getjQueryObject();
    // -> jQuery-Objekt
    jQuery('<div/>').append(obj);
    
    template.getHtmlFragment()
    // -> Document-Fragment Objekt
      
Den Methoden *Template.getjQueryObject()* und *Template.getHtmlFragment()* kann ein
DAtaset übergeben werden. Das Dataset dient dazu Werte innerhalb des Templates zu ersetzen. 

##### Inhaltsmanipulation

In Anlehung an die PHP-Erweiterung [Alvie.Markup.HTML](https://wiki.schukai.com/display/alvine2/HTML+Komponente)
können auch im Javascript data- Attribute zum Einsatz kommen.

    <template id="template">
    <div>
        <h1 data-replace="dataset:h1">Template</h1>
    </div>
    </template>
    
Um das Template mit Werten zu füllen, muss ein Dataset erstellt und mit Werten erfüllt werden

    dataset new Alvine.Markup.Html.Dataset();
    dataset.set('h1', 'My Headline');
    mytemplate = template.getjQueryObject(dataset);

Das Ergebnis-HTML in *mytemplate* hat dann folgendne Inhalt

    <div>
        <h1>My Headline</h1>
    </div>
        
Neben der einfachen Ersetzung können auch noch weitere Befehle durch Pipes getrennt 
im Attribute übergeben werden. Die einzelnen Befehle bearbeiten immer die Ausgabe des
vorherigen Befehls.

    <template id="template">
    <div>
        <h1 data-replace="dataset:h1 | strtolower">Template</h1>
    </div>
    </template>  
        
 Das Ergebnis des folgendne Templates ist dann eine komplett kleingeschriebene Überschrift.
     
    <div>
        <h1>my headline</h1>
    </div>
        
Folgende Befehle sind verfügbar: 

* strtolower (alles kleingeschrieben), 
* strtoupper (alles großgeschrieben), 
* trim (Leerezichen am Anfang un Ende entfernen)
* plaintext (HTML-Tags entfernen)
* rawurlencode (URL-Codirung)
* ucfirst (Erstes Zeichen groß)
* ucwords (Jeder Wortanfang groß)
* intval (Integer)
* length (Länge der Zeichenkette)
* base64 (Base64)
* htmlspecialchars (HTML-Sonderzeichen nach Entities)
* substring (Teilzeichenkette: Die Parameter werden durch Doppelpunkt getrennt => substring:Anfang:Länge)

Über die Operation Eigenschaft des Template-Objektes lassen sich weitere Funktionen einfügen.
Somit lassen sich alle Manipulationen des Wertes bewerkstelligen. Die eigene Operation
wird über das call Schlüsselwort aufgerufen. Alle Werte die nach dem Namen der Funktion per
Doppelpunkt angegeben werden, werden direkt an die Funktion übergeben.

    <p data-replace="dataset:key | call:getFullname:!">Text</p>   
    
    // Template-Objekt erstellen
    template = new Alvine.Markup.Html.Template(selector);
    
    // Operation definieren
    template.Operation.getFullname = function(value, sign) {
        return value+sign;
    };
    
Ein weiterer Befehl ist *data-replaceself*. Hier wird nicht der Wert des Elements gesetzt, sondern
das gesamte Element eretzt.

    <template id="template">
    <div>
        <h1 data-replaceself="dataset:h1 | strtolower">Template</h1>
    </div>
    </template>
        
Führt also zu folgendem Ergebnis ohne H1 Tags

    <div>
        my headline
    </div>
        
Attribute lassen sich ebeso einfach über ein Dataset setzen. Dazu muss im Tag nur der Befehl data-attributes
gesetzt werden.

    <template id="template7" class="templates">
    <div>
        <a data-attributes="href dataset:url | strtolower, style static:color:red, class dataset:class | strtoupper">Link</a>
    </div>
    </template>
        
Template-Tags eignen sich optimal für die Bereitstellung der benötigten HTML-Struktur. Allerding benötigt man manchmal
ein Template aus einer bestehenden Struktur. Auch damit kann das Template-Objekt umgehen. 

    <div id="domtemplate" data-attributes="id static:newid">
        <p><i data-replace="static:testvalue"></i></p>
    </div>    
    
Beim Erstellen des Template-Objektes den gewünschten Selector angeben. Der umschließende Tag wird in diesem Fall 
allerdings im Template verwendet und nicht wie bei den Template-Tags entfernt. Deshalb ist es wichtig, falls das Element 
wie im obigen Beispiel eine ID besitzt, eine neue ID mittels Attribute-Ersetzung zu setzen.

    template = new Alvine.Markup.Html.Template('#domtemplate');        

Eine weitere Funktionalität des Template-Objektes ist das Wiederholen von Werten aus einer Collection. Dadurch lassen sich
Datasets einfach in HTML-Abbilden. Im folgendne Beispiel soll eine Tabelle mit Städten ausgegeben werden. Dazu wird ein
HTML-Template mit einer Tabelle und mit der zu wiederholenden Zeile angelegt.

    <template id="template" class="templates">
        <div>
            <table>
                <tr data-repeat="city dataset:list">
                    <td data-replace="dataset:city">placeholder</td>  
                </tr>
            </table>
        </div>
    </template>
    
Im nächsten Schritt muss das Dataset definiert werden.

    template = new Alvine.Markup.Html.Template('#template9');
    // Collection mit Städten
    list = new Alvine.Types.Collection();
    list.append('München');
    list.append('Frankfurt');
    list.append('Berlin');
    
    // Collection dem Dataset hinzufügen
    dataset = new Alvine.Markup.Html.Dataset();
    dataset.set('list', list);

    // Diese Anweisung erstellt das fertige HTML 
    template.getHtmlFragment(dataset);    

Das Ergebnis dieses Javascripts ist folgender HTML Schnipsel.

    <table>
        <tbody>
        <tr>
            <td>München</td>  
        </tr>
        <tr>
            <td>Frankfurt</td>  
        </tr>
        <tr>
            <td>Berlin</td>  
        </tr>
        </tbody>
    </table>
    
Das Dataset kann neben der Zuweisung durch die Programmierung auch 
über HTML-Attribute zugewiesen werden. Dazu kann das Dataset entweder direkt 
im Tag angegeben werden oder es wird eine URL definiert über die das
Dataset nachgeladen wird.

    <!-- Beispiel mit Dataset im Attribute -->
    <template id="template" class="templates" data-dataset='{"headline": "Staedte","list": ["Muenchen","Frankfurt","Berlin"]}'>
        <div>
            <table>
                <tr data-repeat="city dataset:list">
                    <td data-replace="dataset:city">placeholder</td>  
                </tr>
            </table>
        </div>
    </template>

Wichtig beim Nachladen des Datasets ist es, dass als Ergebnis ein JSON-Objekt zurückgegeben wird.

    <!-- Beispiel mit URL -->
    <template id="template" class="templates" data-dataurl='http://www.example.com/dataset.json'>
        <div>
            <table>
                <tr data-repeat="city dataset:list">
                    <td data-replace="dataset:city">placeholder</td>  
                </tr>
            </table>
        </div>
    </template>
    
Beim Nachladen der Daten von einer URI gibt es eine Besonderheit zu beachten. Da Ajax-Requests
asynchrone ablaufen steht das Datenset nicht sofort zur Verfügung. Vielmehr muss vor der ausführung der
Template.getHtmlFragmen() Funktion im Code auf das *loaddataset.DONE* Event geprüft werden.

    jQuery(window).on('loaddataset.DONE', function(m, p) {
        try {
            template.getHtmlFragment();
        } catch(e) {
            // ..
        }
    });
    
Der Fehlerfall wird über das Event *loaddataset.FAIL* gemeldet. 

Ganz ohne Javascriptprogrammierung kann man ein Template über die *auto-template* Klasse initialisieren.
Dazu muss man neben der *auto-template* Klasse, dem Dataset (als URL oder Json), auch noch ein Target 
definieren. 

    <template class="auto-template" data-target="#mytarget" data-dataset=...

Das Target-Attribute muss ein jQuery-Selector sein und dient als Ziel für das erstellte 
HTML-Fragment. Das HTML-Fragment wird per jQuery(target).append(template) in das Dokument eingefügt.

    <div id="mytarget"></div>
    
Die Funktionsweise ist dann identisch zu den oben beschrieben Funktionen. 

in der anderen Richtung funktioniert die Zuweisung über die Klasse *auto-render*. Ein Element das diese
Klasse besitzt braucht zusätzlich die Attribute *data-template* und *data-dataset* oder *data-dataurl*.
Die Funktionsweise und Konfiguration ist identisch. Als Template wird das erste, über den im *data-template*
Attribute definierten Selektor, gefundene Element genommen. 

##### DOM-Elemente an Dataset binden

Möchte man einzelne Werte oder eine ganze Collection auf Änderungen überwachen, so kann man das per data-bind-Attribute
im Template definieren. Im folgendne Beispiel wird ein Template mit einer Überschirft und einer Tabelle mit einer Spalte
definiert.

    <div id="targetForTemplate"></div>

    <template id="listTemplate">
        <div>
            <h2 data-replace="dataset:headline" data-bind="true">dummy</h2>
            <table>
                <tr data-repeat="city dataset:list" data-bind="true">
                    <td data-replace="dataset:city">placeholder</td>  
                </tr>
            </table>
        </div>
    </template>

Um das Template mit Daten zu füllen, wird ein Dataset aus einer Liste mit den drei Städten München, Frankfurt und Berlin, erstellt.
        
    dataset = new Alvine.Markup.Html.Dataset({"headline": "Staedte", "list": ["München", "Frankfurt", "Berlin"]});

    template = new Alvine.Markup.Html.Template('#listTemplate');
    // Zuweisen des Templates in einen Div-Container
    jQuery('#targetForTemplate').append(template.getHtmlFragment(dataset));
    
Nach dem Laden der Seite wird eine Tabelle angezeigt. Ändert man nun Werte im Dataset, so ändert sich auch die Darstellung in der
Webseite. 

    // Überschrift neu setzen
    dataset.set('headline', 'Meine Städte');
    // Einen Wert zur Liste hinzufügen
    dataset.get('list').append('Bern');
    // Berlin entfernen
    dataset.get('list').remove('Berlin');
    
## jQuery

Die Funktionen und Objekte in diesem Paket dienen dem Zusammenspiel mit jQuery. Ein einfaches
Plugin besteht aus einer Funktion die ein Ojekt vom Type *Alvine.jQuery.Plugin* zurück gibt.

    namespace = 'my.namespace'

    function LoremIpsum(options) {
        return new Plugin(options);
    }

    function Plugin(options) {
        Alvine.Types.testIsCalledAsConstructor.call(this, 'Plugin', namespace);
    }

    Plugin.prototype = new Alvine.jQuery.Plugin();
    
Um dieses Plugin dem DOM-Element mit der ID *my* zuzuordnen,   

    <div id="my"></div>

muss die Alvine-Methode von jQuery aufgerufen werden.  

    $('#my').asignAlvineComponent(myPlugin,{key:'value'})

Auf die Settings des Elements kann dann über die getSetting-Funktion zugegriffen werden.

    $('#my').get(0).Alvine.myPlugin.getSetting('key')
    // -> value
    $('#my').get(0).Alvine.myPlugin.getSetting('label')
    // -> my plugin
    $('#my').get(0).Alvine.myPlugin.getSetting('undefined')
    // -> undefined
    
Statt im letzten Fall, wenn ein Wert nicht definiert wurde, auf undefined zu prüfen, kann
mann beim Aufruf der Funktion gleich einen Default-Wert übergeben. Dieser Default-Wert wird
im Fall das es den gewünschten Schlüssel nicht gib zurückgegeben.

Der letzte Aufruf würde mit Default-Parameter den Wert *nicht vorhanden* zurück geben.

    $('#my').get(0).Alvine.myPlugin.getSetting('undefined','nicht vorhanden')
    // -> nicht vorhanden
    
Eine weitere Möglichkeit Werte zu definieren besteht in dem notieren von data- Attributen 
direkt im Element. In unserem Beispiel könnten wir direkt im HTML-Tag das Attribut 
*data-myplugin-label* definieren und den gewünschten Wert angeben.

    <div id="my" data-myplugin-label="new label"></div>   
    
Ein Aufruf der Settings würde in diesem Fall *new label* zurück geben.

    $('#my').get(0).Alvine.myPlugin.getSetting('label');
    // -> new label
    
Die Reihenfolge der Auswertung ist dabei immer gleich. Zuerst wird geprüft, ob im Tag ein Wert definiert ist.
Ist kein Wert vorhanden, wurde auch hier kein Wert gesetzt, so wird zum Schluß der Wert aus den Plugin Defaults
herangezogen.

### LoremIpsum

Das LoremIpsum jQuery-Plugin kann als Beispiel für ein einfaches jQuery Plugin gesehen werden. Dieses Plugin erlaubt 
es in einem Tag einen Lorem-Ipsum-Text einzufügen. Dazu muss dem gewünschtem Objekt die Klasse loremipsum gegeben 
werden.

    <div class="loremipsum" data-loremipsum-textkey="loremipsum100"></div>
        
Nach dem Laden der Seite wird das DIV-Element mit einem 100 Wörter umfassenden Text gefüllt

    <div class="loremimpsum" data-loremipsum-textkey="loremipsum100">Lorem ipsum dolor sit amet, consetetur 
    sadipscing elitr, sed diam nonumy eirmod tempor invidunt ut labore et dolore magna aliquyam erat, sed diam 
    voluptua. At vero eos et accusam et justo duo dolores et ea rebum. Stet clita kasd gubergren, no sea takimata 
    sanctus est Lorem ipsum dolor sit amet. Lorem ipsum dolor sit amet, consetetur sadipscing elitr, sed diam nonumy 
    eirmod tempor invidunt ut labore et dolore magna aliquyam erat, sed diam voluptua. At vero eos et accusam et justo 
    duo dolores et ea rebum. Stet clita kasd gubergren, no sea takimata sanctus est Lorem ipsum dolor sit amet.</div>  
        
Wird das Attribute *data-loremipsum-textkey* nicht angegeben, so wird der Default-Wert *loremipsum25* aus dem Plugin genommen.
Alternativ besteht auch die Möglichkeit das Plugin manuell zu initialisieren. Hier darf dem HTML-Tag dann nicht die Klasse
loremipsum gegeben werden und die Initialisierung muss manuell über $(selector).Alvine(...) erfolgen.

Im Plugin selber sind Texte mit den Schlüssel *loremipsum100*, *loremipsum50*, *loremipsum25*, *loremipsum10* und *loremipsum*
definiert.

Man kann das Plugin auch mit Hilfe von Javascript eigenständig initialisieren

    jQuery('.myloremipsum').asignAlvineComponent(LoremIpsum, {'textKey': 'loremipsum50'});
    
Ein Text kann auch direkt über die Api des entsprechenden Objektes gesetzt werden

    jQuery('.myloremipsum').get(0).Alvine.LoremIpsum.setText('loremipsum100');
    ...

Die Abgrage der Settings erfolgt über 

    jQuery('.myloremipsum').get(0).Alvine.LoremIpsum.getSetting('textKey')

## Util

Unter dem Namespace Util finden sich verschiedene Hilfsfunktionen.

### Logger

Mit Hilfe des Loggers können schnell und einfach Funktionen analysiert und Fehler ermittelt werden. Eine Logmeldung
wird mit Hilfe einer der folgendne Funktionen geschrieben:

    Alvine.Util.Logger.logFatal('my fatal');
    Alvine.Util.Logger.logError('my error');
    Alvine.Util.Logger.logWarn('my warn');
    Alvine.Util.Logger.logInfo('my info');
    Alvine.Util.Logger.logDebug('my debug');
    Alvine.Util.Logger.LogTrace('my trace');
    
Als Best-Practise Ansatz sollte man als erstes Argument immer das Modul oder die Funktion übergeben. Damit ist
es dann sehr leicht die Meldungen über die Konsole zu filtern.

    Alvine.Util.Logger.logDebug('alvine.types.string','my message');    
    
Nach dem Aufruf einer dieser Funktionen wird zuerst keine Meldung ausgegeben. Erst wenn man den Trashold 
entsprechend setzt werden Meldungen ausgegebn. Der Trashold kann über eine der folgenden Funktionen gesetzt werden:

    Alvine.Util.Logger.setAll();
    Alvine.Util.Logger.setFatal();
    Alvine.Util.Logger.setError();
    Alvine.Util.Logger.setWarn();
    Alvine.Util.Logger.setInfo();
    Alvine.Util.Logger.setDebug();
    Alvine.Util.Logger.setTrace();
    Alvine.Util.Logger.setOff();
    
Die einzelnen Level schließen immer die darunterliegenden Level ein. So wird mit einem Trashold auf WARN auch alle
Errors und Fatals angeteigt. KeineMeldungen erhält man demnach mit dem Trashold Off. Die Reihenfolge der Auswertung ist 
folgendermassen
    
*ALL > TRACE > DEBUG > INFO > WARN > ERROR > FATAL > OFF*
    
Standardmässig werden die Logmeldungen immer über das Consolenobjekt ausgegeben. Diese wird bereits beim Laden des Frameworks
mit Hilfe der addConsoleHandler() aktiviert.

     Alvine.Util.Logger.addConsoleHandler();
     
Alternativ kann man die Logmeldungen auch an eine URL per POST senden. Dazu muss ein Ajax-Handler definiert werden.

    Alvine.Util.Logger.addAjaxHandler('http://example.com/logging');
    
eigene Loghandler lassen sich ebenfalls einfach hinzufügen. Dazu muss nur das log-Event mit den gewünschten 
Namespaces beobachtet werden. Im folgenden Beispiel werden alle Logmeldungen abgefragt.

    jQuery(window).on('log.TRACE.DEBUG.INFO.WARN.ERROR.FATAL', function () {
        // 
    })
    
Möchte man nur FATAL-Fehler bekommen, so reicht es den Namespace auf log.FATAL zu setzen    

    jQuery(window).on('log.FATAL', function () {
        // 
    })

### UUID

Die UUID Funktion ermöglicht es eine UUID vom Typ 4 (Random)
zu erstellen. Dazu muss einfach nur durch den Konstruktor ein
neues UUID-Objekt erstellt werden.

    // Neues UUID-Objekt
    uuid = new Alvine.Util.UUID()
    uuid.toString()
    // -> "f49dc92c-1ac5-4f45-84bc-2645fce5a14a"
    


[alvineio]: http://cdn.alvine.io/ 
[minify]: minify.html 
