# Tools und Setup

## Tools

Um störungsfrei und komfortabel mit React arbeiten zu können sollten einige Bedingungen erfüllt sein. Nicht alles davon ist **zwingend** notwendig, es erleichtert das Entwicklerleben jedoch ungemein, weswegen ich dennoch **dringend** dazu rate und auch bei allen folgenden Beispielen davon ausgehen werde, dass ihr diese Tools installiert habt:

### Node.js und npm

Node werden die meisten möglicherweise als „serverseitiges JavaScript“ kennen, das ist allerdings nicht die ganze Wahrheit. In erster Linie ist Node einmal eine JavaScript Laufzeitumgebung, die sich eben hervorragend für Netzwerkanwendungen eignet, also klassische Webserver. Darüber hinaus bringt Node auch ein Tool zur Paketverwaltung mit, nämlich **npm**, mit dem sich spielend einfach neue JavaScript-Libraries auf dem eigenen Rechner installieren lassen. Außerdem lassen sich auch eigene Commandline-Scripts damit schreiben und ausführen, was sich später noch als sehr praktisch erweisen wird.

Statt Node direkt zu installieren, empfehle ich nvm \(Node Version Manager\) für Mac und Linux bzw. nvm-windows für Windows. Nvm hat den Vorteil, dass es einerseite keine Admin-Rechte benötigt um Packages global zu installieren und man andererseits mit einem simplen Befehl auf der Kommandozeile \(nvm install \[version\]\) die auf dem System installierte Version aktualisieren kann. Für einer Liste aller verfügbaren Version kannst du ganz einfach nvm ls-remote \(Mac/Linux\) bzw. nvm list available \(Windows\) benutzen. Ich empfehle im weiteren Verlaufe dieses Buch die aktuelle LTS \(Long Term Support\) Version zu benutzen.

### Yarn

Während Node mit npm bereits einen guten und soliden Package-Manager mitbringt, geht yarn noch ein Stück weiter, bietet besseres caching, dadurch auch bessere Performance, einfachere Kommandos und kommt darüber hinaus, wie React, ebenfalls aus dem Hause Facebook und wurde dort entwickelt u.a. um die Arbeit mit React noch etwas angenehmer zu gestalten. Während alles, was hier im weiteren Verlauf des Buches beschrieben wird, auch mit npm ausgeführt werden kann, würde ich dennoch empfehlen Yarn zu installieren, da dies gerade in React-Kreisen mehr und mehr an Gewicht gewinnt, insbesondere wegen seiner Einfachheit und seiner verbesserten Performance ggü. npm. Sind Node und npm erst einmal installiert, lässt sich Yarn einfach als globales Package über npm installieren:

`npm install --global yarn`

oder einfach kurz:

`npm i -g yarn`

Wir haben gerade außerdem unser erstes Package installiert. Easy! Das Commandline-Flag `--global` \(bzw. `-g`\) sorgt dabei dafür, dass die `yarn` Executable global installiert wird und von überall auf eurem Gerät auf der Kommandozeile ausgeführt werden kann.

### Babel

Babel ist ein Tool, das für gewöhnlich lediglich als Dependency \(Abhängigkeit\) und für gewöhnlich als npm-Paket in React basierten Projekten zum Einsatz kommt und an dieser Stelle nicht explizit installiert werden muss. Babel erlaubt es nicht oder _noch_-nicht standardkonformen oder noch nicht von allen gängigen Browsern unterstützten JavaScript-Code in interpretierbaren und ausführbaren Code zu _transpilieren_.

{% hint style="info" %}
Transpilieren \(engl. _transpiling_\) nennt man einen Prozess, bei dem der Sourcecode von einer Sprache in ein entsprechendes funktional identisches Gegenstück einer anderen Sprache umgewandelt wird. In unserem Fall eben von JSX oder ES2015+ in valides, ausführbares und vom Browser unterstütztes JavaScript.
{% endhint %}

Babel besteht aus einem Core-Modul \(`@babel/core`\) das lediglich einige APIs bereitstellt, die dann von **Plugins** für das entsprechende Transpiling verwendet werden. Diese Plugins werden oft zu sog. **Presets** zusammengefasst, die dann wiederum mehrere Plugins gleichzeitig installieren. Die in React basierten Projekten üblichsten Presets sind `@babel/preset-react` \(um JSX zu lesen und zu übersetzen\) und `@babel/preset-env`, welches abhängig von einer Ziel-Umgebung modernes JavaScript so umschreibt, dass es eben auch ältere Browser verstehen.

Das @-Zeichen vor dem Namen bedeutet dabei, dass es sich um eine Organisation innerhalb der npm Registry \(dem npm-Paketverzeichnis\) handelt und kann als eine Art Namespace betrachtet werden. Im Fall von Babel findet man dort die offiziellen Pakete die von den Babel-Maintainern dort veröffentlich werden. Bevor Babel in der Version 7 erschien gab es diese Organisation noch nicht und die Pakete wurden mit einem Bindestrich im Namen getrennt. So hieß `@babel/preset-react` eben `babel-preset-react`, `@babel/core` war `babel-core` usw. Also nicht verwirren lassen, sollte euch in einem Projekt mal `babel-core` statt `@babel/core` begegnen. In diesem Fall handelt es sich also einfach um Babel 6 \(oder eine ältere Version\).

### Webpack

Webpack ist ebenfalls eins der zentralen Tools im React-Ecosystem ohne das ein effizentes Arbeiten mit React kaum möglich oder zumindest deutlich umständlicher wäre. Hier handelt es sich um einen sog. **Module-Bundler**, der Modul basierte Entwicklung, wie sie manch einer vielleicht bereits aus NodeJS kennen mag, in den Browser bringt. Dadurch wird es ermöglicht Anwendungscode übersichtlich in einzelnen Files zu verteilen, die jeweils ihre Abhängigkeiten über `import` oder `require()` in ihren eigenen **Module-Scope** laden und damit innerhalb des Moduls verfügbar machen. Am Ende fällt dann nur noch eine einzelne JavaScript-Datei heraus \(auf Wunsch auch mehrere\), so dass nicht mehr jede einzelne unserer Komponenten, und das können schnell mal über 100 werden, einzeln über `<script src="..."></script>` im HTML eingebunden werden muss.

Wow. Klingt unfassbar kompliziert, passiert aber nach einigen wenigen Beispielen nahezu intuitiv von ganz allein und hat man sich erst einmal daran gewöhnt, wird man sich fragen wie man jemals ohne Module-Bundler arbeiten konnte.

Neben dem Module-Bundling selbst kann Webpack auch beigebracht werden Dateien mit JSX durch Babel in JavaScript zu transpilieren, Bilder, Stylesheets oder andere Assets in einen build-Ordner zu kopieren der später auf einen Server deployed wird und viele andere Dinge. Wie eine solche Konfiguration aussehen kann beleuchten wir später noch einmal genau, weshalb das Webpack Kommandozeilen-Tool auch an dieser Stelle noch nicht installiert werden muss.

### IDE-/Editor-Plugins

Alle Bekannten Editoren und IDEs wie bspw. Webstorm, Atom, Visual Studio Code oder Sublime \(aber auch so ziemlich jeder andere moderne Editor oder IDE\) bietet Plugins oder inzwischen sogar bereits native Funktionen für die bessere Unterstützung für React und JSX. Hier rate ich dringend zur Installation dieser Plugins, da diese in der Regel für deutlich besseres Syntax-Highlighting sorgen, teilweise Code-Vervollständigung und andere Nettigkeiten bieten. In Atom ist das [language-babel](https://atom.io/packages/language-babel), in VS Code gibt es hier u.a. [Babel ES6/ES7](https://marketplace.visualstudio.com/items?itemName=dzannotti.vscode-babel-coloring) und in Sublime lohnt sich in Blick auf [babel-sublime](https://github.com/babel/babel-sublime). Nutzt ihr Webstorm, habt ihr seit Version 10 sogar native Unterstützung für React-Syntaxhighlighting.

### Browser-Plugins

Für den Browser empfehle ich dringend jeweils die React-Devtools für [Chrome](https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi) und [Firefox](https://addons.mozilla.org/de/firefox/addon/react-devtools/) zu installieren, für den späteren Verlauf außerdem die Redux-Devtools für beide Browser \([Chrome](https://chrome.google.com/webstore/detail/redux-devtools/lmhkpmbekcpmknklioeibfkpmmfibljd), [Firefox](https://addons.mozilla.org/de/firefox/addon/remotedev/?)\). Die Devtools fügen sich nahtlos als neuer Tab in die bestehenden Browser-Devtools ein und bieten einen enormen Mehrwert beim Debugging von React-Komponenten.

![Chrome mit installierten Devtools-Plugins f&#xFC;r React und Redux](../.gitbook/assets/image.png)

So lässt sich bspw. der State direkt im Browser manipulieren und die Auswirkungen live beobachten. Ich würde soweit gehen und behaupten, dass ein effizientes Debugging ohne die Devtools-Erweiterungen kaum oder sogar gar nicht möglich ist.

## Setup

Manch einer hat in der Vergangenheit darüber gescherzt, dass man gut und gerne Tage damit verbringen kann ein Setup aufzusetzen bevor man die erste Zeile Code schreibt. Und in der Tat: ein ordentliches Setup ist wichtig, bestimmt es doch ein Stück weit auch die Qualität und Wartbarkeit der Anwendung, die man auf Basis seines Setups entwickelt.

Hier hat die große React-Community aber bereits sehr gute Vorarbeit geleistet. Und so listet die Seite JavaScriptStuff aktuell **189 Projekte** in der Rubrik [**React Starter Projects**](https://www.javascriptstuff.com/react-starter-projects/). Auch Facebook selbst, bzw. konkret **Dan Abramov**, Core-Entwickler bei Facebook und Autor von **Redux**, ist dort mit **Create-React-App** \(„CRA“\) vertreten. Das Projekt ist mit über 45.000 Stars auf Github mittlerweile so etwas wie der de-facto Standard wenn es um React Starter Projekte geht und beschreibt sich auf Github selbst mit:

> Create React apps with no build configuration

Und in der Tat, **Create React App** macht es gerade Einsteigern \(aber nicht nur diesen\) sehr einfach ein sehr robustes und gutes Setup mit nur einem Befehl auf der Kommandozeile zu erzeugen:

`yarn create react-app projektname`

Wer stattdessen npm bevorzugt, muss momentan noch zwei Befehle ausführen:

`npm install -g create-react-app`

… um die **Create React App** Executable global zu installieren und anschließend

`create-react-app projektname`

Und schon wird im Ordner „_projektname_“ ein vollständiges React-Setup mit einigen kleinen Beispiel-Komponenten erzeugt. Ich würde empfehlen dies jetzt einfach mal zu tun, denn die ersten Code-Beispiele werden zu Beginn allesamt auf einem gewöhnlichen CRA-Setup basieren und können so recht einfach ausprobiert werden.

{% hint style="warning" %}
Der Projektname muss den [Kriterien für die `name`-Eigenschaft](https://docs.npmjs.com/files/package.json#name) des `package.json`-Formats von npm haben. Dies bedeutet, neben einigen anderen Kriterien, er darf nur Kleinbuchstaben beinhalten, keine Leerzeichen und dürfen maximal aus 214 Buchstaben bestehen. Die vollständigen Kriterien finden sich in der npm Dokumentation
{% endhint %}

Später werde ich euch dann zeigen wie ihr die `eject`-Funktion benutzt um eigene Änderungen an der Konfiguration vornehmen zu können. Aber für den Beginn \(und auch noch recht weit darüber hinaus\) reicht erst einmal das Basis-Setup, da dieses bereits sehr umfangreich ist und viele Themen abdeckt, so dass wir uns weniger mit dem Setup beschäftigen müssen und direkt in den Code eintauchen können.

Nachdem CRA das Basis-Setup erstellt und seine Paket-Abhängigkeiten \(_Dependencies_\) installiert hat gibt es uns noch eine kurze Anleitung wie wir mit CRA an unserem ersten React-Projekt arbeiten können.

![Create-React-App ist bereit!](https://lh6.googleusercontent.com/im1UToBbzUiXTseQk1AZTD2_WvPVnImgokKU-HLPHPzS06C-H9xdoyL0-xn8q4iDsFCCKXKxERxmuRE7Co0nJTTI1aPEaMg99aS5QXa9xYlwkS0JRVjTV0DM8Yuv8Z83FTgJ8XPN)

### yarn start

Hiermit starten wir einen Entwicklungs-Server über den wir unsere neu erstellte App im Browser aufrufen können. Dieser kümmert sich auch darum alle Dateien im Ordner zu beobachten und unsere App mit all seinen Abhängigkeiten neu zu „kompilieren“ sobald wir eine Änderung an einem der Files vornehmen.

### yarn build

Erstellt einen Build unserer App, die wir dann bspw. auf einen öffentlichen Server deployen können. Dieser Build ist gegenüber dem Development-Build \(`yarn start`\) auf Performance optimiert, weswegen das Ausführen von `yarn build` für gewöhnlich deutlich länger dauert als `yarn start`.

### yarn test

Führt Tests aus. Als Test-Framework bringt CRA das ebenfalls von Facebook entwickelte **Jest** mit. Jest bringt hier aus meiner Sicht einen sehr entschiedenen Vorteil mit gegenüber anderen Testing-Frameworks, nämlich das sog. **Snapshot-Testing** bei dem sozusagen eine Art _Abbild_ des aktuellen Zustands einer Komponente erstellt wird, der den Status Quo darstellt und mit dem zukünftige Test-Zustände verglichen werden. So fallen Änderungen, gewünschte wie ungewünschte, sofort ins Auge.

### yarn eject

Mit `yarn eject` können wir uns von Create-React-App „verabschieden“. Dabei werden alle build-Scripts, Dependencies und Config-Files in das aktuelle Projektverzeichnis kopiert und wir sind fortan selbst verantwortlich das alles korrekt läuft. Dadurch haben wir mehr Verantwortung, aber eben auch deutlich mehr Freiheiten, da wir von nun an eigene Änderungen an der Standard-Konfiguration von CRA vornehmen können. Da CRA aber schon sehr viel mitbringt, werden wir mit diesem Schritt in diesem Buch noch eine ganze Weile warten, bis wir beim Kapitel angekommen sind in der es um die Konfiguration von Webpack und Co. geht.

