# Grundsätze und Regeln von Hooks

Die Verwendung von **Hooks** ist an einige Grundsätze und Regeln geknüpft die es zu beachten gilt damit es nicht zu unangenehmen Überraschungen und unerwartete Bugs führt, von denen es mit Hooks ja gerade weniger geben soll. Bei der Einhaltung dieser Regeln hilft uns **ESLint** und das vom React-Team höchstpersönlich entwickelte ESLint-Plugin `eslint-plugin-react-hooks`. Die Verwendung von ESLint hatte ich im Kapitel über **Tools und Setup** bereits empfohlen und an dieser Empfehlung hat sich seitdem auch nichts geändert.

Um das Plugin im Projekt zu installieren hilft folgender Aufruf auf der Kommandozeile:

```bash
npm install --save-dev eslint-plugin-react-hooks
```

bzw. mit Yarn:

```bash
yarn add --dev eslint-plugin-react-hooks
```

Anschließend muss die `.eslintrc` wie folgt angepasst werden:

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

Für diejenigen unter euch die **Create React App** nutzen habe eine gute Nachricht: ihr braucht nichts weiter tun, die beiden Regeln kommen standardmäßig in den aktuellen Versionen von Create React App automatisch mit in euer Setup!

### Die Regeln von Hooks

Die Formalitäten hätten wir nun geklärt, doch nun erst mal der Reihe nach. Was sind also diese sagenumwobenen Regeln die ich eingangs erwähnt habe und für die die es sogar eigens entwickelte ESLint-Regeln gibt?

#### Hooks können nur in React Function Components verwendet werden!

Nicht in Klassen-Komponenten und auch nirgendwo sonst: React Hooks können ausschließlich innerhalb von React Function Components verwendet werden! Dies bedeutet das eine Funktion die Hooks verwendet unweigerlich auch einen Rückgabewert haben muss, der ein korrektes React-Element ist. Also JSX, Arrays, Strings oder `null`.

#### Hooks dürfen nur auf oberster Ebene innerhalb ihrer Function Component verwendet werden!

Konkret bedeutet das, dass es **nicht möglich** ist Hooks innerhalb von **Schleifen, Bedingungen oder verschachtelten Funktionen** zu benutzen. Dies hängt damit zusammen, wie React Hooks intern verarbeitet werden. Hier ist es wichtig dass die Reihenfolge in der Hooks ausgeführt werden bei jedem Re-Rendering einer Komponente identisch sein muss. Aus diesem Grund ist es bspw. nicht möglich einen Hook nur dann aufzurufen wenn eine bestimmte Bedingung erfüllt ist. Dies würde je nachdem ob die Bedingung erfüllt ist eben zu einer anderen Reihenfolge beim Abarbeiten der Hooks führen.



