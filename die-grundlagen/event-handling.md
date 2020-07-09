# Event-Handling

Ein wesentlicher Teil bei der Entwicklung von Anwendungen mit einem komplexen User Interfaces ist natürlich die Interaktion zwischen Benutzer und dem Interface an sich - insbesondere in Form von **Events**.

Ich drücke einen Knopf und es passiert etwas. Ich schreibe einen Text in ein Feld und es passiert etwas. Ich wähle ein Element aus einer Liste und es passiert etwas. In einfachem JavaScript stellt uns der Browser dafür die Methoden `addEventListener()` und `removeEventListener()` bereit. Auf diese beiden Event-Methoden kann in React in den meisten Fällen nahezu komplett verzichtet werden, da React ein eigenes System zum Definieren von Benutzerinteraktion gleich mitbringt. Und zwar, nicht erschrecken: über **Inline-Events**.

Diese **Inline-Events** haben rein äußerlich sehr große Ähnlichkeit zu den HTML Event-Attributen \(also bspw. `<button onclick="myFunction" />`\) und doch ist ihre Funktionsweise grundlegend anders.

Da wurde uns Web-Entwicklern über Jahre immer wieder beigebracht, dass man seine Event-Listener sauber von seinem Markup trennen sollte, **Separation of Concerns**, dann kommt React daher und macht wieder einfach alles anders.

Und das ist in diesem Fall auch gut so, denn durch die Verwendung eines eigenen Systems zur Verwaltung von Events, nimmt uns React hier wieder eine ganze Menge Arbeit ab und ermöglicht es uns außerdem, sehr leicht im Komponenten-Kontext zu bleiben, indem wir alle Event-Handler als **Klassen-Methoden** implementieren können und so sowohl Darstellungslogik als auch Verhaltenslogik in einer einzigen Komponente kapseln können. Kein mühsames und unübersichtliches hin- und herspringen zwischen Controllern und Views!

### Unterschiede zur nativen Event API

Events in React und JSX werden wie erwähnt sehr ähnlich definiert wie HTML Event-Attribute. Doch es gibt einige Unterschiede. So werden Events in React in **CamelCase**-Form definiert, statt in **Lowercase**. Somit wird aus `onclick` in React `onClick`, aus `onmouseover` wird `onMouseOver`, `ontouchstart` wird zu `onTouchStart` usw.

Der dem Event-Handler übergebene erste Parameter ist auch kein Objekt vom Typ `Event`, sondern ein sogenannter `SyntheticEvent`, ein React-eigener Wrapper um das native Event-Objekt. Dieser Wrapper ist Teil des React Event Systems und dient als eine Art Normalisierungsschicht, um Cross-Browser-Kompatibilität zu gewährleisten. Dabei hält diese sich, anders als so mancher Browser, strikt an die [Event-Spezifikation des W3C](https://www.w3.org/TR/DOM-Level-3-Events/).

Ein weiterer Unterschied zur nativen Browser-Event-API ist der, dass explizit `preventDefault()` aufgerufen werden muss um das Standardverhalten des Browsers bei einem Event zu unterdrücken, statt lediglich `false` aus dem Event-Handler zurückzugeben.

Und nicht zuletzt ist der im Event-Attribut \(bzw. Event-**Prop** in JSX\) angegebene Wert eine **Referenz zu einer Funktion** statt wie in HTML ein String. Folglich benötigen wir hier die geschweiften Klammern um JSX mitzuteilen, dass es sich um einen JavaScript-Ausdruck handelt.

Wie das aussieht? So:

```jsx
<button onClick={validateInput}>Validate</button>
```

Wohingegen ein vergleichbarer Event in HTML wie folgt aussähe:

```markup
<button onclick="validateInput">Validate</button>
```

Dieses Vorgehen, das auf den ersten Blick einigen vielleicht erst einmal etwas eigenartig anmuten könnte, bringt aber einige großartige Vorteile mit sich. Und so bekommen wir, wie eingangs bereits angesprochen, Cross-Browser-Kompatibilität „for free“. React registriert dabei im Hintergrund die Events sauber mittels `addEventListener()` und entfernt diese auch automatisch wieder für uns, sobald die Komponente entfernt \(_unmounted_\) wird. Praktisch!

### Scopes in Event-Handlern

Bei der Verwendung von ES2015-Klassen in React ist es für gewöhnlich so, dass Event-Handler als Methoden der Klassen-Komponente implementiert werden. Hierbei muss jedoch beachtet werden, dass **Klassen-Methoden nicht automatisch an die Instanz gebunden werden**. Klingt kompliziert, bedeutet aber lediglich, dass `this` in eurem Event-Handler grundsätzlich erstmal `undefined` ist.

Hierzu ein kurzes Beispiel:

```jsx
class Counter extends React.Component {
  state = {
    counter: 0,
  };

  increase() {
    this.setState((state) => ({
      counter: state.counter + 1,
    }));
  }

  render() {
    return (
      <div>
        <p>{this.state.counter}</p>
        <button onClick={this.increase}>+1</button>
      </div>
    );
  }
}
```

Hier definieren wir also einen `onClick`-Event, um den Zähler jeweils um `1` zu erhöhen, sobald der Benutzer auf den Button `+1` klickt. Beim Klick auf den Button sieht unser Benutzer aber stattdessen:

{% hint style="danger" %}
**TypeError**

Cannot read property 'setState' of undefined
{% endhint %}

Und warum? **Scoping!** Da wir uns beim Klick auf den Button im Event-Handler `increase()` außerhalb der Komponenten-Instanz bewegen, können wir eben auch nicht auf `this.setState()` zugreifen. Dies ist kein fehlerhaftes Verhalten von React, sondern das Standardverhalten von ES2015-Klassen. Um dieses Problem zu lösen, gibt es nun verschiedene Möglichkeiten.

#### Method-Binding in der render\(\)-Methode

Die trivialste Methode ist das Binden der Methode innerhalb der `render()`-Methode. Dazu fügen wir ein `.bind(this)` an die Referenz zur Klassen-Methode:

```jsx
<button onClick={this.increase.bind(this)}>+1</button>
```

Die Methode wird nun **im Scope der Komponenten-Instanz** aufgerufen und unser Counter zählt problemlos hoch. Dieses Muster sieht man relativ häufig, da es einfach implementiert ist, es hat jedoch einen grundlegenden Nachteil. Denn es wird bei jedem Aufruf der Methode „on-the-fly“ eine neue Funktion erzeugt, die nicht mehr mit der vorherigen identisch ist. Ein simpler Check in einer `shouldComponentUpdate()`-Methode, der auf Gleichheit von `this.props.increase === prevProps.increase` prüft, würde somit stets `false` ergeben und ggf. ein Re-Rendering einer Komponente veranlassen, auch wenn sich die Funktion an sich nicht geändert hat. Dies ist ein potentielles **Performance-Bottleneck** und sollte daher möglichst vermieden werden!

#### Method-Binding im Constructor

Eine andere, sauberere Möglichkeit eine Methode an eine Klassen-Instanz zu binden, ist dies im Constructor bei der Instanziierung einer Klasse zu tun.

```jsx
class Counter extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      counter: 0,
    };
    this.increase = this.increase.bind(this);
  }
  // […]
}
```

Auf diesem Weg wird die Methode nur **einmal** an die Instanz gebunden. Ein eventueller Check auf Gleichheit von Methoden würde also `true` ergeben und unnötige Re-Renderings könnten bei der Verwendung sinnvoller `shouldComponentUpdate()`-Checks vermieden werden. Allerdings bringt dieser Weg auch eine ganze Menge Overhead mit sich. Vorausgesetzt, wir nutzen nicht ohnehin bereits einen Constructor, müssen wir diesen nun implementieren. Dazu sollten wir die `super(props)` Methode aufrufen, um die **Props** der Komponente an die `React.Component` Eltern-Klasse zu übergeben. Letztendlich schreiben wir noch zweimal den Namen der Methode, die wir binden wollen, indem wir sie sozusagen mit der an `this` gebundenen Version von sich selbst überschreiben.

Das ist dennoch schon mal besser als die erste Methode, auch wenn es mehr Schreibarbeit ist, da wir uns potentielle Performance-Bottlenecks ersparen. Aber es gibt noch eine weitere, einfache Möglichkeit, um mit minimalem Mehraufwand eine Methode an eine Instanz zu binden.

#### Benutzung von Class Properties

**Achtung:** dieser Weg setzt die Verwendung des Babel-Plugins `@babel/plugin-proposal-class-properties` voraus. Da dies jedoch wie bereits in der Einleitung beschrieben in den meisten React-Setups zum Standard gehört, gehe ich also davon aus, dass **Class Properties** genutzt werden können. Ist dies nicht der Fall, sollten Event-Handler-Methoden immer im Constructor gebunden werden!

Aber wie binden wir jetzt unsere Methode indem wir **Class Properties** nutzen? Streng genommen: indem wir schummeln! Statt wie im Eingangsbeispiel eine echte Klassen-Methode zu implementieren, definieren wir eine **Public Class Property**, die als Wert eine per **Arrow Function** an die jeweilige Klasse gebundene Function zugewiesen bekommt. Das sieht dann wie folgt aus:

```jsx
class Counter extends React.Component {
  state = {
    counter: 0,
  };
  increase = () => {
    this.setState((state) => ({
      counter: state.counter + 1,
    }));
  };
}
```

Der entscheidende Faktor liegt hier in der ersten Zeile. Statt:

```jsx
increase() { … }
```

Schreiben wir:

```jsx
increase = () => { … }
```

**Problem gelöst!**

Der Unterschied liegt hier wie erwähnt darin, dass wir im ersten Beispiel eine **echte Klassen-Methode** implementieren und im zweiten Fall stattdessen einer gleichnamigen Eigenschaft dieser Klasse eine **Arrow Function als Wert** zuweisen. Da diese kein eigenes `this` bindet, greifen wir hier auf das `this` der Klassen-Instanz zu.

### Events außerhalb des Komponenten-Kontexts

Die Verwendung von React schließt auch die Implementierung von nativen Browser-Events nicht aus. Allerdings sollte nach Möglichkeit immer das React-eigene Event-System verwendet werden, da dieses Cross-Browser-Kompatibilität mitbringt, nach dem W3C Standard für Browser-Events arbeitet und zahlreiche Optimierungen vornimmt.

Gelegentlich ist es jedoch notwendig, Events außerhalb des Komponenten-Kontexts zu definieren. Ein Klassisches Beispiel sind `window.onresize` oder `window.onscroll` Events. Hier bietet React von Haus aus keine Möglichkeit, um globale Events außerhalb des komponentenspezifischen JSX zu definieren. Hier ist dann die `componentDidMount()`-Methode der richtige Ort um diese zu definieren. Dabei sollte allerdings auch stets darauf geachtet werden, dass Events, die mittels `addEventListener()` definiert werden, **immer auch entfernt werden!**

Der richtige Ort **dafür** ist dann die `componentWillUnmount()`-Methode. Werden eigene definierte globale Events nicht entfernt, werden diese mit jedem Mounting einer Komponente **erneut** hinzugefügt und auch erneut **mehrmals** aufgerufen, was letztendlich ebenfalls zu **Performance-Bottlenecks** und sogar zu **Memory-Leaks** führen kann.

### Arbeiten mit dem `SyntheticEvent` Objekt

**React übergibt Event-Handlern kein natives Event-Objekt,** sondern ein eigenes Objekt vom Typ `SyntheticEvent`. Dies dient insbesondere dazu, Cross-Browser kompatibles Verhalten zu gewährleisten. React hält das originale Event aber in der Objekt-Eigenschaft `nativeEvent` bereit. Sollte also jemals der Bedarf bestehen auf den Original-Event zuzugreifen \(ist mir bisher noch nie passiert\): ihr findet ihn in der `e.nativeEvent` Eigenschaft.

Das `SyntheticEvent`-Objekt hat aber noch eine weitere Besonderheit im Vergleich zum nativen Event-Objekt, denn es ist **kurzlebig**. Aus Performance-Gründen wird das Objekt nach dem Aufruf des Event-Callbacks **nullified**, also kurz gesagt: es wird zurückgesetzt auf den Ausgangszustand. Ein Zugriff auf Eigenschaften des Event-Objekts außerhalb des originalen Event-Handlers ist daher nicht ohne Weiteres möglich.

Was bedeutet das? Nun, folgendes Beispiel:

```jsx
class TextRepeater extends React.Component {
  state = {};

  handleChange = (e) => {
    this.setState((state) => ({
      value: e.target.value,
    }));
  };

  render() {
    return (
      <div>
        <input type="text" onChange={this.handleChange} />
        <p>{this.state.value}</p>
      </div>
    );
  }
}
```

Wir registrieren ein `onChange`-Event, das bei einer Änderung im Textfeld den eingegebenen Wert in einem Paragraphen unter dem Textfeld wieder ausgibt. Um im Event-Handler auf den eingegebenen Wert zuzugreifen, das kennt ihr möglicherweise bereits aus nativem JavaScript oder von jQuery-Events, stellt das `Event`-Objekt die Eigenschaft `target` bereit. Damit bekommen wir das Element, auf dem das Event stattgefunden hat, in unserem Beispiel also das Textfeld. Dieses wiederum besitzt eine `value`-Eigenschaft mittels der wir dann auf den aktuellen Wert des Textfelds zugreifen, um diesen in unseren State zu schreiben.

Hier haben wir aber nun mit einem Fallstrick zu tun: der `this.setState()`-Aufruf nutzt eine **Updater-Funktion**, also einen Callback. Dieser findet außerhalb des eigentlichen Event-Handler Scopes statt. Das bedeutet, das `SyntheticEvent` wurde zu diesem Zeitpunkt bereits wieder zurückgesetzt und `e.target` existiert zum Zeitpunkt des Aufrufs der Updater-Funktion schon gar nicht mehr:

{% hint style="danger" %}
**TypeError**

Cannot read property 'value' of null
{% endhint %}

Die einfachste Lösung wäre hier statt der Updater-Funktion ein Object-Literal zu verwenden:

```jsx
handleChange = (e) => {
  this.setState({
    value: e.target.value,
  });
};
```

Damit wäre unser Problem in dem Fall zwar gelöst, das hilft uns aber trotzdem nicht sonderlich weiter. Denn auf das gleiche Problem stoßen wir auch, wenn wir auf Eigenschaften des `SyntheticEvent`-Objekts bspw. in einem `setTimeout()`-Callback zugreifen wollen. Wir müssen uns also etwas anderes einfallen lassen.

#### Werte in Variablen schreiben

In den meisten Fällen sollte es ausreichen, wenn einzelne Werte, auf die später im Callback zugegriffen werden soll, in eine Variable geschrieben werden. Im Callback wird dann nicht mehr auf das `SyntheticEvent` zugegriffen, sondern lediglich auf die Variable, die einen Wert aus dem `SyntheticEvent` zugewiesen bekommen hat.

```jsx
handleChange = (e) => {
  const value = e.target.value;
  this.setState(() => ({
    value: value,
  }));
};
```

Klappt. Bonuspunkte für Eleganz gibt es bei der Verwendung von **Object Destructuring** und dem **Object Property Shorthand**:

```jsx
handleChange = (e) => {
  const { value } = e.target;
  this.setState(() => ({ value }));
};
```

#### Persistieren von `SyntheticEvents` mittels `e.persist()`

Theoretisch möglich, in der Praxis aus meiner Erfahrung eher irrelevant: Das `SyntheticEvent`-Objekt stellt eine eigene Methode `persist()` bereit, mit der eine Referenz zum entsprechenden Event beibehalten \(also persistiert\) werden kann. Ein möglicher Anwendungsfall wäre hier, das gesamte `SyntheticEvent`-Objekt an eine Callback-Funktion **außerhalb** des Event-Handlers weiterzugeben.

Sollte das jedoch notwendig sein, lohnt es sich aber womöglich darüber nachzudenken, ob der Code der externen Callback-Funktion nicht besser im Event-Handler selbst aufgehoben wäre. Unsere Beispielfunktion von oben sieht in diesem Fall so aus:

```jsx
handleChange = (e) => {
  e.persist();
  this.setState(() => ({
    value: e.target.value,
  }));
};
```

Zuerst rufen wir besagte `e.persist()`-Methode auf. Anschließend können wir auch in der **Updater-Funktion** sorglos auf `e.target` und dessen `value`-Eigenschaft zugreifen.

<span class="force-break-before"></span>

### Fazit

- Zum Definieren von Events möglichst **immer** die Event-Props im JSX verwenden: `onChange`, `onMouseOver`, `onTouchStart`, `onKeyDown`, `onAnimationStart` usw. auch wenn es anfangs gewöhnungsbedürftig wirken sollte.
- Event-Handler müssen explizit an die Klassen-Instanz gebunden werden, wenn auf andere Klassen-Methoden wie bspw. `this.setState()` zugegriffen wird. Der eleganteste Weg hierfür sind **Public Class Properties** und **Arrow Functions.**
- Eigene Events über die `addEventListener()`-API zu definieren sollte möglichst vermieden werden. Lässt es sich einmal nicht vermeiden, unbedingt daran denken, diese Events beim Unmounting der Komponente mittels `removeEventListener()` zu entfernen!
- `SyntheticEvent`-Objekte werden „nullified“. Vorsicht bei der Verwendung in Callback-Funktionen außerhalb des jeweiligen Event-Handlers! Hier ist es gut möglich, dass das Event-Objekt beim Aufruf des Callbacks mittlerweile nicht mehr existiert.
- Mittels `event.persist()` kann erzwungen werden, dass das das Event-Objekt nicht durch React auf `null` gesetzt wird.
