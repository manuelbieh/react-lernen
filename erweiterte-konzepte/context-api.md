# Context API

Die **Context API** wurde in React sehr lange Zeit eher stiefmütterlich behandelt und wurde erst einmal nur prototypisch implementiert und als experimentell bezeichnet ehe sie in React **16.3.** in grundlegend überarbeiteter Form offizieller Teil von React wurde. 

Sie wurde dafür konzipiert, um Daten innerhalb einer Komponenten-Hierarchie an sog. _Konsumenten_ zu verteilen ohne dabei Props an jede einzelne Komponente explizit übergeben zu müssen. Dies kann in einigen Fällen sehr mühsam sein wenn es sich um Daten handelt die von vielen Komponenten innerhalb des Baumes gemeinsam immer wieder verwendet werden. Dazu gehören bspw. Sprach-Einstellungen oder ein globales Style-Schema \(„Theme“\).

In solchen Fällen bietet sich die **Context API** an, die jeweils aus einem **Context** _**Provider**_ und beliebig vielen **Context** _**Consumern**_ besteht. Der **Provider** dient hier als eine Art zentrale Instanz für die jeweilige Datenstruktur, der **Consumer** kann dann die entsprechenden Daten _konsumieren_, die vom Provider bereitgestellt werden. Sozusagen eine „semi-globale“ Dateninstanz, die nur für einen bestimmten Baum innerhalb der Komponenten-Hierarchie gilt.

Die Datenstruktur kann dabei durchaus auch sehr komplex sein und ist nicht bspw. auch einfache Datentypen wie Strings oder Arrays beschränkt. Eine Anwendung kann dabei auch beliebig viele Contexts besitzen \(bspw. einen für die vom Benutzer eingestellte Sprache, einen für das Style-Schema, etc.\) und auch ein Provider selbst kann mit wechselnden Werten mehrfach verwendet werden. Aber eins nach dem anderen.

### API

Zur Erstellung eines neuen **Contexts** stellt React die Methode `createContext` bereit:

```jsx
const LanguageContext = React.createContext(defaultValue);
```

Mit lediglich dieser einen Zeile haben wir bereits einen neuen **Context** erstellt. Der **Context** besteht nun aus einer **Provider-** und eine **Consumer-**Komponente: `LanguageContext.Provider` sowie `LanguageContext.Consumer`.

Innerhalb unserer Anwendung kann der **Context** nun genutzt werden, indem ein bestimmter Baum von einem Provider umschlossen wird:

```jsx
// LanguageContext.js
import React from 'react';
const LanguageContext = React.createContext('de');
export default LanguageContext;
```

```jsx
// index.js
import React from 'react';
import ReactDOM from 'react-dom';
import LanguageContext from './LanguageContext';

const App = () => (
  <LanguageContext.Provider value={'en'}>
    {/* innerhalb dieses Baums steht uns nun der Wert 'de' zur Verfügung */}
  </LanguageContext.Provider>
);

ReactDOM.render(<App />, document.getElementById('#root'));
```

Möchten wir in einer Komponente nun auf den Wert des **Contexts** zugreifen, umschließen wir eine Komponente und machen uns das **Function as a Child** Prinzip das wir uns im vorherigen Kapitel angeschaut haben zu nutze:

```jsx
// DisplaySelectedLanguage.js
import React from 'react';
import LanguageContext from './LanguageContext';

const DisplaySelectedLanguage = () => (
  <LanguageContext.Consumer>
    {(value) => (<p>Die ausgewählte Sprache ist {value}</p>)}
  </LanguageContext.Consumer>
);

export default DisplaySelectedLanguage;
```

Wir können nun an einer beliebigen Stelle innerhalb unserer Anwendung die `SelectedLanguage` Komponente verwenden und haben dort immer den jeweils vom Provider bereitgestellten Wert verfügbar. Ändert sich der Wert im **Provider** werden auch alle **Consumer-**Komponenten unterhalb des entsprechenden Providers mit dem aktualisierten Wert neu gerendert!

Ein vollständiges, wenn auch recht konstruiertes Beispiel kann dann wie folgt aussehen:

```jsx
import React from 'react';
import ReactDOM from 'react-dom';
import LanguageContext from './LanguageContext';
import DisplaySelectedLanguage from './DisplaySelectedLanguage';

const App = () => (
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

ReactDOM.render(<App />, document.getElementById('#root'));
```

Obwohl wir keinerlei **Props** an die `DisplaySelectedLanguage`-Komponente übergeben hat diese dennoch Kenntnis von der aktuell ausgewählten Sprache und zeigt korrekt an: 

```markup
<p>Die ausgewählte Sprache ist en</p>
```

Ändert sich der `value` einer Provider-Komponente werden alle Consumer-Komponenten die sich innerhalb dieses Providers befinden neu gerendert!

Erweitern wir das Beispiel können wir uns so eine relativ simple kleine Komponentenstruktur aufbauen, um bspw. Mehrsprachigkeit in einer Anwendung zu realisieren. 

Im folgenden Beispiel legen wir ein Objekt mit Übersetzungen an und geben ein relativ umfangreiches Objekt mit verschiedenen Datentypen \(bestehend aus einem Array, einem String, einer Funktion um die Sprache zu wechseln und einem Objekt mit den eigentlichen Übersetzungen\) in den Context herein:

```jsx
import React from "react";
import ReactDOM from "react-dom";

const translationStore = {
  de: {
    greeting: "Guten Tag!",
    headline: "Heute lernen wir, wie Context funktioniert.",
  },
  en: {
    greeting: "Good day!",
    headline: "Today we learn how context works.",
  },
};

const defaultLanguage = "de";

const defaultLanguageContextValue = {
  availableLanguages: Object.keys(translationStore),
  changeLanguage: () => {
    console.warn('Funktion changeLanguage() nicht implementiert!');
  },
  language: defaultLanguage,
  translations: translationStore[defaultLanguage],
};

const LanguageContext = React.createContext(defaultLanguageContextValue);

class Localized extends React.Component {
  changeLanguage = (newLanguage) => {
    this.setState((state) => ({
      translations: translationStore[newLanguage],
      language: newLanguage,
    }));
  };

  state = {
    ...defaultLanguageContextValue,
    changeLanguage: this.changeLanguage,
  };

  render() {
    return (
      <LanguageContext.Provider value={this.state}>
        {this.props.children}
      </LanguageContext.Provider>
    );
  }
}

const Greeting = () => (
  <LanguageContext.Consumer>
    {(contextValue) => contextValue.translations.greeting}
  </LanguageContext.Consumer>
);

const Headline = () => (
  <LanguageContext.Consumer>
      {(contextValue) => contextValue.translations.headline}
  </LanguageContext.Consumer>
);

const LanguageSelector = () => {
  return (
    <LanguageContext.Consumer>
      {(contextValue) => (
        <select
          onChange={(event) => {
            contextValue.changeLanguage(event.target.value);
          }}
        >
          {contextValue.availableLanguages.map((language) => (
            <option value={language}>{language}</option>
          ))}
        </select>
      )}
    </LanguageContext.Consumer>
  );
};

const App = () => (
  <Localized>
    <LanguageSelector />
    <p><Greeting /></p>
    <p><Headline /></p>
  </Localized>
);

ReactDOM.render(<App />, document.getElementById("root"));
```

Zuerst definieren wir ein Objekt `defaultLanguageContextValue` welches den Default-Wert unseres neuen Context-Objekts darstellt. Dieses besteht aus:

* einem Objekt `translationStore`, welches alle vorhandenen Übersetzungen beinhaltet. 
* einer Standardsprache Deutsch \(`de`\) , die in der `language` Eigenschaft gespeichert wird
* einem Array \(`availableLanguages`\) mit allen verfügbaren Sprachen aus dem `translationStore`-Objekt, den wir mittels `Object.keys()` dynamisch aus den Eigenschaften auf erster Ebene erzeugen \(in unserem Beispiel also `['de', 'en']`\).
* einer Platzhalterfunktion \(`changeLanguage()`\), die später in der `Localized`-Komponente durch eine echte Implementierung ersetzt wird. Dies dient dazu, dass wir bei inkorrekter Benutzung des Contexts nicht Gefahr laufen eine Funktion aufzurufen die noch gar nicht existiert. In diesem Fall würde die Warnung ausgegeben werden _„Funktion changeLanguage\(\) nicht implementiert!“_.

Die `changeLanguage()`-Funktion kann erst später in der Komponente selbst implementiert werden, da React ansonsten keine Möglichkeit hätte mit Bordmitteln den State \(also in diesem konkreten Fall die Sprache und die Übersetzungen\) zu ändern, da State für React nur **innerhalb einer Komponente** existiert. So könnten wir zwar die aktuelle Spracheinstellung bspw. in einer globalen Variable speichern, React würde die Komponente dann bei einer Änderung aber nicht neu rendern, da sich weder Props noch State geändert haben, dies aber eine Bedingung ist, um React den Seitenbaum neu rendern zu lassen.

Die `Localized`-Komponente dient nun als Wrapper-Komponente für unseren neu erstellten Context. In ihr speichern \(und ändern!\) wir die vom Benutzer ausgewählte Sprache, indem wir den State entsprechend setzen. Wir speichern dazu das `defaultLanguageContextValue` Objekt im State der Komponente und implementieren hier zusätzlich die `changeLanguage()`-Methode. Diese empfängt eine Sprache \(also `de` oder `en`\), modifiziert den State entsprechend und holt sich die Übersetzungen für die neu ausgewählte Sprache aus dem `translationStore` Objekt und schreibt diese als `translations` neu in den State. Wechselt der User bspw. die Sprache von Deutsch \(Voreinstellung\) zu Englisch, überschreibt die Funktion alle deutschen Übersetzungen im State mit den englischen Übersetzungen. Durch den Aufruf von `this.setState()` wird ein Re-Rendering ausgelöst und alle Context-Consumer innerhalb des Komponenten-Baums werden mit dem aktualisierten Wert, den wir in der render\(\)-Methode der Komponente an den Context-Provider übergeben, neu gerendert. 

Das ganze klingt jetzt erstmal kompliziert, ist in der Praxis aber tatsächlich gar nicht so schwierig. Ich möchte an dieser Stelle daher dringend dazu ermutigen das obige Beispiel einmal live auszuprobieren.

Übrigens gibt es in der obigen Komponente einen Fallstrick: für gewöhnlich wird der State in einer Klassen-Komponente als erstes definiert und erst danach alle anderen Eigenschaften und Methoden der Klasse. In diesem Fall sind wir jedoch von der Konvention abgewichen und haben die `changeLanguage()`-Methode zuerst implementiert. Dies hat den einfachen Grund, dass `this.changeLanguage` ansonsten noch gar nicht definiert, also `undefined` wäre. Um dies zu umgehen, definieren wir die Methode bevor wir die `state`-Eigenschaft der Klasse konstruieren.

Nun ist unser Code-Beispiel noch relativ komplex und komplexer als er tatsächlich sein müsste. Und so haben wir hier für die Headline und den Grußtext jeweils eine eigene Komponente erstellt nur um in dieser jeweils einen Context-Consumer zu verwenden, um in diesem wiederum Zugriff auf das Objekt mit unseren Übersetzungen zu haben. Hier lässt sich der Code gleich so optimieren, dass wir eine generische Komponente erstellen um mit dieser direkt auf bestimmte Übersetzungen aus dem `translations`-Objekt zugreifen zu können. Wir nennen diese Komponente Translated und als einzige Prop erhält sie die Eigenschaft auf die wir im `translations`-Objekt zugreifen wollen. In unserem konkreten Beispiel kann das also `greeting` oder `headline` sein.

```jsx
const Translated = ({ translationKey }) => (
  <LanguageContext.Consumer>
    {(contextValue) => contextValue.translations[translationKey]}
  </LanguageContext.Consumer>
)
```

Unsere `App`-Komponente sieht dann entsprechend so aus:

```jsx
const App = () => (
  <Localized>
    <LanguageSelector />
    <p><Translated translationKey="greeting" /></p>
    <p><Translated translationKey="headline" /></p>
  </Localized>
);
```

Die `Headline` und `Greeting` Komponenten aus dem vorherigen Beispiel können wir uns dann einfach sparen.

**Achtung:** gerade bei Übersetzungen ist es nicht unüblich die Schlüssel für die Übersetzungen einfach kurz „key“ zu nennen und so hätte es durchaus einen gewissen Charme auch die Prop in der `Translated`-Komponente `key` zu nennen. So wäre das doch schön kurz und gut lesbar:

```jsx
<Translated key="greeting" />
```

Hier macht uns React aber einen Strich durch die Rechnung, da `key` ein reservierter Name in JSX ist und zur Identifizierung von Elementen dient wenn diese Elemente in einem Array verwendet werden. Die genauen Gründe dazu sind im Kapitel über „Listen, Refs, Fragments und Conditional Rendering“ in „Die Grundlagen“ nachzulesen, dort konkret im Abschnitt „Listen“.

### Verwendung mehrerer Contexts

Es ist kein Problem auch mehrere Context-Provider innerhalb einer Komponenten-Hierarchie zu haben. Das Verschachteln von mehreren Provider-Komponenten ist also kein Problem. Selbst Provider vom selben Context-Typen können ineinander verschachtelt werden. Dabei wird den Consumer-Komponenten stets der Context-Value des nächst höheren Providers übergeben:

```jsx
<MyContext.Provider value="1">
  <MyContext.Provider value="2">
    <MyContext.Consumer>
      {(value) => <p>Der Wert ist {value}</p>}
    </MyContext.Consumer>
  </MyContext.Provider>
</MyContext.Provider>
```

Das obige Beispiel wäre also problemlos möglich. Die Ausgabe wäre hier:

```markup
<p>Der Wert ist 2</p>
```

Die **Consumer-Komponente** bezieht ihre daten hier nach aus dem nächsthöheren **Context-Provider**, dieser hat im obigen Beispiel den Wert "2". 

Ergibt das Verschachteln von gleichen **Context-Providern** meist relativ wenig Sinn so ist es dennoch keine unübliche oder gar schlechte Praktik verschiedene **Context-Provider** ineinander zu verschachteln. So kann eine Anwendung sehr einfach aus einem Theme-Provider, einem Language-Provider und einem Account-Provider bestehen. Letzterer würde sich dann bspw. um das Daten-Handling des eingeloggten Benutzers kümmern, ggf. Access Tokens oder benutzerspezifische Einstellungen verwalten.

### Abkürzung: contextType

Bei der Verwendung von Klassen-Komponenten können wir uns eines Tricks bedienen und auf die Verwendung einer Consumer-Komponente verzichten die unseren Komponenten-Baum weiter aufbläht. 

Der `contextType` ist hier das Stichwort. Dieser kann einer Klassen-Komponente in Form einer gleichnamigen statischen Eigenschaft zugewiesen werden, anschließend kann dann innerhalb der Komponente mittels `this.context` auf den Wert des jeweiligen Contexts zugegriffen werden. Als Wert bekommt die `contextType`-Eigenschaft einen Context zugewiesen, der zuvor mittels `React.createContext()` erzeugt wurde.

Allerdings ist es nur möglich jeder Klasse lediglich **einen einzigen Context-Typen** zuzuweisen. Möchten wir auf den Wert zweier oder mehrerer Contexts müssen wir unser JSX wieder in Consumer-Komponenten wrappen. Bei der Verwendung der _Public Class Fields Syntax_ aus ES2015+ reicht es dazu eine statische Klassen-Eigenschaft `contextType` zu definieren und dieser einen Context zuzuweisen. 

Als Beispiel, angewendet auf die Translated-Komponente von weiter oben, sähe das dann etwa so aus:

```jsx
class Translated extends React.Component {
  static contextType = LanguageContext;
  render() {
    return this.context.translations[this.props.translationKey];
  }
}
```

Wir weisen der statischen `contextType`-Eigenschaft der Komponente \(die nun eine Klassen-Komponente und keine Function-Component mehr ist\) als Wert unseren `LanguageContext` zu und schon haben wir in `this.context` den Wert des Contexts zur Verfügung.

Ohne die Verwendung der Public Class Fields Syntax \(die ich einige Kapitel vorher allerdings dringend empfohlen habe, da sie uns das Leben an einigen Stellen einfacher macht\) sähe der gleiche Code dann folgendermaßen aus:

```jsx
class Translated extends React.Component {
  render() {
    return this.context.translations[this.props.translationKey];
  }
}

Translated.contextType = LanguageContext;
```

Wir würden den `contextType` also außerhalb der Komponente definieren und nicht mehr innerhalb. Im Grunde genommen ist das aber letztendlich Geschmackssache und hat sonst keine Implikationen oder Nachteile. Es ist allein eine andere Syntax-Variante, die erst in späteren ECMAScript-Versionen oder eben durch Transpiling mittels Babel möglich ist. Möglich gemacht wird sie durch das Babel-Plugin `@babel/plugin-proposal-class-properties`.

### Performance Fallstrick

React optimiert **Context** unter der Haube massiv um unnötiges Re-Rendering von Komponenten oder gar  ganzen Komponenten-Hierarchien bestmöglich zu vermeiden. Zu diesem Zweck wird bei jedem Rendering der alte Context-Wert des Providers mit dem neuen Wert verglichen und **Consumer-Komponenten** anschließend nur dann neu gerendert wenn sich der Wert ihres **Context-Providers** geändert hat.

Was in der Theorie ausnahmsweise mal relativ einfach klingt, bringt aber einen kleinen Stolperstein mit sich. Und zwar betrifft das Consumer-Provider, deren Value innerhalb der `render()`-Methode einer Komponente stets neu on-the-fly erzeugt werden. Daher empfiehlt es sich normalerweise den Context-Wert außerhalb der render\(\)-Methode zu erzeugen und eine _Referenz zum Wert_ statt eines _stets neu erzeugten Werts_ zu übergeben.

Um dies zu verdeutlichen hier vorab ein Negativ-Beispiel:

```jsx
class App extends React.Component {
  state = {
    color: 'red',
  };

  render() {
    <Provider value={{color: this.state.color}}>
      <MoreComponents />
    </Provider>
  }
}
```

In diesem Fall erzeugen wir bei jedem neuen Aufruf der render\(\)-Methode ein neues Objekt `{color: this.state.color}` welches wir als Wert für den Context-Provider benutzen. Da React lediglich überprüft ob die Referenz zum entsprechenden `value` im aktuellen `render()`-Aufruf der aus dem vorherigen `render()`-Aufruf entspricht, dies hier jedoch niemals der Fall ist, da ja ein neues Objekt an Ort und Stelle erzeugt wird, werden hier sämtliche Consumer-Komponenten neu gerendert.

Das obige Beispiel lässt sich jedoch sehr einfach so umschreiben, dass die Performance-Optimierungen von React hier greifen:

```jsx
class App extends React.Component {
  state = {
    color: 'red',
  };

  render() {
    <Provider value={this.state}>
      <MoreComponents />
    </Provider>
  }
}
```

Im zweiten Beispiel übergeben wir lediglich eine _Referenz_ zum `state`-Objekt der Komponente. Da diese auch beim Re-Rendering der Komponente erhalten bleibt, löst dieses Vorgehen kein Re-Rendering aus, solange sich der Inhalt des States der Komponente nicht ändert!

