# Context API

Die **Context API** wurde in React sehr lange Zeit eher stiefmütterlich behandelt und wurde erst einmal nur prototypisch implementiert und als experimentell bezeichnet ehe sie in React **16.3.** in grundlegend überarbeiteter Form offizieller Teil von React wurde. 

Sie wurde dafür konzipiert, um Daten innerhalb einer Komponenten-Hierarchie in einer Anwendung an sog. _Konsumenten_ zu verteilen ohne dabei Props an jede einzelne Komponente explizit übergeben zu müssen. Dies kann in einigen Fällen sehr mühsam sein wenn es sich um Daten handelt die von vielen Komponenten innerhalb des Baumes gemeinsam immer wieder verwendet werden. Dazu gehören bspw. Sprach-Einstellungen oder ein globales Style-Schema \(„Theme“\).

In solchen Fällen bietet sich die **Context API** an, die jeweils einem **Context** _**Provider**_ und beliebig vielen **Context** _**Consumern**_ besteht. Der **Provider** dient hier als eine Art zentrale Instanz für die jeweiligen Datenstruktur, der **Consumer** kann dann die entsprechenden Daten _konsumieren_, die vom Provider bereitgestellt werden. Sozusagen eine semi-globale Dateninstanz die nur für einen bestimmten Baum innerhalb der Komponenten-Hierarchie gilt.

Die Datenstruktur kann dabei durchaus auch sehr komplex sein und ist nicht bspw. auch einfache Datentypen wie Strings oder Arrays beschränkt. Eine Anwendung kann dabei auch beliebig viele Kontexte besitzen \(bspw. einen für die vom Benutzer eingestellte Sprache, einen für das Style-Schema, etc.\) und auch ein Provider selbst kann mit wechselnden Werten mehrfach verwendet werden. Aber eins nach dem anderen.

### API

Zur Erstellung eines neuen **Contexts** stellt React die Methode `createContext` bereit:

```jsx
const ExampleContext = React.createContext(defaultValue);
```

Mit lediglich dieser einen Zeile haben wir bereits einen neuen **Context** erstellt. Der **Context** besteht nun aus einer **Provider-** und eine **Consumer-**Komponente: `LanguageContext.Provider` sowie `LanguageContext.Consumer`.

Innerhalb unserer Anwendung kann der **Context** nun genutzt werden, indem ein bestimmter Baum von einem Provider umschlossen wird:

{% code-tabs %}
{% code-tabs-item title="LanguageContext.js" %}
```jsx
import React from 'react';


const LanguageContext = React.createContext('de');

export default LanguageContext;
```
{% endcode-tabs-item %}
{% endcode-tabs %}

```jsx
import React from 'react';
import ReactDOM from 'react-dom';
import LanguageContext from './LanguageContext';

const App = () => {
  return (
    <LanguageContext.Provider value={'en'}>
      {/* innerhalb dieses Baums steht uns nun der Wert 'de' zur Verfügung */}
    </LanguageContext.Provider>
  );
}

ReactDOM.render(
  <App />, 
  document.getElementById('#root')
);
```

Möchten wir in einer Komponente nun auf den Wert des **Contexts** zugreifen, umschließen wir eine Komponente und machen uns das **Function as a Child** Prinzip aus dem vorherigen Kapitel zu nutze:

{% code-tabs %}
{% code-tabs-item title="DisplaySelectedLanguage.js" %}
```jsx
import React from 'react';
import LanguageContext from './LanguageContext';

const DisplaySelectedLanguage = () => {
  return (
    <LanguageContext.Consumer>
      {(value) => (<p>Die ausgewählte Sprache ist {value}</p>)}
    </LanguageContext.Consumer>
  );
}

export default DisplaySelectedLanguage;
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Wir können nun an einer beliebigen Stelle innerhalb unserer Anwendung die `SelectedLanguage` Komponente verwenden und haben dort immer den jeweils vom Provider bereitgestellten Wert verfügbar. Ändert sich der Wert im **Provider** werden auch alle **Consumer-**Komponenten unterhalb des entsprechenden Providers mit dem aktualisierten Wert neu gerendert!

Ein vollständiges, wenn auch recht konstruiertes Beispiel kann dann wie folgt aussehen:

```jsx
import React from 'react';
import ReactDOM from 'react-dom';
import LanguageContext from './LanguageContext';
import DisplaySelectedLanguage from './DisplaySelectedLanguage';

const App = () => {
  return (
    <LanguageContext.Provider value="en">
      <header>Herzlich willkommen</header>
      <div className="content">
        <div className="sidebar"></div>
        <div className="mainContent">
          <DisplaySelectedLanguage />
        </div>
      </div>
      <footer>© 2019</footer>
    </LanguageContext.Provider>
  );
}

ReactDOM.render(
  <App />, 
  document.getElementById('#root')
);
```

Obwohl wir keinerlei **Props** an die `DisplaySelectedLanguage`-Komponente übergeben hat diese dennoch Kenntnis von der aktuell ausgewählten Sprache und zeigt korrekt an: 

```markup
<p>Die ausgewählte Sprache ist en</p>
```



