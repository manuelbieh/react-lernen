# Komponenten in React

## Die zwei Erscheinungsformen von React Components

Eine erste einfache **HelloWorld**-Komponente haben wir schon beim Sprung ins kalte Wasser implementiert. Jedoch war dies natürlich bewusst eine sehr simple Komponente, die nicht gerade sehr praxisnah war und auch längst nicht alles beinhaltet hat was uns React bietet. Aber sie diente als gute erste Veranschaulichung, um die grundsätzliche Funktionsweise von von **React** und **React-Komponenten** kennenzulernen.

Das Prinzip von **Komponenten** ist einfach erklärt: eine **Komponente** erlaubt es komplexe User Interfaces in einzelne kleine Stücke zu unterteilen. Diese sind im Idealfall wiederverwendbar, isoliert und in sich geschlossen. Sie verarbeiten beliebigen Input von außen in Form von sogenannter **Props** \(engl. für „Properties“, also Eigenschaften\) und beschreiben letztendlich anhand ihrer `render()`-Funktion was auf dem Bildschirm erscheint.

Komponenten können grob in zwei verschiedenen Varianten auftreten: in Form einer einfachen Funktion \(engl. **Function Component**\), sowie **Class Components**, die eine gewöhnliche ES2015-Klasse repräsentieren. Bis zur Einführung der React **Hooks** war es in **Function Components** nicht möglich einen lokalen State zu verwalten, daher stößt man mancherorts noch auf den Begriff **Stateless Functional Component**, der aus einer Zeit stammt, als **Function Components** noch keinen State halten konnten. Wie **Hooks** im Detail funktionieren und wie State seit React 16.8.0 nun auch in Function Components verwendet werden kann, wird im weiteren Verlauf noch ausführlich im Kapitel über **Hooks** erläutert.

### Function Components

Die deutlich einfachste Art um in React eine Komponente zu definieren ist sicherlich die **Function Component**, die, wie der Name es bereits andeutet, tatsächlich lediglich eine gewöhnliche JavaScript-Funktion ist:

```jsx
function Hello(props) {
  return <div>Hello {props.name}</div>
}
```

Diese Funktion erfüllt alle Kriterien einer gültigen **React-Komponente**: sie hat als `return`-Wert ein explizites `null` \(`undefined` ist dagegen **nicht** gültig!\) oder ein gültiges `React.Element` \(hier in Form von **JSX**\) und sie empfängt ein `props`-Objekt als erstes Funktionsargument, wobei sogar dieses optional ist und ebenfalls `null` sein kann.

###  Class Components / Klassen-Komponenten

Die zweite Möglichkeit wie eine **React-Komponente** erstellt werden kann habe ich im Eingangsbeispiel schon kurz gezeigt: **Class Components** oder auf gut Deutsch **Klassen-Komponenten**. Diese bestehen aus einer ES2015-Klasse, die von der `React.Component` oder `React.PureComponent`\(dazu später mehr\) Klasse ableitet und hat mindestens eine Methode mit dem Namen `render()`:

```jsx
class Hello extends React.Component {
  render() {
    return <div>Hello {this.props.name}</div>
  }
}
```

Wichtiger Unterschied hier: während eine **Function Component** ihre **Props** als Funktionsargumente übergeben bekommt, bekommt die `render()`-Methode einer **Klassen-Komponente** selbst keinerlei Argumente übergeben, sondern es kann einzig über die Instanz-Eigenschaft `this.props` auf die **Props** zugegriffen!

Die beiden obigen Komponenten resultieren hier in einer komplett identischen Ausgabe!

{% hint style="info" %}
Ein Kriterium das beide Arten von Komponenten gemeinsam haben ist, dass der `displayName`, also der Name einer gültigen Komponente stets mit einem **Großbuchstaben** anfängt. Der Rest des Namens kann aus Großbuchstaben oder Kleinbuchstaben bestehen, wichtig ist lediglich, dass der erste Buchstabe stets ein Großbuchstabe ist! 

Beginnt der Name einer Komponente mit einem Kleinbuchstaben, behandelt React diese stattdessen als reines DOM-Element. `section` würde React also als DOM-Element interpretieren, während eine eigene Komponente durchaus den Namen `Section` haben kann und wegen ihres Großbuchstabens am Anfang von React korrekt vom `section` DOM-Element unterschieden werden würde.
{% endhint %}

Wie wir innerhalb der **Komponenten** jeweils mit dem **State** arbeiten, diesen modifizieren und uns zu eigen machen ist sehr komplex, weswegen dem Thema ein eigenes Kapitel gewidmet ist. Dieses folgt direkt im Anschluss an dieses hier und ich würde empfehlen erst dieses Kapitel zu beenden um die Funktionsweise von Komponenten zu verstehen, bevor wir hier tiefer einsteigen.

### Sonderfall PureComponent

Eine Sonderform der **Class Component** stellt die **Pure Component** dar. Diese erbt von `React.PureComponent` und funktioniert vom Grundsatz her wie die gewöhnliche Klassen-Komponente, die von `React.Component` abgeleitet wird. Mit dem Unterschied, dass React diese Art der Komponente nur dann neu rendert wenn sich ihre **Props** oder ihr **State** im Vergleich zur vorherigen Render-Phase geändert haben. Sie kann zur Optimierung der Performance verwendet werden.

Die Klasse führt dazu einen oberflächlichen \(_„shallow“_\) Vergleich durch, schaut also lediglich ob die **Referenzen** noch identisch zum vorherigen Rendering sind und **nicht**, ob sich deren **Werte** selbst geändert haben. Dies bedeutet, dass eine **Pure Component** ggf. auch dann neu rendert wenn die Werte zwar identisch sind, sich die Referenzen jedoch geändert haben, etwa weil eine neue Arrow-Function erzeugt wurde.

Am besten lässt sich dies an einem kurzen Beispiel erklären:

```jsx
const logFunction = (message) => console.log(message);

class App extends React.Component {
  render() {
    return <MyComponent logger={changeHandler} />;
  }
}
```

Hier definieren wir eine Funktion `logFunction` außerhalb der Klasse. Die fiktive `MyComponent`-Komponente bekommt als `logger`-Prop hier die **Referenz** zur `logFunction` übergeben. Die **Identität** der **Referenz** bleibt hier bei jedem erneuten Rendering bestehen und eine **Pure Component** würde nicht erneut gerendert, sofern sich ihr eigener State nicht geändert hat, da die Props hier in der Rendering-Phase mit denen aus dem vorherigen Rendering exakt übereinstimmen, also identisch sind.

Im folgenden Beispiel hingegen, würde auch die **Pure Component** neu gerendert:

```jsx
class App extends React.Component {
  render() {
    return <MyComponent logger={(message) => console.log(message)} />;
  }
}
```

Zwar übergeben wir auch hier stets eine immer gleiche Funktion an die `MyComponent`, allerdings erzeugen wir für den Interpreter bei jedem neuen Rendering auch eine neue Funktionsidentität. Dadurch geht der Bezug zur alten Funktion verloren und die _Shallow-Comparison_, also der oberflächliche Vergleich zwischen den vorherigen und den neuen Props schlägt fehl, ist also unterschiedlich und löst damit ein Rerendering aus.

Dies gilt gleichermaßen auch für Objekte und Arrays:

```jsx
<MyComponent
  logConfig={{ logLevel: 'info' }}
  logEntries={['Nachricht 1', 'Nachricht 2']} />  
```

Hier würde ein Rendering ausgelöst, weil auch das `logConfig`-Objekt bzw. das `logEntries`-Array bei jedem erneuten Rendering an Ort und Stelle neu erzeugt werden würde.

### „Pure“ Function Components

**Klassen-Komponenten** leiten von den Klassen `Component` oder `PureComponent` ab und wir bestimmen durch die Auswahl von welcher Klasse wir ableiten, ob die Komponente mit jeder Änderung einer sich in der Hierarchie höher befindlichen Komponente neu gerendert werden soll. Diese Möglichkeit haben wir bei **Function Components**, die lediglich einfache JavaScript-Funktionen sind, nicht. Seit **React 16.6.0** bietet React jedoch durch die neue Wrapper-Funktion `React.memo()` auch in **Function Components** die Möglichkeit der Rerendering-Optimierung. Dazu wird diese einfach um die entsprechende Funktion gelegt:

```jsx
const MyComponent = React.memo((props) => {
  return <p>Ich werde nur neu gerendert wenn sich meine Props ändern</p>;
});
```

Die **Function Component** verhält sich dann so wie eine vergleichbare **Klassen-Komponente**, die von der `React.PureComponent` Klasse abgeleitet wird.

Wer neugierig geworden ist, kann sich für das einfachere Verständnis auch mit der folgenden Demo etwas austoben:

```jsx
import React from "react";
import ReactDOM from "react-dom";

class ClassComponent extends React.Component {
  render() {
    return <p>Class Component: {new Date().toISOString()}</p>;
  }
}

class PureClassComponent extends React.PureComponent {
  render() {
    return <p>Pure Class Component: {new Date().toISOString()}</p>;
  }
}

const FunctionComponent = () => {
  return <p>Function Component: {new Date().toISOString()}</p>;
};

const MemoizedFunctionComponent = React.memo(() => {
  return <p>Memoized Function Component: {new Date().toISOString()}</p>;
});

class App extends React.Component {
  state = {
    lastRender: new Date().toISOString(),
  };

  componentDidMount() {
    this.interval = setInterval(() => {
      this.setState({ lastRender: new Date().toISOString() });
    }, 200);
  }

  componentWillUnmount() {
    clearInterval(this.interval);
  }

  render() {
    return (
      <div>
        <p>App: {this.state.lastRender}</p>
        <ClassComponent />
        <PureClassComponent />
        <FunctionComponent />
        <MemoizedFunctionComponent />
      </div>
    );
  }
}

ReactDOM.render(<App />, document.getElementById("root"));
```

Hier löst die `App`-Komponente alle 0,2 Sekunden automatisch ein Rerendering aus, ohne das dabei jedoch **Props** an ihre Kind-Komponenten übergeben werden. Preisfrage: welche beiden Komponenten werden dabei jeweils mit neu gerendert und welche beiden nicht?

## Component Composition – mehrere Komponenten in einer

Bisher haben unsere Beispiel-Komponenten jeweils nur DOM-Elemente ausgegeben. **React-Komponenten** können aber auch andere React-Komponenten beinhalten. Wichtig hierbei ist nur, dass die Komponente sich im selben Scope befindet, also entweder direkt im gleichen Scope definiert wurde oder bei der Verwendung von CommonJS- oder ES-Modules ins aktuelle File importiert wurden mittels `require()` oder `import`. 

**Ein Beispiel:**

```jsx
function Hello(props) {
  return <div>Hallo {props.name}</div>;
}

function MyApp() {
  return (
    <div>
      <Hello name="Manuel" />
      <Hello name="Tom" />
    </div>
  );
}

ReactDOM.render(<MyApp />, document.getElementById('app'));
```

Die Komponente `<MyApp>` gibt hier ein `<div>` zurück, das zweimal die `Hello`-Komponente aus dem vorherigen Beispiel benutzt um Manuel und Tom zu begrüßen. Das Resultat:

```jsx
<div>
  <div>Hallo Manuel</div>
  <div>Hallo Tom</div>
</div>
```

Wichtig: eine Komponente darf stets nur ein einzelnes **Root-Element** zurückgeben! Dies kann sein:

* ein einzelnes React-Element:

```jsx
<Hello name="Manuel />
```

* Auch in verschachtelter Form, solange es nur ein einzelnes Element auf äußerer Ebene gibt:

```jsx
<Parent>
  <Child />
</Parent>
```

* ein DOM-Element \(auch dieses darf wiederum verschachtelt sein und andere Elemente beinhalten\):

```jsx
<div>…</div>
```

* … oder selbstschließend:

```jsx
<img src="logo.jpg" alt="Bild: Logo" />
```

* Oder auch

```javascript
null
```

… jedoch **nicht** `undefined`!

Seit **React 16.0.0** dürfen das außerdem auch sein:

* ein Array welches wiederum gültige return-Werte \(s.o.\) beinhaltet:

```jsx
[
  <div key="1">Hallo</div>, 
  <Hello key="2" name="Manuel" />
]
```

* ein simpler String:

```javascript
'Hallo Welt'
```

* oder ein sogenanntes „Fragment“ – eine Art spezielle „Komponente“, das selbst nicht im gerenderten Output auftaucht und als Container dienen kann, falls man andererseits gegen die Regel verstoßen würde nur ein Root-Element aus der Funktion zurückzugeben oder invalides HTML erzeugen würde:

```jsx
<React.Fragment>
  <li>1</li>
  <li>2</li>
  <li>3</li>
</React.Fragment>
```

Beim **Transpiling** mit **Babel ab Version 7** kann außerdem die `Fragment`-Kurzform benutzt werden die aus einem leeren öffnenden und schließenden Element besteht:

```jsx
<>
  <li>1</li>
  <li>2</li>
  <li>3</li>
</>
```

Komponenten können dabei beliebig zusammengesetzt \(_„composed“_\) werden. So bietet es sich oftmals an große und komplexe Komponenten in einzelne, kleinere und übersichtlichere Komponenten zu unterteilen um diese leichter verständlich und optimalerweise sogar auch wiederverwendbar zu machen. Dies ist oftmals ein lebender Prozess bei dem man ab einem gewissen Punkt bemerkt, dass eine Unterteilung in mehrere einzelne Komponenten möglicherweise sinnvoll wäre.

## Komponenten aufteilen – Übersicht bewahren

Werfen wir doch mal einen Blick auf eine beispielhafte Kopfleiste, die ein Logo, eine Navigation und eine Suchleiste enthält. Kein ganz unübliches Muster also schaut man sich gewöhnliche Web-Anwendungen an:

```jsx
function Header() {
  return (
    <header>
      <div className="logo">
        <img src="logo.jpg" alt="Image: Logo" />
      </div>
      <ul className="navigation">
        <li><a href="/">Homepage</a></li>
        <li><a href="/team">Team</a></li>
        <li><a href="/services">Services</a></li>
        <li><a href="/contact">Contact</a></li>
      </ul>
      <div className="searchbar">
          <form method="post" action="/search">
            <p>
              <label htmlFor="q">Suche:</label>
              <input type="text" id="q" name="q" />
            </p>
            <input type="submit" value="Suchen" />
          </form>
      </div>
    </header>
  );
}
```

Wir wissen bereits, dass Komponenten in React problemlos auch andere Komponenten beinhalten können und das diese Komponenten-basierte Arbeitsweise auch der Idee und dem Mindset von React entspricht. Was bietet sich hier also an? Richtig: wir teilen unsere doch bereits relativ große, unübersichtliche Komponente in mehrere kleinere Häppchen auf, die jeweils alle nur einen einzigen, ganz bestimmten Zweck erfüllen.

Da wäre das Logo, das wir sicherlich an anderer Stelle nochmal verwenden können. Die Navigation kann möglicherweise neben dem Header auch nochmal in einer Sitemap eingesetzt werden. Auch die Suchleiste soll vielleicht irgendwann mal nicht mehr nur im Header zum Einsatz kommen, sondern vielleicht auch auf der Suchergebnisseite selbst.

In Komponenten gesprochen, landen wir dann bei folgendem Endresultat:

```jsx
function Logo() {
  return (
    <div className="logo">
      <img src="logo.jpg" alt="Image: Logo" />
    </div>
  );
}

function Navigation() {
  return (
    <ul className="navigation">
      <li><a href="/">Homepage</a></li>
      <li><a href="/team">Team</a></li>
      <li><a href="/services">Services</a></li>
      <li><a href="/contact">Contact</a></li>
    </ul>
  );
}

function Searchbar() {
  return (
    <div className="searchbar">
      <form method="post" action="/search">
        <p>
          <label htmlFor="q">Suche:</label>
          <input type="text" id="q" name="q" />
        </p>
        <input type="submit" value="Suchen" />
      </form>
    </div>
  );
}

function Header() {
  return (
    <header>
      <Logo />
      <Navigation />
      <Searchbar />
    </header>
  );
}
```

Auch wenn der Code jetzt erstmal länger geworden ist haben wir uns dadurch dennoch einige große Vorteile geschaffen.

### Leichtere Kollaboration

Alle Komponenten können \(und sollten!\) in einem eigenen File gespeichert werden, was die Arbeit im Team immens erleichtert. So könnte jedes Team-Mitglied oder auch einzelne Teams innerhalb eines großen Projekt-Teams für eine oder mehrere Komponenten hauptverantwortlich sein \(„Ownership übernehmen“\) und Änderungen in diesen vornehmen, während das Risiko die Änderungen eines Kollegen zu überschreiben oder später Git Merge-Konflikte auflösen zu müssen immens sinkt. Teams werden zu _Konsumenten_ von Komponenten anderer Teams, die anhand eventuell verfügbarer Props ein simples Interface für ihre Komponente bereitstellen.

### Single Responsibility Prinzip

Wir haben nun außerdem „sprechende“ Komponenten, von denen jede eine klar definierte Aufgabe hat, die direkt anhand ihres Namens ersichtlich wird. Das Logo zeigt mir überall wo es verwendet wird dasselbe Logo an. Möchte ich später eine Änderung an der Suchleiste vornehmen, suche ich gezielt nach der Searchbar.js und ändere diese entsprechend meinen neuen Anforderungen. Die Header-Komponente dient als übergeordnete Komponente die selbst dafür verantwortlich ist, alle ihre Bestandteile zu beinhalten und diese überall hin mitzubringen, wo sie eingesetzt wird. 

### Wiederverwendbarkeit

Und nicht zuletzt haben wir ganz nebenbei noch Wiederverwendbarkeit geschaffen. Möchte ich wie erwähnt das Logo nicht nur im Header sondern auch im Footer verwenden hält mich natürlich nichts davon ab, dieselbe Komponente auch in meiner Footer-Komponente zu verwenden. Habe ich verschiedene Seitenbereiche mit unterschiedlichen Layouts, die jedoch alle denselben Header darstellen, kann ich dazu meine schlanke und übersichtliche Header-Komponente überall dort verwenden, wo ich ich ihn benötige.  Der Konsument einer Komponente muss dazu nicht einmal wissen aus welchen einzelnen Komponenten sie besteht. Es reicht, lediglich die gewünschte Komponente zu importieren da diese sich selbst um ihre Abhängigkeiten kümmert.

## Props – die „Datenempfänger“ einer Komponente

Nun habe ich bereits soviel über **Props** geschrieben. Höchste Zeit also einmal das Geheimnis zu lüften und genauer darauf einzugehen. Was sind also „Props“?

Durch die **Props** nehmen Komponenten beliebige Arten von Daten entgegen und können innerhalb der **Komponente** auf diese Daten zugreifen. Denken wir an unsere **funktionale Komponente** zurück, erinnern wir uns vielleicht, dass in diesem Fall die **Props** tatsächlich als ganz gewöhnliches Argument an die Funktion übergeben wurden. Ähnlich ist das Prinzip bei einer **Klassen-Komponente**, mit dem Unterschied, dass die **Props** über den **Constructor** der Klasse in die Komponente hereingereicht werden und über `this.props` innerhalb der Klassen-Instanz verfügbar sind, statt über ein Funktionsargument, wie das bei funktionalen Komponenten der Fall ist. Darum kümmert sich React beim Parsing der `createElement()`-Aufrufe.

Wichtig dabei ist: wann immer eine Komponente von außen neue **Props** hereingereicht bekommt, löst dies ein Rerendering der Komponente aus! Dieses Verhalten kann mittels der `shouldComponentUpdate()` **Lifecycle-Methode** explizit unterbunden werden, doch dazu gibt es im nachfolgenden Kapitel über **State und Lifecycle-Methoden** mehr. Wichtig ist erst einmal der allgemeine Grundsatz: empfängt eine **Komponente** von außen neue **Props**, veranlasst dies React dazu eine **Komponente** mitsamt ihrer Kind-Komponenten neu zu rendern.

### Props sind readonly innerhalb einer Komponente

Unabhängig davon wie die **Props** in welcher Art von Komponente auch immer landen, eines ist ihnen gemeinsam: sie sind innerhalb der Komponente **immer readonly**, dürfen \(und können\) also nur gelesen, nicht aber modifiziert werden! Der Kenner spricht hier auch von **Immutability** oder **Immutable Objects**. Um mit veränderlichen Daten zu arbeiten kommt später der React **State** ins Spiel. Aber eins nach dem anderen.

Modifiziert eine Funktion ihren Input nicht und hat auch keine Abhängigkeit nach außen, so spricht man in der funktionalen Programmierung von einer puren Funktion \(engl: **Pure Function**\) und die Idee dahinter ist recht simpel: so soll sichergestellt werden, dass eine Funktion in sich geschlossen ist, daher davon unbeeindruckt bleibt wenn sich außerhalb der Funktion etwas ändert, die Funktion bekommt alle benötigten Parameter hereingereicht, ist frei von Seiteneffekten \(engl: **Side Effects**\) und erzielt somit bei gleichen Eingabewerten auch immer die exakt identische Ausgabe. **Gleicher Input, gleicher Output!**

Mit anderen Worten: egal welche Variablen außerhalb der Funktion ihren Wert ändern, egal wie oft andere Funktionen anderswo aufgerufen werden: bekommt eine **Pure Function** die gleichen Parameter wie zuvor, gibt sie mir auch das gleiche Ergebnis wie zuvor zurück. Immer und ausnahmslos.

Warum ist das wichtig? Nun, React verfolgt bei seinen Komponenten das Prinzip von **Pure Functions**. Erhält eine Komponente die gleichen Props von außen hineingereicht und ist der State der Komponente der gleiche, sollte auch der Output immer identisch sein.

### Pure Functions im Detail

Da das Prinzip von **Pure Functions** ein grundlegendes ist bei der Arbeit mit React möchte ich diese Anhand einiger Beispiele etwas näher beleuchten. Hier geht es überwiegend um Theorie, die sich sicherlich komplizierter anhört als das später bei der praktischen Arbeit mit React der Fall sein wird. Dennoch möchte ich diese zum besseren Verständnis nicht unerwähnt lassen.

#### Beispiel für eine simple „Pure Function“

```javascript
function pureDouble(number) {
  return number * 2; 
}
```

Unsere erste simple Funktion bekommt eine Nummer übergeben, verdoppelt diese und gibt das Ergebnis zurück. Egal ob ich die Funktion 1, 10 oder 250 mal aufrufe: übergebe ich der Funktion bspw. eine `5` als Wert, erhalte ich eine `10` zurück. Immer und ausnahmslos. Same input, same output.

#### Beispiel für eine „Impure Function“

```javascript
function impureCalculation(number) {
  return number + window.outerWidth;
}
```

Die zweite Funktion ist nicht mehr _pure_, weil sie nicht zuverlässig immer den gleichen Output liefert, selbst wenn ihr Input identisch ist. Momentan ist mein Browser-Fenster 1920 Pixel breit. Rufe ich die Funktion mit `10` als Argument auf, erhalte ich `1930` zurück \(`10 + 1920`\). Verkleinere ich nun das Fenster auf 1280 Pixel und rufe die Funktion erneut, mit exakt der gleichen `10` als Argument auf bekomme ich dennoch ein anderes Ergebnis \(`1290`\) als beim ersten Mal. Es handelt sich also nicht um eine **Pure Function**.

Eine Möglichkeit diese Funktion „pure“ zu machen wäre, ihr meine Fensterbreite als weiteres Funktionsargument zu übergeben:

```javascript
function pureCalculation(number, outerWidth) {
  return number + outerWidth;
}
```

So liefert die Funktion beim Aufruf von `pureCalculation(10, window.outerWidth)` zwar immer noch ein Ergebnis was von meiner Fensterbreite abhängt, die Funktion ist dennoch _„pure“_ da sie beim gleichen Input weiterhin den gleichen Output liefert. Einfacher kann man das nachvollziehen wenn man die Funktion mal auf ihre wesentlichen Eigenschaften reduziert:

```javascript
function pureSum(number1, number2) {
  return number1 + number2;
};
```

**Gleicher Input, Gleicher Output.**

#### Weiteres Beispiel für eine Impure Function

Stellen wir uns einmal vor wir möchten eine Funktion implementieren die als Input ein Objekt empfängt mit Parametern zu einem Auto.

```javascript
var car = {speed: 0, seats: 5};
function accelerate(car) {
  car.speed += 1;
  return car;
}
```

Das obige Beispiel ist ebenfalls eine Funktion die nicht „pure“ ist, da sie ihren Eingabewert modifiziert und somit beim zweiten Aufruf bereits ein anderes Ergebnis als Ausgabewert hat als noch beim ersten Aufruf:

```javascript
console.log(accelerate(car)) 
// {speed: 1, seats: 5}

console.log(accelerate(car)) 
// {speed: 2, seats: 5}
```

Wie sorgen wir also nun dafür, dass auch unser letztes Beispiel „pure“ wird? Indem wir den Eingabewert nicht mehr modifizieren und stattdessen jedes Mal ein neues Objekt erzeugen, basierend auf dem Eingabewert, und dieses neue Objekt aus der Funktion zurückgebend,

```javascript
var car = {speed: 0};
function accelerate(car) {
  return {
    speed: car.speed + 1,
  }
}
```

Neues Ergebnis:

```javascript
console.log(accelerate(car)) 
// {speed: 1}

console.log(accelerate(car)) 
// {speed: 1}
```

**Gleicher Input: gleicher Output:** wir sind „pure“!

Ihr wundert euch jetzt vielleicht warum ich euch das erzähle und hier mit langweiliger Theorie nerve, wo ihr doch eigentlich nur React lernen wollt \(jedenfalls würde ich mir das an dieser Stelle denken, wenn ich mir vorstelle dieses Buch auch aus diesem Grund zu lesen\).

React ist eine sehr liberale Library, die dem Entwickler sehr viele Freiheiten lässt. Aber eine Sache ist oberstes Gebot und da kennt React auch wirklich keinen Spaß: **Komponenten müssen sich im Hinblick auf ihre Props wie „Pure Functions“ verhalten und bei gleichen Props stets die gleiche Ausgabe erzeugen!**

Haltet ihr euch da nicht dran, kann es bei der Arbeit mit React zu sehr eigenartigen Effekten kommen, zu unerwünschten und nicht nachvollziehbaren Ergebnissen führen und euch das Leben beim Bugfixing zur Hölle machen. Und genau aus diesem Grund lernt ihr ja React: weil ihr ein einfaches aber dennoch zugleich unglaublich mächtiges Tool haben wollt, mit denen ihr nach etwas Einarbeitung in unglaublich schneller Zeit wirklich professionelle User Interfaces entwickeln könnt, ohne euch dabei selbst in den Wahnsinn zu treiben. All das bietet euch React, solange ihr euch an diese Regel haltet!

Das hat für uns aber gleichzeitig den sehr angenehmen Nebeneffekt, dass sich Komponenten in der Regel auch sehr einfach testen lassen.

So und was bedeutet jetzt genau das _„readonly innerhalb einer Komponente“_? Das ist mit unserem neuen Wissen über „Pure Functions“ recht schnell erklärt: egal wie ich in der Komponente auf die Props zugreife, ob direkt über das `props`-Argument einer **Function Component**, über den `constructor()` in einer **Klassen-Komponente** oder an beliebiger anderer Stelle innerhalb einer **Klassen-Komponente** mittels `this.props`: ich kann und darf \(und soll! und will!!\) den Wert der hereingereichten **Props** nicht ändern.

Anders sieht das natürlich **außerhalb** von Komponenten aus. Hier kann ich den Wert problemlos ändern \(vorausgesetzt natürlich, wir befinden uns nicht wiederum in einer Komponente, welche die Prop die wir modifizieren wollen selbst nur hereingereicht bekommen hat\).

#### Was nicht möglich ist

```jsx
function Example(props) {
  props.number = props.number + 1;
  props.fullName = [props.firstName, props.lastName].join(' ');
  return (
    <div>({props.number}) {props.fullName} </div>
  );
}

ReactDOM.render(
  <Example number={5} firstName="Manuel" lastName="Bieh" />,
  document.getElementById('app')
);
```

**Ausgabe:**

{% hint style="danger" %}
 TypeError: Cannot add property number, object is not extensible
{% endhint %}

Hier versuche ich direkt die `number` und `fullName` Props innerhalb meiner `Example`-Komponente zu ändern, was natürlich nicht funktionieren kann, da wir ja gelernt haben, dass Props grundsätzlich **readonly** sind.

#### Was allerdings möglich ist

Manchmal möchte ich aber eben doch einen neuen Wert von einer hereingereichten Prop ableiten. Das ist auch gar kein Problem, React 16.3.0 bietet dafür sogar noch eine umfassende Funktion `getDerivedStateFromProps()`, auf die ich im nächsten Kapital nochmal gesondert und sehr detailliert eingehen werde.

Möchte ich aber eben nur mal eben einen Wert anzeigen der sich von der Prop ableitet, die ich als Komponente hereingereicht bekomme, geht das indem nur die Ausgabe auf Basis der Prop anpasse ohne zurück zu schreiben.

```jsx
var React = require('react');
var ReactDOM = require('react-dom');

function Example(props) {
  return (
    <div>({props.number + 1}) {[props.firstName, props.lastName].join(' ')}</div>
  );
}

ReactDOM.render(
  <Example number={5} firstName="Manuel" lastName="Bieh" />,
  document.getElementById('app')
);
```

**Ausgabe:**

```jsx
<div>(6) Manuel Bieh</div>
```

In diesem Fall modifiziere ich also lediglich die Ausgabe basierend auf den `props`, nicht jedoch das `props`-Objekt selbst. Das ist überhaupt kein Problem.

#### Was ebenfalls möglich ist

Jetzt bleibt noch abschließend zu klären wie Props denn nun außerhalb einer Komponente geändert werden können, denn bisher war immer nur die Rede davon, dass Props nur innerhalb einer Komponente nicht verändert werden dürfen.

Auch das lässt sich am Besten anhand eines konkreten, allerdings noch recht abstrakten Beispiels erklären:

```jsx
var React = require('react');
var ReactDOM = require('react-dom');

var renderCounter = 0;
setInterval(function () {
  renderCounter++;
  renderApp();
}, 2000);

const App = (props) => {
  return <div>{props.renderCounter}</div>
};

function renderApp() {
  ReactDOM.render(
    <App renderCounter={renderCounter} />,
    document.getElementById('app')
  );
}

renderApp();
```

Was passiert hier? Zunächst einmal setzen wir eine Variable `renderCounter` auf den Anfangswert `0`. Diese Variable zählt für uns gleich mit wie oft wir unsere `App`-Komponente rendern oder genauer gesagt, wie oft wir im Endeffekt die `ReactDOM.render()` Funktion aufrufen, die dann entsprechend bei jedem Aufruf dafür sorgt, dass die `App`-Komponente erneut gerendert wird.

Anschließend starten wir einen Intervall, der die besagte Funktion regelmäßig alle 2000 Millisekunden ausführt. Dabei führt der Intervall nicht nur im 2 Sekunden-Takt die Funktion aus, sondern zählt auch gleichzeitig unsere `renderCounter` Variable um 1 hoch. Was hier jetzt passiert ist ganz spannend: wir modifizieren die `renderCounter` Prop unserer App **„von außen“**.

Die Komponente selbst bleibt dabei komplett „pure“. Wird sie aufgerufen mit:

```jsx
<App renderCounter={5} />
```

gibt sie uns als Ergebnis zurück:

```markup
<div>5</div>
```

Und zwar egal wie oft die Komponente inzwischen tatsächlich gerendert wurde. **Gleicher Input, gleicher Output.**

Innerhalb unserer Komponente sind und bleiben wir weiterhin _pure_. Wir modifizieren den Eingabewert nicht und wir haben in der Komponente auch keinerlei direkten Abhängigkeiten nach außen, die unser Rendering-Ergebnis beeinflussen könnten. Der Wert wird lediglich außerhalb unserer Komponente geändert und neu in die Komponente **hereingegeben**, was uns aber an dieser Stelle auch gar nicht weiter interessieren braucht, da es für uns lediglich wichtig ist, dass unsere Komponente mit gleichen Props auch weiterhin das gleiche Ergebnis liefert. Und das ist hier zweifellos gegeben. Wer die Props außerhalb unserer Komponente modifiziert, wie oft und in welcher Form ist uns ganz gleich, solange wir das nicht selber innerhalb unserer Komponente tun. Okay, Prinzip verstanden?

#### Props sind ein abstrahiertes Funktionsargument

Da Props, reduziert man sie auf das Wesentliche, nichts anderes als ein Funktionsargument sind, können sie auch in dessen diversen Formen auftreten. Alles, was auch Functions oder Constructors in JavaScript als Argument akzeptieren, kann auch als Wert für eine Prop verwendet werden. Vom simplen String, über Objekte, Funktionen oder gar andere React-Elemente \(die ja, wie wir bereits wissen, hinter den Kulissen auch nichts anderes als ein Aufruf von `createElement()` sind\) kann das nahezu alles sein, solange es eben ein valider Ausdruck ist.

```jsx
<MyComponent
  counter={3}
  text="example"
  showStatus={true}
  config={{ uppercase: true }}
  biggerNumber={Math.max(27, 35)}
  arbitraryNumbers={[1, 4, 28, 347, 1538]}
  dateObject={Date}
  dateInstance={new Date()}
  icon={
    <svg x="0px" y="0px" width="32px" height="32px">
      <circle fill="#CC3300" cx="16" cy="16" r="16" />
    </svg>
  }
  callMe={() => {
    console.log('Somebody called me');
  }}
/>
```

Auch wenn die meisten Props hier inhaltlich wenig Sinn ergeben und nur zur Veranschaulichung dienen, so sind sie dennoch syntaktisch korrektes JSX, demonstrieren wie mächtig sie sind und in welchen verschiedenen Formen sie auftreten können.

### Props sind nicht auf eine Verschachtelungsebene beschränkt

Eine Komponente die Props empfängt kann diese problemlos auch an Kind-Elemente weiterreichen. Dies kann einerseits hilfreich sein wenn man große Komponenten in mehrere kleinere Komponenten unterteilt und gewisse Props an Kind-Komponenten weitergegeben werden müssen, kann aber bei komplexen Anwendungen teilweise dazu führen dass es schwer erkenntlich wird wo der genaue Ursprung einer Prop ist und wo ich anfangen muss zu suchen, wenn ich den Wert einer Prop ändern möchte.

```jsx
function User(props) {
  return (
    <div>
      <h1>{props.name}</h1>
      <UserImage image={props.image} />
      <ListOfPosts items={props.posts} />
    </div>
  )
}

ReactDOM.render(
  <User name={user.name} image={user.image} posts={user.posts} />,
  document.getElementById('app')
);
```

### Die wichtigsten Punkte im Überblick

{% hint style="info" %}
Komponenten müssen sich hinsichtlich ihrer Props als **Pure Functions** verhalten und bei gleichen Props stets die gleiche Ausgabe erzeugen.

* Props sind innerhalb einer Komponente grundsätzlich als **readonly** zu betrachten
* Komponenten können eine **beliebige Menge an Props** übergeben bekommen.
* In JSX übergibt man Props in ähnlicher Form wie in HTML Attribute
* Anders als in HTML, sind in JSX diverse Arten von Werten erlaubt. Werte die nicht vom Typ String sind, werden dabei in **geschweifte Klammern** gefasst
* Props können **sämtliche JavaScript-Ausdrücke** \(„Expressions“\) als Wert entgegennehmen
* Empfangene Props können beliebig viele Ebenen tief im Komponenten-Baum an Kind-Elemente weitergegeben werden
{% endhint %}



