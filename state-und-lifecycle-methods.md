# State und Lifecycle-Methods

Kommen wir zu dem, was die Arbeit mit React erstmal wirklich effizient macht: **State**, **stateful Components** und die sogenannten **Lifecycle-Methods**.

Wie im Kapitel über Komponenten bereits angesprochen können **stateful Components** einen eigenen  Zustand, den **State**, halten, verwalten und verändern. Dabei gilt der Grundsatz: **ändert sich der State einer Komponente, löst dies immer auch ein Re-Rendering der Komponente aus!** Dieses Verhalten kann in **Class Components** auch explizit unterbunden werden, was in einigen Fällen sinnvoll ist. Aber der Grundsatz bleibt unverändert: eine State-Änderung löst ein Re-Rendering einer Komponente und ihrer Kind-Komponenten aus.

Das ist insofern hilfreich, als dass wir nicht mehr manuell `ReactDOM.render()` aufrufen müssen wann immer wir meinen dass sich etwas an unserem Interface geändert hat, sondern die Komponenten dies stattdessen selbst entscheiden können.

Neben dem State an sich gibt es auch eine handvoll sogenannter Lifecycle-Methoden. Dies sind Methoden die **optional** in einer **Class Component** definiert werden können und von React bei bestimmten Anlässen ausgeführt werden. Beispielsweise wenn eine Komponente erstmals gemountet wird oder sich dessen Props aktualisiert haben.

## Eine erste stateful Component

Der **State** innerhalb einer Komponente ist verfügbar über die Instanz-Eigenschaft `this.state` und ist innerhalb einer **Komponente** gekapselt. Weder Eltern- noch Kind-Komponenten können ohne weiteres auf den State einer anderen Komponente zugreifen.

Um in einer Komponente einen initialen Zustand zu definieren gibt es zwei einfache Wege, einen dritten lernen wir später noch mit der Lifecycle-Methode `getDerivedStateFromProps()` kennen.

Initialen State kann man definieren, indem man im Constructor die Instanz-Eigenschaft `this.state` setzt:

```javascript
class MyComponent extends React.Component {
  constructor(props) {
    this.state = {
      counter: props.counter,
    };
  }
}
```

… oder indem man den State als **ES2017** **Class Property** definiert, was deutlich kürzer ist, jedoch momentan noch das **Babel-Plugin** `@babel/plugin-proposal-class-properties` \(vor Babel 7: `babel-plugin-transform-class-properties`\) benötigt:

```javascript
class MyComponent extends React.Component {
  state = {
    counter: this.props.counter,
  };
}
```

**Create-React-App** unterstützt die **Class Property Syntax** standardmäßig und da viele React-Projekte heute vollständig oder zumindest zu gewissen Teilen auf dem CRA-Setup oder Varianten davon basieren, kommt diese Syntax heute in den meisten Projekten zum Einsatz und kann genutzt werden. Sollte dies mal in einem Projekt nicht der Fall zu sein, empfehle ich dringend die Installation und Nutzung des Babel-Plugins, da es wirklich viele unnötige Zeilen Code bei der täglichen Arbeit mit React einspart, während es gleichzeitig in nur wenigen Minuten eingerichtet ist.

Ist der **State** erst einmal definiert, können wir innerhalb der **Class Component** mittels `this.state` **lesend** auf ihn zugreifen. Lesend ist hier ein entscheidendes Stichwort. Denn auch wenn es prinzipiell möglich ist den State direkt über `this.state` zu verändern sollte dies aus verschiedenen Gründen vermieden werden.

## Den State verändern mit this.setState\(\)

Um State zu verändern stellt React eine eigene Klassen-Methode bereit: 

```javascript
 this.setState()
```

Wann immer der State innerhalb einer Komponente verändert werden soll, sollte dafür `this.setState()` verwendet werden. Der Aufruf von `this.setState()` führt dann dazu, dass React entsprechende Lifecycle-Methoden \(wie bspw. `componentDidUpdate()`\) ausführt und eine Komponente **neu rendert!** Würden wir den State stattdessen direkt verändern, also bspw. `this.state.counter = 1;` schreiben, hätte dies keinerlei Auswirkungen auf unsere Komponente und alles würde aussehen wie bisher, da der Render-Prozess **nicht** ausgelöst werden würde!

Die Methode ist von der Funktionsweise her allerdings etwas komplexer als es auf den ersten Moment aussehen mag. Und so wird nicht einfach nur der alte State durch den neuen State ersetzt und ein Re-Rendering ausgelöst. Es passieren auch noch allerhand andere Dinge. Der Reihe nach.

Zuerst einmal kann die Funktion **zwei verschiedene Arten von Argumenten** entgegennehmen. Das ist einerseits ein **Objekt** mit neuen oder aktualisierten State-Eigenschaften, sowie andererseits eine **Updater-Funktion,** die wiederum ein Objekt zurückgibt oder `null`, falls nichts geändert werden soll. Bestehende gleichnamige Eigenschaften innerhalb des State-Objekts werden dabei **überschrieben**, alle anderen bleiben **unangetastet!** Möchten wir eine Eigenschaft im State zurücksetzen, müssen wie diese dazu also explizit auf `null` oder `undefined` setzen. Der übergebene State wird also immer mit dem bestehenden State **zusammengefügt**, niemals **ersetzt!**

Nehmen wir nochmal unseren oben definierten State mit einer `counter` Eigenschaft, deren initialer Wert für dieses Beispiel erst einmal `0` ist. Nun verändern wir den State und möchten diesem zusätzlich eine `date` Eigenschaft mit dem aktuellen Datum hinzufügen. Übergeben als Objekt wäre unser Aufruf:

```javascript
this.setState({
  date: new Date(),
});
```

Nutzen wir stattdessen eine **Updater-Funktion**, wäre unser Aufruf:

```javascript
this.setState(() => {
  return {
    date: new Date(),
  };
});
```

Oder kurz:

```javascript
this.setState(() => ({
  date: new Date(),
});
```

Unsere Komponente hat anschließend den neuen State:

```javascript
{
  counter: 0,
  date: new Date(),
}
```

Um sicherzustellen stets auf den aktuellen State zuzugreifen, sollte eine **Updater-Funktion** verwendet werden, die den jeweils aktuellen State als ersten Parameter übergeben bekommt. Ein beliebter Fehler der vielen Entwicklern bei der Arbeit mit React schon passiert ist, ist es direkt nach einem `setState()`-Aufruf auf `this.state` zuzugreifen und sich zu wundern, dass der State noch immer der alte ist.

{% hint style="danger" %}
React „sammelt“ schnell aufeinanderfolgende `setState()`-Aufrufe und **führt diese nicht unmittelbar aus**, um unnötig häufiges und überflüssiges Re-Rendering von Komponenten zu vermeiden. Schnell aufeinanderfolgende `setState()`-Aufrufe werden später in gesammelter Form ausgeführt. Das ist wichtig, da wir somit auch nicht unmittelbar nach einem `setState()`-Aufruf mittels `this.state` auf den neu gesetzten State zugreifen können.
{% endhint %}

Stellen wir uns eine Situation vor in der unser `counter`-State dreimal in schneller Abfolge erhöht werden soll. Intuitiv würde man nun vermutlich folgenden Code schreiben:

```text
this.setState({ counter: this.state.counter + 1 });
this.setState({ counter: this.state.counter + 1 });
this.setState({ counter: this.state.counter + 1 });
```

Was denkst du, wie ist der neue State wenn der initiale State `0` war? `3`? Falsch. Er ist `1`! Hier kommt der angesprochene **Batching-Mechanismus** von React zum Zug. Um ein sich zu schnell aktualisierendes User Interface zu vermeiden, wartet React hier erst einmal ab. Am Ende kann man den obigen Code simpel gesagt **vom Funktionsprinzip her** in etwa gleichsetzen mit:

```text
this.state = Object.assign(
  this.state, 
  { counter: this.state.counter + 1 },
  { counter: this.state.counter + 1 },
  { counter: this.state.counter + 1 },
);
```

Die `counter`-Property überschreibt sich hier also immer wieder selbst, nimmt aber stets `this.state.counter` als Basiswert für die Erhöhung um 1.

