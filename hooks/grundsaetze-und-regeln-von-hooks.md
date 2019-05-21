# Grundsätze und Regeln von Hooks

Die Verwendung von **Hooks** ist an einige Grundsätze und Regeln geknüpft, die es zu beachten gilt, damit es nicht zu unangenehmen Überraschungen und unerwarteten Bugs kommt, von denen es mit **Hooks** ja gerade weniger geben soll. Bei der Einhaltung dieser Regeln hilft uns **ESLint** und das vom React-Team höchstpersönlich entwickelte ESLint-Plugin `eslint-plugin-react-hooks`. Die Verwendung von ESLint hatte ich im Kapitel über **Tools und Setup** bereits empfohlen und an dieser Empfehlung hat sich seitdem auch nichts geändert.

Um das Plugin im Projekt zu installieren, hilft folgender Aufruf auf der Kommandozeile:

```bash
npm install --save-dev eslint-plugin-react-hooks
```

bzw. mit Yarn:

```bash
yarn add --dev eslint-plugin-react-hooks
```

Anschließend muss die `.eslintrc` Konfiguration wie folgt angepasst werden:

```javascript
{
  "plugins": [
    // ...
    "react-hooks"
  ],
  "rules": {
    // ...
    "react-hooks/rules-of-hooks": "error",
    "react-hooks/exhaustive-deps": "warn"
  }
}
```

Für diejenigen unter euch, die **Create React App** nutzen, habe eine gute Nachricht: Ihr braucht nichts weiter tun, die beiden Regeln kommen standardmäßig in den aktuellen Versionen von **Create React App** automatisch mit in euer Setup!

### Die Regeln von Hooks

Die Formalitäten hätten wir nun geklärt, doch nun erst mal der Reihe nach. Was sind also diese sagenumwobenen Regeln, die ich eingangs erwähnt habe und für die die es sogar eigens entwickelte ESLint-Regeln gibt?

#### Hooks können nur in React Function Components verwendet werden!

Nicht in **Klassen-Komponenten** und auch nirgendwo sonst: **Hooks** können ausschließlich innerhalb von React **Function Components** verwendet werden! Dies bedeutet, dass eine Hooks verwendende Funktion eine React-Komponente sein muss, also unweigerlich auch einen Rückgabewert haben muss, der ein valides React-Element ist. Also JSX, Arrays, Strings oder `null`.

{% hint style="danger" %}
**Nicht erlaubt, da eine Klassen-Komponente verwendet wird:**
{% endhint %}

```jsx
class MyComponent extends React.Component {
  render() {
    const [value, setValue ] = useState();
    return (
      <input type="text" onChange={(e) => setValue(e.target.value)} />
    );
  }
}
```

{% hint style="success" %}
**Erlaubt, da eine Function Component verwendet wird:**
{% endhint %}

```jsx
const MyComponent = () => {
  const [value, setValue ] = useState();
  return (
    <input type="text" onChange={(e) => setValue(e.target.value)} />
  );
}
```

#### Hooks dürfen nur auf oberster Ebene innerhalb ihrer Function Component verwendet werden!

Konkret bedeutet das, dass es **nicht möglich** ist, Hooks innerhalb von **Schleifen, Bedingungen oder verschachtelten Funktionen** zu benutzen. Dies hängt damit zusammen, wie React **Hooks** intern verarbeitet. Hier ist es wichtig, dass die Reihenfolge, in der **Hooks** ausgeführt werden, bei jedem Re-Rendering einer Komponente identisch sein muss. Aus diesem Grund ist es bspw. nicht möglich, einen **Hook** nur dann aufzurufen, wenn eine bestimmte Bedingung erfüllt ist. Dies würde, je nachdem ob die Bedingung erfüllt ist, eben zu einer anderen Reihenfolge beim Abarbeiten der **Hooks** führen. Möglich ist es jedoch, Bedingungen _innerhalb_ von **Hooks** zu verwenden.

{% hint style="danger" %}
**Nicht erlaubt, da der Hook sich innerhalb einer Bedingung befindet:**
{% endhint %}

```javascript
if (title) {
  useEffect(() => {
    document.title = title;
  }, [title])
}
```

{% hint style="success" %}
**Erlaubt, da sich die Bedingung innerhalb des Hooks befindet:**
{% endhint %}

```javascript
useEffect(() => {
  if (title) {
    document.title = title;
  }
}, [title])
```

Wer das ESLint-Plugin installiert und die Regeln aktiviert, hat hier nichts zu befürchten. ESLint zeigt im Fall einer Missachtung dieser Regeln dann eine Warnung an und weist den Entwickler auf seinen Fehler hin.

