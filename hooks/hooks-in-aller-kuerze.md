# Verwendung von Hooks

Insgesamt bringt React aktuell **10** eigene Hooks mit von denen die offizielle Dokumentation **3** als **Basic**, also als _grundlegende_, die restlichen **7** als **Additional**, also als _zusätzliche_ Hooks betitelt. Und in der Tat ist diese Unterteilung sinnvoll, denn in den meisten Fällen in denen mit Hooks gearbeitet wird, wird man die 3 Basic Hooks `useState()`, `useEffect()` und `useContext()` verwenden. 

Die als **Additional Hooks** bezeichneten Ausprägungen sind insbesondere für spätere Optimierungen oder zum Abdecken von Edge-Cases vorgesehen. In diesem Kapitel soll es daher erst einmal um die „simplen“ Hooks gehen und ich möchte demonstrieren wie man Funktionalität mit **Function Components** realisieren kann, für die bisher **Klassen-Komponenten** notwendig waren.

### State mit useState\(\)

Werfen wir zunächst einen Blick darauf, wie wir bisher auf State in Komponenten zugegriffen haben und wie wir ihn modifiziert haben. 

```jsx
import React from 'react';
import ReactDOM from 'react-dom';

class Counter extends React.Component {
  state = {
    value: 0,
  };
  render() {
    return (
      <div>
        <p>Zählerwert: {this.state.value}</p>
        <button onClick={() => this.setState((state) => ({ value: state.value + 1 }))}>
          +1
        </button>
      </div>
    );
  }
}

ReactDOM.render(<Counter />, document.getElementById('root'));
```

Hier implementieren wir beispielhaft einen Zähler, der mitzählt, wie oft wir den **+1**-Button gedrückt haben. Zugegeben, nicht gerade kreativ, demonstriert aber sehr gut, wie uns Hooks das Leben an dieser Stelle einfacher machen. Wir zeigen den aktuellen Wert an indem wir `this.state.value` auslesen und haben darunter einen Button, mit dem wir den Wert erhöhen können. Dazu rufen wir `this.setState()` auf und setzen den neuen `value` auf den vorherigen Wert den wir um 1 erhöhen.

Schauen wir uns die identische Funktionalität noch einmal an, diesmal in einer Function Component mit dem neuen `useState()`-Hook:

```jsx
import React from 'react';
import ReactDOM from 'react-dom';

const Counter = () => {
  const [ value, setValue ] = React.useState(0);
  return (
    <div>
      <p>Zählerwert: {value}</p>
      <button onClick={() => setValue(value + 1)}>
        +1
      </button>
    </div>
  );
}

ReactDOM.render(<Counter />, document.getElementById('root'));
```

Hier haben wir kein `state`-Objekt mehr und auch kein `this` über das wir auf den State zugreifen. Stattdessen sehen wir hier den Aufruf des internen `useState()`-Hooks. Dieser funktioniert so, dass er einen **Initialwert** übergeben bekommt \(hier: `0`\) und einen Tupel zurückgibt, also ein Array mit der gleichen Anzahl von Werten. Im Falle des `useState()`-Hooks sind das zum einen der aktuelle State sowie eine Setter-Funktion, mit der wir den Wert modifizieren können.

Um direkt auf den Wert und die Setter-Funktion zuzugreifen machen wir uns die **ES2015 Array Destructuring**-Methode zu nutze. Diese sorgt dafür, dass der erste Wert des Arrays in die Variable `value`, der zweite Wert in die Variable `setValue` geschrieben wird. Beide Namen sind dabei frei wählbar, jedoch hat es sich schnell eingebürgert dem State einen kurzen, prägnanten Namen zu geben und der Setter-Funktion den gleichen Namen zu geben mit einem set davor. Die Syntax ist also eine Abkürzung für das folgende ES5 Äquivalent:

```javascript
var state = React.useState(0);
var value = state[0];
var setValue = state[1];
```

Im JSX, das wir aus der `Counter`-Komponente zurückgeben, greifen wir nun direkt auf `value` zu statt auf `this.state.value` und setzen den neuen Wert mittels eines sehr kurzen `setValue(value + 1)` statt wie zuvor in der Klassen-Komponente mittels `this.setState((state) => ({ value: state + 1 }))`. 

Wir haben so eben unsere erste **Function Component** erstellt die **stateful** ist!

{% hint style="info" %}
Wir haben an dieser Stelle unseren ersten **Hook** verwendet: `useState()`. Dazu haben wir die Methode `React.useState()` aufgerufen. **Hooks** können auch direkt aus dem `react`-Package importiert werden. 

Dies ist insbesondere sinnvoll wenn ein Hook mehrmals verwendet wird. So schreibt man sich dann in der Komponente einiges an Schreibarbeit:

```jsx
import React, { useEffect, useState } from 'react';
```

So können wir in der Komponente bequem direkt `useState()` nutzen statt jedesmal `React.useState()` schreiben zu müssen.
{% endhint %}

### Seiteneffekte mit useEffect\(\)

Der `useEffect()`-**Hook** erhält seinen Namen daher, da dieser dazu vorgesehen ist ihn für **Side Effects** zu benutzen. Also Seiteneffekte wie dem Laden von Daten via API, dem Registrieren von globalen Events oder der Manipulation von DOM-Elementen. Der Hook bildet die Funktionalität der `componentDidMount()`, `componentDidUpdate()` und `componentWillUnmount()`-Lifecycle Methoden ab.

Ja, richtig gelesen: statt der genannten _drei_ Methoden gibt es nun nur noch _einen einzigen_ **Hook**, der an die vergleichbare Stelle der Methoden aus Klassen-Komponenten tritt. Der Trick dabei ist die genaue Verwendung ganz bestimmter Funktionsparameter und Rückgabewerte, wie sie für den `useEffect()`-Hook vorgesehen sind.

Zur Benutzung des Hooks wird der `useEffect()`-Funktion selbst eine Funktion als ersten Parameter übergeben. Diese Funktion, nennen wir sie der Einfachheit halber **„Update-Funktion“**, wird von React grundsätzlich erst einmal **nach** jedem Rendering der Komponente ausgeführt und tritt damit an die Stelle von `componentDidUpdate()` in Klassen-Komponenten.

Da diese Update-Funktion nach **jedem** Rendering der Komponente aufgerufen wird, wird sie eben auch nach dem **ersten** Rendering aufgerufen, was an dieser Stelle gleichgesetzt werden kann mit der `componentDidMount()` Lifecycle-Methode aus den Klassen-Komponenten. 

Darüber hinaus kann die _Update_-Funktion optional selbst wiederum eine Funktion zurückgeben. Nennen wir  sie **„Aufräum-Funktion“**. Diese _Aufräum_-Funktion wird beim **Unmounting** der Komponente aufgerufen, womit wir bei der nächsten Lifecycle-Methode wären, nämlich `componentWillUnmount()`. 

Doch Vorsicht: hier haben wir gleichzeitig auch die erste Abweichung in der Funktionsweise verglichen mit Klassen-Komponenten. Und so wird unsere _Aufräum_-Funktion nicht nur beim **Unmounting** der Komponente aufgerufen sondern auch **vor jeder erneuten Ausführung** der _Update_-Funktion. 

Dieses Verhalten kann aktiv gesteuert werden, indem dem `useEffect()`-Hook als zweiten Parameter ein Array mit sog. **Dependencies** übergeben wird. Dabei handelt es sich um Werte von denen die Ausführung der _Update_-Funktion abhängt. Wird ein **Dependency-Array** übergeben wird der Hook nur einmal initial ausgeführt und dann erst wieder, wenn sich mind. einer der Werte im **Dependency-Array** geändert hat.

Möchte man hingegen gezielt ein `componentDidMount()` simulieren kann ein leeres Array als zweiter Parameter übergeben werden. React führt die Effekt-Funktion dann nur beim ersten Rendern aus und ruft dann erst beim **Unmounting** wieder eine eventuell definierte Aufräum-Funktion auf.

Das klingt jetzt alles sicher wieder fürchterlich kompliziert wenn man mit der Funktionsweise des Hooks nicht vertraut ist, lässt sich aber mit einem kurzen Code-Beispiel relativ einfach demonstrieren:

```jsx
import React, { useEffect, useState } from 'react';
import ReactDOM from 'react-dom';

const defaultTitle = 'React mit Hooks';

const Counter = () => {
  const [ value, setValue ] = useState(0);
  
  useEffect(() => {
    // `document.title` wird bei jeder Änderung (didMount/didUpdate) gesetzt.
    // Vorausgesetzt der `value` hat sich gendert
    document.title = `Der Button wurde ${value} mal geklickt`;

    // Hier geben wir unsere „Aufräum-Funktion“ zurück die vor jedem Update
    // den Titel auf den Standardwert zurücksetzt
    return () => {
      document.title = defaultTitle;
    }
  
  // Zuletzt unsere Dependency. Durch sie wird die Effekt-Funktion nur aufgerufen
  // wenn sich auch der `value` geändert hat.
  }, [value]);
  
  return (
    <div>
      <p>Zählerwert: {value}</p>
      <button onClick={() => setValue(value + 1)}>
        +1
      </button>
    </div>
  );
}

ReactDOM.render(<Counter />, document.getElementById('root'));
```

Da der **Hook** sich stets _innerhalb_ der Funktion befindet hat er, ähnlich wie die Lifecycle-Methoden in Klassen-Komponenten, den vollen Zugriff auf die **Props** und den **State** der Komponente. Wobei der State in der **Function Component** selbst natürlich wiederum nur ein Hook ist, nämlich der `useState()`-Hook.

Durch die Verwendung des `useEffect()`-**Hooks** können wir hier die Komplexität verringern, da die Komponente nicht viele, zum Teil sehr ähnliche Dinge an mehreren Stellen innerhalb der Komponente ausführen muss, sondern sich alle für die Komponente relevanten Lifecycle-Events in nur einer einzigen Funktion, eben dem **Hook**, abspielen.

Zum Vergleich dazu möchte ich einmal ein Äquivalent zeigen wie der oben gezeigte `useEffect()`-**Hook** mit einer Klassen-Komponente implementiert werden würde.

```jsx
import React from "react";
import ReactDOM from "react-dom";

const defaultTitle = "React mit Hooks";

class Counter extends React.Component {
  state = {
    value: 0,
  };

  componentDidMount() {
    document.title = `Der Button wurde ${this.state.value} mal geklickt`;
  }

  componentDidUpdate(prevProps, prevState) {
    if (prevState.value !== this.state.value) {
      document.title = `Der Button wurde ${this.state.value} mal geklickt`;
    }
  }

  componentWillUnmount() {
    document.title = defaultTitle;
  }

  render() {
    return (
      <div>
        <p>Zählerwert: {this.state.value}</p>
        <button
          onClick={() => {
            this.setState(state => ({ value: state.value + 1 }));
          }}
        >
          +1
        </button>
      </div>
    );
  }
}

ReactDOM.render(<Counter />, document.getElementById("root"));
```

Hier kann natürlich darüber diskutiert werden, dass man den Aufruf um den `document.title` zu ändern in eine eigene Klassen-Methode wie etwa `setDocumentTitle()` auslagert, das sind allerdings Details die an der höheren Komplexität der Klassen-Komponente im direkten Vergleich mit **Hooks** nicht wirklich etwas ändern. 

Auch dann müsste noch immer zweimal die gleiche \(nun abstrahierte\) Funktion an zwei verschiedenen Stellen \(nämlich `componentDidMount()` und `componentDidUpdate()`\) aufgerufen werden. Zusätzlich hätten wir eine weitere Klassen-Methode, die die Klasse nur noch weiter aufbläht und so die Duplikation nur zu Lasten einer Abstraktion auflöst.

### Zugriff auf Context mit useContext\(\)

Der dritte Basic Hook im Bunde ist `useContext()`. Mit ihm wird es ein Kinderspiel Daten aus einem Context-Provider zu konsumieren, ohne dass dazu umständlich eine Provider-Komponente mit einer Function as a Child verwendet werden muss. 

Dazu erhält der `useContext()`-Hook einen mittels `React.createContext()` erzeugten Context übergeben und gibt dann den Wert des in der Komponenten-Hierarchie nächsthöheren Providers zurück. Wird Wert des Contexts im Provider geändert, löst der `useContext()`-Hook ein Rerendering aus mit den aktualisierten Daten aus dem Provider. Damit ist über die Funktionsweise auch schon alles gesagt. 

Der Hook ist tatsächlich eher simpler Natur:

```jsx
import React, { useContext } from "react";
import ReactDOM from "react-dom";

const AccountContext = React.createContext({});

const ContextExample = () => {
  const accountData = useContext(AccountContext);

  return (
    <div>
      <p>Name: {accountData.name}</p>
      <p>Rolle: {accountData.role}</p>
    </div>
  );
};

const App = () => (
  <AccountContext.Provider value={{ name: "Manuel", role: "admin" }}>
    <ContextExample />
  </AccountContext.Provider>
);

ReactDOM.render(<App />, document.getElementById("root"));
```

Hier sehen wir, wie die `ContextExample`-Komponente Daten \(in diesem Beispiel pseudo Account-Daten\) aus dem `AccountContext`-Provider verwendet, ohne dass dafür alles von einer `AccountContext.Consumer`-Komponente umschlossen sein muss. Dies spart nicht nur einige Zeilen Code in der Komponente selbst sondern führt auch in der Debug-Ansicht zu einem deutlich übersichtlicheren Baum, da die Verschachtelungstiefe flacher ist.

Diese Vereinfachung ist aber völlig optional und wer mag kann auch weiterhin die bekannte Consumer-Komponente für den Zugriff auf Daten aus einem Context-Provider verwenden.

