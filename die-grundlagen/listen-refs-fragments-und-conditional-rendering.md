# Listen, Fragments und Conditional Rendering

Bis hierher habt ihr schon eine ganze Menge über React erfahren. Ihr wisst, wofür die **Props** sind, was der **State** ist und wie er sich von den **Props** unterscheidet, ihr wisst wie eine React-Komponente implementiert wird, was der Unterschied einer React-**Komponente** und einem React-**Element** ist und wie ihr mit **JSX** einen Elementenbaum beschreibt, der später in eurer Anwendung gerendert wird. **Lifecycle-Methoden** helfen euch, auf Änderungen eurer Daten zu reagieren. Damit habt ihr auch schon alles beisammen um eine simple React-Anwendung zu entwickeln. 

Allerdings gibt es noch einige Details, die in den vorherigen Kapiteln bisher gar keine Erwähnung fanden oder ohne weitere Erklärung in Beispielen benutzt wurden, die aber mit steigender Komplexität eurer Anwendung zunehmend relevanter werden.

Im Speziellen betrifft das die Arbeit mit **Listen**, also Arrays mit Daten, sogenannte **Refs**, damit sind Referenzen zu DOM-Repräsentationen von React-Elementen gemeint, **Fragments**, eine spezielle Art Komponente, die keine Spuren im gerenderten Output hinterlässt und **Conditional Rendering**, also Unterscheidungsmöglichkeiten, wann ihr was rendert, basierend auf **Props** und **State**.

Die Themen haben eins gemeinsam: sie sind zu wichtig um sie nicht im Grundlagenteil dieses Buchs zu erwähnen aber gleichzeitig nicht umfänglich genug, um ihnen jeweils ein komplettes eigenes Kapitel zu widmen.

## Listen

Mit Listen sind hier tatsächlich stumpfe JavaScript-Arrays gemeint, also einfache **Daten**, durch die iteriert werden kann. Sie sind bei der Arbeit \(nicht nur mit React\) alltäglich und keine Anwendung kommt ohne sie aus. **ES2015+** bietet uns mit `Array.map()`, `Array.filter()` oder `Array.find()` schöne deklarative Methoden, die wir als Ausdrücke in JSX innerhalb von geschweiften Klammern `{}` nutzen können.

Welche Rolle Ausdrücke in JSX spielen und wie wir Ausdrücke in JSX nutzen können, habe ich bereits im Kapitel über JSX angesprochen. Kurz aufgefrischt: Arrays können als Ausdruck in JavaScript genutzt werden und somit auch in JSX. Das heißt sie können in geschweiften Klammern stehen und werden dann beim Transpiling von JSX als Child-Node behandelt.

Das ist aber noch nicht alles, denn `Array.map()` kann bspw. modifizierte Items zurückgeben, die selbst wiederum JSX beinhalten. Das ist insofern praktisch, als es uns weitere Flexibilität verschafft und es uns ermöglicht, Datensammlungen in React-Elemente zu verwandeln.

Nehmen wir als Beispiel an, wir wollen eine Liste aus Cryptocurrencies anzeigen. Unser Array mit den entsprechenden Daten hat die folgende Form:

```jsx
const cryptos = [
    {
        id: 1,
        name: 'Bitcoin',
        symbol: 'BTC',
        quotes: { EUR: { price: 7179.92084586 } }
    },
    {
        id: 2,
        name: 'Ethereum',
        symbol: 'ETH',
        quotes: { EUR: { price: 595.218568203 } }
    },
    {
        id: 3,
        name: 'Litecoin',
        symbol: 'LTC',
        quotes: { EUR: { price: 117.690716234 } }
    }
];
```

Dargestellt werden sollen die Daten erst einmal als einfache ungeordnete Liste in simplem HTML. Die entsprechende Komponente könnte dann zum Beispiel so aussehen:

```jsx
const CryptoList = ({ currencies }) => (
  <ul>
    {currencies.map((currency) => (
      <li>
        <h1>{currency.name} ({currency.symbol})</h1>
        <p>{currency.quotes.EUR.price.toFixed(2)} €</p>
      </li>
    ))}
  </ul>
);
```

Und würde dann etwa so benutzt werden:

```jsx
<CryptoList currencies={cryptos} />
```

Heraus kommt eine Liste mit den entsprechenden Kryptowährungen und ihrem jeweiligen Preis. Allerdings bekommen wir auch direkt eine Warnung von React an den Kopf geworfen:

![Fehlermeldung bei fehlender key-Prop in einer durch einen Iterator erzeugten Liste](../.gitbook/assets/react-missing-key.png)

React erwartet bei allen Arrays und von einem Iterator zurückgegebenen Werten eine `key`-Prop. Diese dient dazu, dem **Reconciler** \(also dem React-Vergleichsalgorithmus\) eine Möglichkeit zu geben, um Listen-Elemente zu identifizieren und letztendlich vergleichen zu können. Der **Reconciler** erkennt dadurch, welche Array-Elemente hinzugefügt, entfernt oder modifiziert wurden. Die `key`-Prop nimmt dabei die Funktion einer eindeutigen ID ein und muss **innerhalb dieses Arrays einmalig** sein. In der Praxis wird hier typischerweise die ID eines Datensatzes verwendet. 

In unserem Fall haben wir eine solche ID vorliegen; das oberste Element, das aus der `map()`-Methode zurückgegeben wird, würde also korrekt so aussehen:

```jsx
const CryptoList = ({ currencies }) => (
    <ul>
        {currencies.map((currency) => (
            <li key={currency.id}>
                <h1>{currency.name} ({currency.symbol})</h1>
                <p>{currency.quotes.EUR.price.toFixed(2)} €</p>
            </li>
        ))}
    </ul>
);
```

Der Key muss dabei nur **innerhalb eines Arrays/Iterators inmitten seiner Geschwister-Elemente einmalig sein, nicht innerhalb der Komponente!** Dies bedeutet, dass wir die gleiche `CryptoList`-Komponente mit denselben Keys an anderer Stelle, auch in der gleichen Komponente, problemlos noch ein zweites Mal verwenden könnten. Nur eben nicht innerhalb dieses einen Loops. 

Sind zu einer Liste aus Datensätzen keine eindeutigen Schlüssel vorhanden, kann als letzter Ausweg der **Index** des Array-Elements verwendet werden. Davon wird jedoch **ausdrücklich abgeraten**, da dies zu Problemen bei der Performance sowie zu unvorhersehbarem Verhalten beim Rendering des User Interfaces führen kann.

Wichtig ist außerdem, dass die `key`-Prop immer **direkt in der von der Iterator-Funktion** zurückgegebenen **Toplevel-Komponente** oder dem **Array-Element** vorhanden sein muss, nicht in der von dieser Komponente zurückgegebenen JSX. 

Um besser zu veranschaulichen was das genau bedeutet, machen wir aus unserem obigen Listen-Element eine eigene kleine `CryptoListItem`-Komponente:

```jsx
const CryptoListItem = ({ name, symbol, quotes }) => (
  <li>
    <h1>{name} ({symbol})</h1>
    <p>{quotes.EUR.price.toFixed(2)} €</p>
  </li>
);
```

Was fällt auf? Richtig: die `key`-Prop, die wir zuvor hinzugefügt haben ist nun nicht mehr da. Unser `map()`-Aufruf würde sich dafür wie folgt verändern:

```jsx
const CryptoList = ({ currencies }) => (
  <ul>
    {currencies.map((currency) => (
      <CryptoListItem
        key={currency.id}
        name={currency.name}
        symbol={currency.symbol}
        quotes={currency.quotes}
      />
    ))}
  </ul>
);
```

Obwohl es das `<li></li>`-Element ist, welches letztendlich gerendert wird, muss dennoch die `<CryptoListItem />`-Komponente die `key`-Prop bekommen, da sie es ist, die von `Array.map()` an der entsprechenden Stelle im JSX zurückgegeben wird. 

Offtopic: die `CryptoList`-Komponente könnte durch Verwendung der **Object-Spread Syntax** weiter vereinfacht werden:

```jsx
const CryptoList = ({ currencies }) => (
  <ul>
    {currencies.map((currency) => <CryptoListItem key={currency.id} {...currency} />)}
  </ul>
);
```

Auf diese Art werden alle Eigenschaften des `currency`-Objekts entsprechend als gleichnamige Props an die `CryptoListItem`-Komponente übertragen.

Bei der direkten Arbeit mit Arrays, ohne einen Iterator wie `Array.map()` sähe das analog dazu so aus:

```jsx
const MyList = () => (
  <ul>
  {[
    <li key="1">One</li>,
    <li key="2">Two</li>,
    <li key="3">Three</li>
  ]}
  </ul>
);
```

## Fragments

Fragments sind eine Art Spezial-Komponente und dienen hilfsweise dazu, gültiges JSX zu erzeugen, ohne dabei sichtbare Spuren in der gerenderten Ausgabe zu hinterlassen. Gültiges JSX in dem Sinne, dass die `render()`-Methode immer nur ein Element auf oberster Ebene zurückgeben darf. Also etwa:

```jsx
render() {
  return (
    <ul>
      <li>Bullet Point 1</li>
      <li>Bullet Point 2</li>
      <li>Bullet Point 3</li>
    </ul>
  );
}
```

Aber eben nicht so etwas wie:

```jsx
render() {
  return (
    <li>Bullet Point 1</li>
    <li>Bullet Point 2</li>
    <li>Bullet Point 3</li>
  );
}
```

Hier geben wir aus der `render()`-Methode direkt und ohne umschließendes Eltern-Element mehrere `li`-Elemente zurück, was zu einer Fehlermeldung führt. Manchmal ist dies aber notwendig, bspw. wenn sich das umschließende Element in einer Eltern-Komponente befinden, die Kind-Elemente aber durch eine eigene Komponente erzeugt werden soll. 

Innerhalb einiger Elemente \(`table`, `ul`, `ol`, `dl`, …\) ist es aber nicht erlaubt, bspw. ein `div`-Element als Zwischenebene zu verwenden, um die Regel zu erfüllen stets nur ein einzelnes Root-Element aus einer Komponente zurückzugeben. In diesem Fall kommt das **Fragment** ins Spiel und würde angewendet auf das obige Beispiel folgende Änderung bedeuten, um valides JSX zu erzeugen:

```jsx
render() {
  return (
    <React.Fragment>
      <li>Bullet Point 1</li>
      <li>Bullet Point 2</li>
      <li>Bullet Point 3</li>
    </React.Fragment>
  );
}
```

Dabei gilt auch hier die Regel, dass eine iterativ, also durch eine Schleife erzeugte Ausgabe eine `key`-Prop besitzen muss. Mit der `Fragment`-Komponente ist dies möglich. Schauen wir uns ein weiteres, etwas umfassenderes und praxisnäheres Beispiel an:

```jsx
import React from 'react';
import ReactDOM from 'react-dom';

const TicketMeta = ({ metaData }) => (
  <dl>
    {Object.entries(metaData).map(([property, value]) => (
      <React.Fragment key={property}>
        <dt>{property}</dt>
        <dd>{value}</dd>
      </React.Fragment>
    ))}
  </dl>
);

ReactDOM.render(
  <TicketMeta
    metaData={{
      createdAt: '2018-06-09',
      author: 'Manuel Bieh',
      category: 'General',
    }}
  />,
  document.getElementById('root')
);
```

Die so erzeugte Ausgabe wäre die folgende:

```markup
<dl>
  <dt>createdAt</dt>
  <dd>2018-06-09</dd>
  <dt>author</dt>
  <dd>Manuel Bieh</dd>
  <dt>category</dt>
  <dd>General</dd>
</dl>
```

Hier wäre es beispielsweise nicht möglich, ein `div` oder `span` oder ein sonstiges Element um `<dt><dt>` und `<dd></dd>` zu wrappen. Dies würde zu folgender Ausgabe führen:

```markup
<dl>
  <div>
    <dt>createdAt</dt>
    <dd>2018-06-09</dd>
  </div>
  <div>
    <dt>author</dt>
    <dd>Manuel Bieh</dd>
  </div>
  <div>
    <dt>category</dt>
    <dd>General</dd>
  </div>
</dl>
```

… und wäre damit ungültiges HTML, da ein `dl`-Element nur `dt` und `dd` als Kind-Element erlaubt. Das **Fragment** hilft uns hier also gültiges JSX zu erzeugen, ohne dabei gleichzeitig das HTML ungültig werden zu lassen. Dies war in React bis zur Einführung von Fragments in Version 16.3. ein Problem und führte dazu, dass Komponenten unnötig kompliziert implementiert werden mussten, um weder gegen JSX- noch gegen HTML-Regeln zu verstoßen.

Die Fragment-Komponente kann auch als benannter Import direkt aus React importiert werden:

```javascript
import React, { Fragment } from 'react';
```

Bei der Verwendung kann dann `<Fragment>` geschrieben werden statt `<React.Fragment>`. Dies kann, insbesondere wenn viele `Fragment`-Elemente verwendet werden noch etwas Schreibarbeit sparen.

Wer es noch etwas kürzer möchte, der sollte Babel 7 zur Transpilierung des Codes verwenden. Hier ist außerdem eine Kurzform der Fragment-Syntax möglich. Dazu wird lediglich ein _leeres_ Element erzeugt:

```jsx
<>Fragment in Kurzform-Syntax</>
```

Eine komfortable Möglichkeit, um sich noch etwas mehr Schreibarbeit zu ersparen. Doch aufgepasst: die Verwendung der Fragment-Kurzform in einer Schleife ist hier nicht möglich, da die Kurzform-Syntax von `Fragment` keine Props besitzen kann, alle Elemente, die in einer Schleife verwendet werden jedoch eben eine `key`-Prop besitzen müssen. In diesem Fall muss dann doch wieder auf `<React.Fragment>` zurückgegriffen werden.

## Conditional Rendering

**Conditional Rendering**, also das Rendering von Komponenten auf Basis verschiedener Bedingungen ist ein zentrales Konzept in React. Da React-Komponenten unter der Haube lediglich eine Komposition aus JavaScript-Funktionen, -Objekten und -Klassen sind, funktionieren und verhalten sich Bedingungen hier exakt wie auch in herkömmlichem JavaScript.

Eine React-Komponente rendert **Zustände** eines **User Interfaces** basierend auf ihren **Props** und ihrem **aktuellen State**, optimalerweise **frei von Seiten-Effekten.** Um also korrekt auf diese verschiedenen Parameter reagieren zu können, machen wir uns Rendering-Funktionen zunutze, die an verschiedene Bedingungen geknüpft sind. Ist mein Parameter A, rendere dies; ist mein Parameter B, rendere das. Habe ich eine Liste mit Daten, zeige mir die Daten in einer HTML-Liste an. Habe ich keinerlei Daten, zeige mir stattdessen einen Platzhalter an.

Was so einfach klingt, ist es im Grunde genommen auch. Aber man sollte die richtigen Wege kennen, insbesondere in JSX. Die `render()`-Funktion von Komponenten, also sowohl von **Class Components** als auch **Stateless Functional Components** kann grundsätzlich ein **React-Element** \(natürlich auch in Form von JSX\), einen **String**, eine **Nummer**, `null` \(für den Fall, dass nichts gerendert werden soll\) oder ein **Array** aus den zuvor genannten Typen zurückgeben.

Darüber hinaus gibt es einige Möglichkeiten, die `render()`-Methoden in den Komponenten übersichtlich zu halten. Diese Möglichkeiten werde ich euch hier vorstellen.

### if/else

Die wohl einfachste und wahrscheinlich auch gängigste Form des **Conditional Renderings** ist ein klassisches `if`/`else`-Konstrukt. 

```jsx
const NotificationList = ({ items }) => {
  if (items.length) {
    return (
      <ul>
        {items.map((notification) => (
          <li>{notification.title}</li>
        ))}
      </ul>
    );
  }
  return <p>Keine neuen Benachrichtigungen</p>
};
```

Einfacher Anwendungsfall. Wir haben eine Komponente `NotificationList`, die eine Liste an Items in Form einer Prop entgegen nimmt. Enthält diese Liste Einträge, werden diese als simple ungeordnete Liste ausgegeben. Ist die Liste hingegen leer, lassen wir unsere Komponente stattdessen eben den Hinweis ausgeben, dass keine neuen Benachrichtigungen vorhanden sind.

Ein weiteres Beispiel mit einem komplexeren Fall. Wir haben einen Wert und möchten diesen editierbar machen. Unsere Komponente kennt zwei verschiedene Modi: `edit` und `view`. Je nachdem, ob wir uns im **View-Mode** oder im **Edit-Mode** befinden, möchten wir nur den Text anzeigen oder ein vorausgefülltes Textfeld mit dem jeweiligen letzten aktuellen Wert.

```jsx
import React from 'react';
import { render } from 'react-dom';

class EditableText extends React.Component {
  state = {
    value: null,
  };

  static getDerivedStateFromProps(nextProps, prevState) {
    if (prevState.value === null) {
      return {
        value: nextProps.initialValue || '',
      };
    }
    return null;
  }

  handleChange = (e) => {
    const { value } = e.target;
    this.setState(() => ({
      value,
    }));
  }

  setMode = (mode) => () => {
    this.setState(() => ({
      mode,
    }));
  }

  render() {
    if (this.state.mode === 'edit') {
      return (
        <div>
          <input 
            type="text" 
            value={this.state.value} 
            onChange={this.handleChange} />
          <br />
          <button onClick={this.setMode('view')}>Done</button>
        </div>
      );
    }

    return (
      <div>
        {this.state.value}
        <br />
        <button onClick={this.setMode('edit')}>Edit</button>
      </div>
    )
  }
}

render(
  <EditableText initialValue='Example' />, 
  document.getElementById('root')
);

```

Der für dieses Kapitel relevante Teil spielt sich innerhalb der `render()`-Methode der Komponente ab. Wir prüfen hier auf den Wert der State-Eigenschaft `mode`:  Ist dieser `edit`,  geben wir direkt das Eingabefeld zurück \(„early return“\). Ist dieser nicht `edit`, gehen wir davon aus, dass der „Standardfall“ eintritt, der in diesem Falle der Ansichtsmodus \(`view`\) wäre. Der `else`-Teil der Condition ist hier also gar nicht nötig und würde lediglich unnötig Komplexität hinzufügen. Gerendert wird jeweils der Text, einmal editierbar als `value` eines `input`-Felds, einmal lediglich als Textknoten und dazu jeweils ein Button, um die State-Eigenschaft `mode` der Komponente zwischen `view` und `edit` hin und her zu wechseln.

Derartige `if`, `if`/`else` oder `if`/`else if`/`else`-Konstrukte sind in verschiedenen Varianten, auf die ich hier gleich noch eingehen werde, eine häufige Form wenn es darum geht, eine Ausgabe auf Basis von **State** und **Props** innerhalb einer Komponente zu erzeugen.

### null

Nein, die Überschrift ist kein Fehler. `null` zurückzugeben ist wohl der einfachste Fall für **Conditional Rendering.** Gibt die `render()`-Methode einer Komponente `null` zurück, wird diese nicht gerendert und erscheint daher auch nicht im DOM. Dies kann manchmal sinnvoll sein, bspw. wenn eine Fehler-Komponente nur dann angezeigt werden soll, wenn auch ein Fehler aufgetreten ist.

```jsx
render() {
  if (!this.state.error) {
    return null;
  }

  return (
    <div className="error-message">{this.state.error.message}</div>
  );
}
```

Hier wird geprüft, ob im State der Komponente eine error-Eigenschaft gesetzt ist. Ist dies nicht der Fall, wird `null` zurückgegeben und somit auch nichts gerendert. Existiert die Eigenschaft hingegen, wird die entsprechende Fehlermeldung in einem `div` ausgegeben, wozu wir wieder auf das Conditional Rendering mit einem einfachen `if` zurückgreifen.

### Ternary Operator

Dies waren Beispiele für Bedingungen, die relativ grundlegende Unterschiede in ihren Komponenten ausgeben. Oftmals möchte man allerdings nur kleine Unterschiede ausgeben, etwa eine CSS-Klasse hinzufügen, wenn ein bestimmter State gesetzt ist. Hier hilft uns der Ternary Operator weiter. Kurze Auffrischung: der **Ternary Operator** ist ein Ausdruck und hat die Form `Bedingung ? Erfüllt : Nicht Erfüllt`. Also etwa: `isLoggedIn ? 'Logout' : 'Login';` 

Und damit hätten wir auch schon unser erstes Beispiel für die Verwendung des **Ternary Operators** innerhalb von JSX. Er kann sowohl innerhalb von Props verwendet werden, als auch einfach um je nach Bedingung, verschiedene Elemente zu rendern. Ein konkreter Anwendungsfall für das eben genannte Beispiel wäre die Ausgabe von Text in Abhängigkeit zu einer Bedingung:

```jsx
render() {
  const { isLoggedIn } = this.props;
  return (
    <button type="submit">{ isLoggedIn ? 'Logout' : 'Login' }</button>
  );
}
```

In diesem Fall würden wir stets einen Button ausgeben, dieser hätte aber abhängig von seiner `isLoggedIn`-Prop entweder die Beschriftung **Logout** oder **Login**. 

Genau in der gleichen Form kann der **Ternary Operator** in Props verwendet werden. Nehmen wir an, wir wollen eine Liste mit Benutzern ausgeben, von denen einige deaktiviert wurden. In diesem Fall möchten wir eine Klasse setzen, um diese mittels CSS markieren zu können. Ein entsprechendes Markup könnte dann bspw. so aussehen:

```jsx
render() {
  const { user } = this.props;
  return (
    <div className={user.isDisabled ? 'is-disabled' : 'is-active'}>{user.name}</div>
  );
}
```

Deaktivierte Benutzer würden hier mit einer Klasse `is-disabled` gekennzeichnet, aktive Benutzer hingegen mit einer Klasse `is-active`. 

Auch komplexeres JSX lässt sich mittels **Ternary Operator** abbilden. Dazu muss lediglich die allgemein gültige Regel befolgt werden, dass sich über mehrere Zeilen erstreckendes JSX in Klammern gefasst werden muss:

```jsx
render() {
  const { country } = this.props;
  return (
    <div>
      <p>State:</p>
      {country === 'de' ? (
        <select name="state">
          <option value="bw">Baden-Württemberg</option>
          <option value="by">Bayern</option>
          <option value="be">Berlin</option>
          <option value="bb">Brandenburg</option>
          […]
        </select>
      ) : (
        <input type="text" name="state" />
      )}
    </div>
  );
}
```

In diesem Fall rendern wir also eine Select-Liste mit allen deutschen Bundesländern, wenn das zuvor ausgewählte Land **Deutschland** \(`de`\) ist. In allen anderen Fällen zeigen wir dem Benutzer nur ein Textfeld an, in das dieser sein entsprechendes Bundesland frei eintragen kann. Hier sollte jedoch immer abgewogen werden ob dies sinnvoll ist, denn der **Ternary Operator** kann insbesondere in komplexerem JSX schnell unübersichtlich werden.

### Logical AND \(`&&`\) und Logical OR \(`||`\)

Der **Logical Operator** hat auf den ersten Blick Ähnlichkeit zum **Ternary Operator**, jedoch mit dem Unterschied, dass er noch kürzer und prägnanter ist. Anders als beim **Ternary Operator** wird hier kein „zweiter Fall“ benötigt, also ein Wert der verwendet wird, falls die Bedingung nicht erfüllt ist. Ist die Bedingung in einem **Logical AND Operator** nicht erfüllt, ist der Ausdruck `undefined` und verursacht somit keinerlei sichtbare Ausgabe im User Interface:

```jsx
render() {
  const { isMenuVisible } = this.props;
  return (
    <header>
      { isMenuVisible && <Menu /> }
    </header>
  );
}
```

In diesem Fall würde eine Komponente prüfen, ob der Wert der `isMenuVisible`-Prop `true` ist **und** dann eine `Menu`-Komponente anzeigen. Ist der Wert `false`, gibt der Ausdruck `undefined` zurück und die Komponente rendert dementsprechend keine Ausgabe an dieser Stelle. 

In Verbindung mit dem **Logical OR Operator** kann hier ein Fall wie beim **Ternary Operator** herbeigeführt werden:

```jsx
render() {
  const { isLoggedIn } = this.props;
  return (
    <button type="submit">{ isLoggedIn && 'Logout' || 'Login' }</button>
  );
}
```

Die Beschriftung des Buttons ist in diesem Fall **Logout**, wenn die `isLoggedIn` **Prop** `true` ist, der Benutzer also eingeloggt ist oder **Login**, wenn der Benutzer nicht eingeloggt ist.

### Eigene `render()`-Methoden

Eine Möglichkeit, die Übersicht bei **Conditional Rendering** zu erhöhen, ist bestimmte Teile aus der `render()`-Methode in eigene `renderXY()`-Methoden zu verfrachten. Die `render()`-Methode stellt so gesehen den Kern einer Komponente dar, ist sie doch dafür verantwortlich zu entscheiden, was ein Benutzer später auf seinem Bildschirm sieht. Sie sollte also nicht zu komplex werden, nicht unnötig viel Logik enthalten und lesbar sein.

Nicht unüblich ist es daher, sehr komplexe und lange `render()`-Methoden in kleine, übersichtliche Häppchen zu unterteilen und als eigene Klassenmethoden zu implementieren. Dies führt bei sinnvoller Benennung der jeweiligen Methoden meist zur Erhöhung und zu besserer Verständlichkeit des Codes. Meist werden die einzelnen `render()`-Methoden noch mit `if`-Blöcken kombiniert:

```jsx
class Countdown extends React.Component {
  renderTimeLeft() {
    // […]
  }
  
  renderTimePassed() {
    // […]
  }
  
  render() {
    const { currentDate, eventDate } = this.props;
    if (currentDate < eventDate) {
      // currentDate is before eventDate so render countdown
      return this.renderTimeLeft();
    }
    // time is over so render how much time has passed since then
    return this.renderTimePassed();
  }
}
```

Dies **kann** bei kluger Verwendung die Lesbarkeit einer `render()`-Methode erhöhen, führt aber unweigerlich auch dazu, dass sich die Komplexität einer Komponente \(in etwas geringerem Maß\) erhöht. Viele Leute – ich zähle mich dazu – raten daher eher dazu, Teile des Codes wiederum in eigene gekapselte **Function Components** auszulagern statt `renderXY()`-Methoden zu verwenden.

{% hint style="info" %}
Sobald die Überlegung ansteht eine weitere `render()`-Methode innerhalb einer Komponente zu implementieren sollte darüber nachgedacht werden, stattdessen eine eigene, separate **Function Component** zu erstellen.
{% endhint %}

### Eigene Komponenten bei komplexen Conditions

Statt weiterer `render()`-Methoden innerhalb einer Komponente können wie eben bereits angesprochen auch eigene, neue, bevorzugterweise **Function Components** erstellt werden. Diese bekommen dann entsprechende **Props** aus ihrer Eltern-Komponente hereingereicht und kümmern sich dann als eigenständige, unabhängige, wiederverwendbare und testbare Komponente um die Anzeige der ihnen übergebenen Daten. 

An erster Stelle sollte die Überlegung stehen, wie einfach sich die Daten aus der ursprünglichen Eltern-Komponente in die neue\(n\) Kind-Komponente\(n\) übertragen lassen und vor allem, welche Daten überhaupt in eine neue Komponente ausgelagert werden sollten. Dabei sollte beachtet werden, dass die neuen Komponenten selbst wiederum nicht wieder zuviel Logik oder gar State enthalten sollten.

Dieses Vorgehen bietet sich vor allem dann an, wenn wiederkehrende Elemente in einer Komponente verwendet werden oder eine `render()`-Methode eben zu groß und unübersichtlich wird. 

Stellen wir uns ein Formular vor, das aus zumeist sehr ähnlichen Textfeldern besteht. Jedes Textfeld befindet sich in einem eigenen Paragraphen, hat ein Label und natürlich auch ein `type`-Attribute. Zum Label gehört außerdem auch eine id, die ebenfalls für jedes Feld angegeben werden muss:

```jsx
render() {
  return (
    <form>
      <p>
        <label for="email">
          Email
        </label>
        <br />
        <input type="email" name="email" id="email" />
      </p>
      <p>
        <label for="password">
          Password
        </label>
        <br />
        <input type="password" name="password" id="password" />
      </p>
      <input type="submit" value="Send" />
    </form>
  );
}
```

In diesem Fall haben wir lediglich zwei Formularfelder. Oft ist es aber bereits in durchschnittlich komplexen Anwendungen so, dass es deutlich größere Formulare mit deutlich mehr Feldern gibt. Doch bereits in diesem Fall kann es sinnvoll sein, die sich wiederholenden Felder in eigene Komponenten auszulagern, da wir uns viel Schreibarbeit ersparen können.

Wir erstellen also zunächst eine `TextField`-Komponente und lagern das sich wiederholende JSX aus unserer Formular-Komponente dorthin aus:

```jsx
const TextField = ({ id, label, ...HTMLInputAttributes }) => (
  <p>
    <label for={id}>
      {label}
    </label>
    <br />
    <input {...HTMLInputAttributes} id={id} />
  </p>
);

export default TextField;
```

Unsere neue Komponente empfängt eine `id`, die wir benötigen, um das Label mit dem Eingabefeld zu verknüpfen und ein Label als solches. Mittels **Object Rest/Spread** fügen wir dem `input`-Element dann außerdem alle weiteren Props, die der Komponente übergeben werden, als Attribut hinzu.

Unsere Komponente von oben sieht dann wie folgt aus:

```jsx
render() {
  return (
    <form>
      <TextField name="email" label="Email" id="email" type="email" />
      <TextField name="password" label="Password" id="password" type="password" />
      <input type="submit" value="Send" />
    </form>
  );
}
```

Aus einem langen und potentiell sehr unübersichtlichen Markup haben wir also eine übersichtliche, prägnante `render()`-Methode gemacht, die auf oberster Ebene aus wenigen Komponenten besteht. Möchten wir in Zukunft außerdem eine Änderung vornehmen, die sich auf alle Textfelder auswirkt, bspw. eine neue Klasse hinzufügen, muss dies nur noch an einer einzigen Stelle geändert werden – in der neuen `TextField`-Komponente.

