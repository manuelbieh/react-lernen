# Komponenten in React

## Die zwei Erscheinungsformen von React Components

Eine erste einfache **HelloWorld**-Komponente haben wir schon beim Sprung ins kalte Wasser implementiert. Jedoch war dies natürlich eine sehr simple Komponente, die nicht gerade sehr praxisnah war und auch längst nicht alles beinhaltet hat was uns React bietet und diente lediglich zur ersten Anschauung dienen sollte, um die grundsätzliche Funktionsweise von von React-Komponenten kennenzulernen.

Das Prinzip von Komponenten ist einfach erklärt: eine Komponente erlaubt es komplexe User Interfaces in einzelne kleine Stücke zu unterteilen. Diese sind im Idealfall wiederverwendbar, isoliert und in sich geschlossen. Sie verarbeiten beliebigen Input von außen in Form von Props \(engl. für „Properties“, also Eigenschaften\) und beschreiben letztendlich anhand ihrer `render()`-Funktion was auf dem Bildschirm erscheint.

Komponenten können grob in zwei verschiedenen Varianten auftreten: rein funktionale Komponenten \(engl. **Functional Component**\), auch **Stateless Functional Component** \(_SFC_\) genannt, sowie **Class Components**, die eine gewöhnliche standard ES2015-Klasse repräsentieren.

Die deutlich einfachste Art um in React eine Komponente zu definieren ist sicherlich die funktionale Komponente, die, wie der Name es bereits andeutet, tatsächlich lediglich eine einfache JavaScript-Funktion ist:

```jsx
function Hello(props) {
  return <div>Hello {props.name}</div>
}
```

Diese Funktion erfüllt alle Kriterien einer gültigen React-Komponente: sie hat als `return`-Wert ein explizites `null` \(`undefined` ist dagegen **nicht** gültig!\) oder ein gültiges `React.Element` \(hier in Form von JSX\) und sie empfängt ein `props`-Objekt als erstes und einziges Funktionsargument, wobei sogar dieses optional ist und ebenfalls `null` sein kann.

Die zweite Möglichkeit wie eine React-Komponente erstellt werden kann habe ich im Eingangsbeispiel schon kurz gezeigt. sie besteht aus einer ES2015-Klasse, die von der `React.Component`-Klasse ableitet und hat mindestens eine Methode mit dem Namen `render()`:

```jsx
class Hello extends React.Component {
  render() {
    return <div>Hello {this.props.name}</div>
  }
}
```

Wichtiger Unterschied hier: während eine funktionale Komponente die Props einer Komponente als Funktionsargumente übergeben bekommt, bekommt die `render()`-Methode einer Klassen-Komponente selbst keinerlei Argumente übergeben, sondern es wird allein über die Instanz-Eigenschaft `this.props` auf die Props zugegriffen!

Die beiden obigen Komponenten resultieren hier in einer komplett identischen Ausgabe!

{% hint style="info" %}
Ein Kriterium das beide Arten von Komponenten gemeinsam haben ist, dass der `displayName`, also der Name einer gültigen Komponente stets mit einem **Großbuchstaben** anfängt. Der Rest des Namens kann aus Großbuchstaben oder Kleinbuchstaben bestehen, wichtig ist lediglich, dass der erste Buchstabe stets ein Großbuchstabe ist! 

Beginnt der Name einer Komponente mit einem Kleinbuchstaben, behandelt React diese stattdessen als reines DOM-Element. `section` würde React also als DOM-Element interpretieren, während eine eigene Komponente durchaus den Namen `Section` haben kann und wegen ihres Großbuchstabens am Anfang von React korrekt vom `section` DOM-Element unterschieden werden würde.
{% endhint %}

## Component Composition – mehrere Komponenten in einer

Bisher haben unsere Beispiel-Komponenten jeweils nur DOM-Elemente ausgegeben. React-Komponenten können aber auch andere React-Komponenten beinhalten. Wichtig hierbei ist nur, dass die Komponente sich im selben Scope befindet, also entweder direkt im gleichen Scope definiert wurde oder bei der Verwendung von CommonJS- oder ES-Modules ins aktuelle File importiert wurden mittels `require()` oder `import`. 

Ein Beispiel:

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

ReactDOM.render(
  <MyApp />, 
  document.getElementById('app')
);

```

Die Komponente `<MyApp>` gibt hier ein `<div>` zurück, das zweimal die Hello-Komponente aus dem vorherigen Beispiel benutzt um Manuel und Tom zu begrüßen. Das Resultat:

```jsx
<div>
  <div>Hallo Manuel</div>
  <div>Hallo Tom</div>
</div>
```

Wichtig: eine Komponente darf stets nur ein einzelnes Root-Element zurückgeben! Dies kann sein:

* ein einzelnes React-Element: `<Hello name="Manuel" />` 
* Auch in verschachtelter Form, solange es nur ein einzelnes Element auf äußerer Ebene gibt: `<Parent><Child /></Parent>` 
* ein DOM-Element \(auch dieses darf wiederum verschachtelt sein und andere Elemente beinhalten\): `<div>…</div>` 
* Oder selbstschließend: `<img src="logo.jpg" alt="Bild: Logo" />` 
* `null` \(aber niemals `undefined`!\)

Seit React 16 dürfen das außerdem auch sein:

* ein Array welches wiederum gültige return-Werte \(s.o.\) beinhaltet: `[<div key="1">Hallo</div>, <Hello key="2" name="Manuel" />]` 
* ein String: `'Hallo Welt'` 
* Ein sogenanntes „Fragment“ – eine Art spezielle „Komponente“, das selbst nicht im gerenderten Output auftaucht und als Container dienen kann, falls man andererseits gegen die Regel verstoßen würde nur ein Root-Element aus der Funktion zurückzugeben oder invalides HTML erzeugen würde: `<React.Fragment><li>1</li><li>2</li><li>3</li></React.Fragment>`

Komponenten können dabei beliebig zusammengesetzt \(_„composed“_\) werden. So bietet es sich oftmals an große und komplexe Komponenten in einzelne, kleinere und übersichtlichere Komponenten zu unterteilen um diese leichter verständlich und optimalerweise sogar auch wiederverwendbar zu machen. Dies ist oftmals ein lebender Prozess bei dem man ab einem gewissen Punkt bemerkt, dass eine Unterteilung in mehrere einzelne Komponenten möglicherweise sinnvoll wäre.

## Komponenten aufteilen – Übersicht bewahren

Werfen wir doch mal einen Blick auf eine beispielhafte Kopfleiste, die ein Logo, eine Navigation und eine Suchleiste enthält. Kein ganz unübliches Muster also schaut man sich Web-Anwendungen an:

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

## Props – die „Datenträger“ einer Komponente

Nun habe ich bereits soviel über **Props** geschrieben. Höchste Zeit also einmal das Geheimnis zu lüften und genauer darauf einzugehen. Was sind also „Props“?

Durch die Props nehmen Komponenten beliebige Arten von Daten entgegen und können innerhalb der Komponente auf diese Daten zugreifen. Denken wir an unsere funktionale Komponente zurück, erinnern wir uns vielleicht, dass in diesem Fall die Props tatsächlich als ganz gewöhnliches Argument an die Funktion übergeben wurden. Ähnlich ist das Prinzip bei einer Class Component, mit dem Unterschied, dass die Props über den Constructor der Klasse in die Komponente hereingereicht werden und über this.props innerhalb der Klassen-Instanz verfügbar sind, statt über ein Funktionsargument, wie das bei funktionalen Komponenten der Fall ist.

### Props sind readonly innerhalb einer Komponente

Unabhängig davon wie die Props in welcher Art von Komponente auch immer landen, eines ist ihnen gemeinsam: sie sind innerhalb der Komponente immer readonly, dürfen also nur gelesen, nicht aber modifiziert werden!

Hat eine Funktion dann auch keine Abhängigkeit nach außen, so spricht man in der funktionalen Programmierung von einer puren Funktion \(engl: **pure function**\) und die Idee dahinter ist recht simpel: so soll sichergestellt werden, dass eine Komponente in sich geschlossen ist, daher davon unbeeindruckt bleibt wenn sich außerhalb der Komponente etwas ändert, die Komponente bekommt alle benötigten Parameter hereingereicht, ist frei von Seiteneffekten \(eng: **side effects**\) und erzielt somit im Render-Prozess mit den gleichen Props auch immer die exakt identische Ausgabe. **Gleicher Input, gleicher Output!**

Mit anderen Worten: egal welche Variablen ihren Wert ändern, egal wie oft andere Funktionen aufgerufen werden oder ein Benutzer in der App herum klickt: bekommt meine „Pure Function“ die gleichen Parameter wie zuvor, gibt sie mir auch das gleiche Ergebnis wie vorher zurück. Immer und ausnahmslos.  


