# Hooks API

In diesem Kapitel möchte ich auf alle verfügbaren internen Hooks einmal eingehen, ihre genaue Verwendung und ihre Einsatzbereiche beschreiben. In der React Dokumentation werden diese unterteilt in drei generelle \(_basic_\) und sieben zusätzliche \(_additional_\) Hooks. Die zusätzlichen Hooks sind insbesondere für spezielle Anwendungsfälle \(bspw. Performance-Optimierung\) gedacht oder sie sind Abwandlungen der generellen Hooks.

Zu den drei **generellen Hooks** gehören die bereits in den vorherigen Kapiteln vorgestellten Hooks:

* `useState`
* `useEffect`
* `useContext`

Die sieben **weiteren Hooks** sind:

* `useReducer` 
* `useCallback` 
* `useMemo` 
* `useRef` 
* `useImperativeHandle` 
* `useLayoutEffect` 
* `useDebugValue`

## useState

```javascript
const [state, setState] = useState(initialState);
```

Dieser Hook gibt uns einen **Wert** zurück und eine **Funktion** um selbigen zu aktualisieren. Während des ersten Renderings einer Komponente, die den `useState()`-Hook nutzt, entspricht dieser Wert dem als `initialState` übergebenen Parameter. Ist der übergebene Parameter eine Funktion nutzt React den Rückgabewert der Funktion als initialen Wert.

Bei der Update-Funktion stellt React sicher, dass diese stets die selbe **Identität** hat, erstellt die Funktion als nicht bei jedem Aufruf des Hooks neu. Dies ist wichtig, um unnötige Re-Renderings zu verhindern und führt außerdem dazu, dass sie nicht als **Dependency** an andere **Hooks** wie `useEffect()` oder `useCallback()` übergeben werden muss.

Ganz konkret gibt `useState()` ein **Array** zurück in dem das erste Element immer der **Wert** des States ist und das zweite Element immer eine **Funktion**, um diesen Wert zu aktualisieren. Für die Benennung von Wert und Funktion gibt es durch die Array Destructuring Syntax keine formellen Einschränkungen. Jedoch hat es sich schnell herauskristallisiert, dass in den deutlich überwiegenden Fällen die Form `value`/`setValue` eingehalten werden. Also etwa `user` und `setUser`. Aber auch andere Namen wie `changeUser` oder `updateUserState` wären natürlich denkbar.

Der Mechanismus zum Aktualisieren des States funktioniert hier ansatzweise ähnlich wie `this.setState()` in Klassen-Komponenten. So kann der Funktion entweder ein **neuer Wert** übergeben werden, der dann an die Stelle des alten Werts tritt oder eine **Updater-Funktion**. Diese bekommt den letzten Wert übergeben und nutzt den **Rückgabewert** aus der Funktion als neuen State.

Doch Achtung, es gibt einen entscheidenden Unterschied: anders als bei `this.setState()` werden Objekte **nicht zusammengeführt** mit dem bestehenden State, ****sondern der State wird **komplett** durch den neuen State **ersetzt!**

Um diesen Unterschied zu illustrieren schauen wir uns einmal den direkten Vergleich an:

```jsx
import React, { useState, useEffect } from "react";
import ReactDOM from "react-dom";

class StateClass extends React.Component {
  state = { a: 1, b: 2 };

  componentDidMount() {
    this.setState({ c: 3 });
    this.setState(() => {
      return { d: 4 };
    });
  }

  render() {
    // { a: 1, b: 2, c: 3, d: 4 }
    return <pre>{JSON.stringify(this.state, null, 2)}</pre>;
  }
}

const StateHook = () => {
  const [example, setExample] = useState({ a: 1, b: 2 });

  useEffect(() => {
    setExample({ c: 3 });
    setExample(() => {
      return { d: 4 };
    });
  }, []);

  // { d: 4 }
  return <pre>{JSON.stringify(example, null, 2)}</pre>;
};

const App = () => {
  return (
    <>
      <StateClass />
      <StateHook />
    </>
  );
};

ReactDOM.render(<App />, document.getElementById("root"));
```

Während die `StateClass` die Daten aller Aufrufe von `this.setState()` sammelt und mit dem bestehenden State zusammenführt, überschreibt die `setState()`-Funktion im `StateHook` jeweils den kompletten State und ersetzt den kompletten alten Wert mit dem neuen Wert. Die Ausgabe ist also in der **Klassen-Komponente**: `{a: 1, b: 2, c: 3, d: 4}` und in der **Function Component** mit dem Hook ein einfaches `{d: 4}`, da dieses zuletzt in den State geschrieben wurde.

Gibt die Update-Funktion als neuen State exakt gleichen Wert zurück wie den vorherigen kommt das dem Abbruch eines State-Updates gleich und es wird kein Rerendering ausgelöst und auch keine Seiten-Effekte ausgelöst.

## useEffect

```javascript
useEffect(effectFunction, dependenciesArray);
```

Dieser **Hook** ist für **imperative Seiteneffekte** vorgesehen, wie etwa API Requests, Timer oder globale Event-Listener. Diese sind innerhalb von **Function Components** grundsätzlich nicht erlaubt oder zumindest zu vermeiden, da sie zu nicht nachvollziehbarem Verhalten und möglicherweise zu schwer zu behebenden Bugs führen können und werden. 

Der `useEffect()`-**Hook** schafft hier Abhilfe und erlaubt die _sichere_ Verwendung von Seiten-Effekten auch innerhalb einer **Function Component**. 

Der **Hook** erwartet eine **Funktion** als ersten Parameter und optional ein **Dependency Array** als zweiten Parameter. Die Funktion wird aufgerufen **nachdem** eine Komponente gerendert wurde. Wird ein optionales **Dependency Array** übergeben, wird die Funktion nur dann ausgeführt wenn sich mindestens einer der Werte aus dem **Dependency Array** seit der letzten Ausführung der Funktion geändert hat. Wird ein leeres **Dependency Array** übergeben, also `[]`, wird die Funktion nur beim **ersten** Rendering der Komponente aufgerufen, vergleichbar also mit der `componentDidMount()` **Lifecycle Methode**, die wir bereits aus Klassen-Komponenten kennen.

In einigen Fällen hinterlassen Seiteneffekte "Spuren", die wieder aufgeräumt werden müssen wenn eine Komponente nicht mehr verwendet wird. Werden beispielsweise Intervalle mittels `setInterval()` gestartet, sollten diese beim Entfernen der Komponente via `clearTimeout()` gestoppt werden, andernfalls kann es zu Problemen wie im schlimmsten Falle Memory Leaks führen. 

Auch global registrierte Event Listener wie `resize` oder `orientationchange` die dem `window`-Objekt mittels `addEventListener()` hinzugefügt werden, sollten spätestens beim Unmounting einer Komponente wieder durch `removeEventListener()` entfernt werden, damit der Code nicht mehr ausgeführt wird, wenn die Komponente sich gar nicht mehr im Seitenbaum befindet.

Zu diesem Zweck ist es möglich eine **Cleanup-Funktion** aus der **Effekt-Funktion** zurück zu geben. Gibt eine **Effekt-Funktion** eine **Cleanup-Funktion** zurück, wird diese vor jedem Aufruf der **Effekt-Funktion** ausgeführt, außer vor dem ersten Aufruf:

```javascript
import React, { useState, useEffect } from "react";
import ReactDOM from "react-dom";

const Clock = () => {
  const [time, setTime] = useState(new Date());
  
  useEffect(() => {
    const intervalId = setInterval(() => {
      setTime(new Date());
    }, 1000);
    return () => {
      clearInterval(intervalId);
    };
  }, []);

  return `${time.getHours()}:${time.getMinutes()}:${time.getSeconds()}`;
);

ReactDOM.render(<Clock />, document.getElementById("root"));
```

Im obigen Beispiel installieren wir einen Intervall beim Mounting der Komponente. Beim Unmounting der Komponente halten wir den Timer an, da wir sonst den State einer Komponente ändern würden, die sich nicht mehr im Seitenbaum befindet. Dies würde uns React mit einer Fehlermeldung quittieren, vor Memory Leaks warnen und uns darauf hinweisen, dass subscriptions und andere asnychrone Tasks in der Cleanup-Funktion aufgeräut werden müssen:

{% hint style="danger" %}
Warning: Can't perform a React state update on an unmounted component. This is a no-op, but it indicates a memory leak in your application. To fix, cancel all subscriptions and asynchronous tasks in a useEffect cleanup function.
{% endhint %}



