# Typechecking mit PropTypes, Flow und TypeScript

**Typechecking** ist eine einfache Möglichkeit um potentielle Fehler in einer Anwendung zu vermeiden. Das Prinzip dabei ist ganz einfach: Komponenten sollten „Pure“ sein, wie wir schon in der Einführung gelernt haben. Sie sollten also keine Seiten-Effekte auslösen und vor allem sollten sie bei den **gleichen Eingabeparametern** \(was im Fall von Komponenten die **Props** und deren daraus abgeleiteter **State** ist\) auch die **identische Ausgabe** erzeugen.

Das bedeutet, dass es möglichst vorhersehbar und sehr strikt sein sollte welche **Props** in eine Komponente hereingereicht werden können und welche von ihr verarbeitet werden. Um dies sicherzustellen können wir uns das sog. **Typechecking** zu nutze machen. JavaScript ist prinzipiell eine untypisierte Sprache. Eine Variable die mal ein **String** war, kann problemlos in eine **Number** oder gar ein **Object** umgewandelt werden, ohne dass der JavaScript-Interpreter ein Problem damit hat.

Auch wenn dies bei der Entwicklung mitunter sehr praktisch ist weil wir uns nicht festlegen müssen, öffnet das die Tür für einige ärgerliche Fehler und macht es daher nötig, regelmäßig manuell auf den korrekten Typen zu prüfen. Wollen wir bspw. auf eine tief verschachtelte Eigenschaft `user.settings.notifications.newMessages`zugreifen, sollten wir zuvor prüfen ob `user` überhaupt ein Objekt und nicht `null` ist, anschließend sollten wir prüfen ob das gleiche für `settings` zutrifft, usw. Andernfalls könnten wir es mit einem Type Error zu tun haben:

{% hint style="danger" %}
TypeError: Cannot read property 'settings' of undefined
{% endhint %}

**Typechecking** kann uns hier also helfen derartige potentielle Fehler schon vorher zu entdecken. Dazu gibt es neben **Flow** und **TypeScript**, die statische Typisierung ermöglichen, mit den sogenannten **PropTypes** auch eine recht simple React-eigene Lösung. Während **Flow** und **TypeScript** generell statische Typisierung in JavaScript ermöglichen, beschränken sich die React **PropTypes** allein auf React-Komponenten und finden außerhalb von Komponenten keine Anwendung. Wer also Gefallen findet an statischer Typisierung, sollte durchaus mal einen Blick auf **Flow** oder **TypeScript** wagen.

## PropTypes

**PropTypes** reichen zurück bis in ganz frühe Versionen von React, lange bevor React seine heutige Popularität erreicht hat und wurden in React 15.5. aus dem Core heraus und in ein eigenes `prop-types` Package ausgelagert. Während man seine **PropTypes** vorher mittels bspw. `React.PropTypes.string` direkt in der Core-Library definieren konnte, erfolgt der Zugriff nun über das zuvor importierte `PropTypes` Modul: `PropTypes.string`. 

Das bedeutet auch, dass das Modul zuerst als `devDependency` installiert werden muss. Auf der Kommandozeile reicht dafür ein simples:

```text
yarn add --dev prop-types
```

oder:

```text
npm install --save-dev prop-types
```

**PropTypes** dienen dabei als eine Art Interface und legen fest, welche Form bzw. welchen Typen eine Prop annehmen darf und ob diese optional ist oder erforderlich ist. Gibt es Abweichungen, weist uns React **im Development-Modus** darauf hin. Bei einer korrekt veröffentlichten Anwendung, die die Production-Version von React nutzt und/oder mit der Umgebungsvariable `process.env.NODE_ENV=production` gebaut wurde, bekommen wir diese Warnungen **nicht** mehr zu sehen!

Doch wie sieht die Verwendung von **PropTypes** nun aus? Hier müssen wir unterscheiden zwischen der **Class Component** und der **Stateless Functional Component**. 

Bei der **Class Component** sind die **PropTypes**  eine statische Eigenschaft `propTypes` der Komponente:

```jsx
import React from 'react';
import ReactDOM from 'react-dom';
import PropTypes from 'prop-types';
​
class EventOverview extends React.Component {
  static propTypes = {
    date: PropTypes.instanceOf(Date).isRequired,
    description: PropTypes.string,
    ticketsUrl: PropTypes.string,
    title: PropTypes.string.isRequired
  };
​
  render() {
    const { date, description, ticketUrl, title } = this.props;
    return (
      <div>
        <h1>{title}</h1>
        <h2>{date.toLocaleString()}</h2>
        {description && (
          <div className="description">{description}</div>
        )}
        {ticketsUrl && <a href={ticketsUrl}>Tickets!</a>}
      </div>
    );
  }
}
​
ReactDOM.render(
  <EventOverview date={new Date()} title="React lernen und verstehen" />,
  document.getElementById('root')
);
```

In diesem Beispiel möchten wir eine Übersicht zu einem Event ausgeben. Wir definieren, dass die `EventOverview`-Komponente die beiden Props `date` und `title` haben **muss**, darüber hinaus die beiden Props `description` und `ticketsUrl` haben **kann**. Ob eine Prop **vorausgesetzt** wird, kann mittels des angehängten `.isRequired` gekennzeichnet werden. Die `date`-Prop muss dabei in unserem Beispiel eine Instanz des nativen JavaScript `Date`-Objekts sein, `title` muss ein `string` sein. Die beiden optionalen Props `description` und `ticketsUrl` müssen nicht übergeben werden, werden sie allerdings übergeben, müssen auch sie vom Typ `string` sein.

Trifft eine dieser Bedingungen nicht zu, weist uns React darauf ziemlich deutlich mit einer Warnung in der Konsole hin:

{% hint style="danger" %}
Warning: Failed prop type: Invalid prop \`title\` of type \`number\` supplied to \`EventOverview\`, expected \`string\`.
{% endhint %}

Bei **Stateless Functional Components** werden die **PropTypes** in gleicher Art und Weise definiert, allerdings haben wir hier natürlich keine Klasse, in der wir eine `static propTypes` Eigenschaft definieren können. Hier können wir einfach der Funktion selbst eine `propTypes`-Eigenschaft hinzufügen. Das sieht dann so aus:

```jsx
const EventOverview = ({ date, description, ticketUrl, title }) => (
  <div>
    <h1>{title}</h1>
    <h2>{date.toLocaleString()}</h2>
    {description && (
      <div className="description">{description}</div>
    )}
    {ticketsUrl && <a href={ticketsUrl}>Tickets!</a>}
  </div>
);

EventOverview.propTypes = {
  date: PropTypes.instanceOf(Date).isRequired,
  description: PropTypes.string,
  ticketsUrl: PropTypes.string,
  title: PropTypes.string.isRequired
}
```

Und damit wäre auch unsere **Stateless Functional Component** mit **PropType**-Checking ausgestattet!

In einigen Fällen ist es wünschenswert sinnvolle Standardwerte zu vergeben. Auch hierfür bietet uns React eine Möglichkeit, die sogenannten `defaultProps`. Diese werden ähnlich verwendet wie die `propTypes`, nämlich als statische Eigenschaft. Aber schauen wir uns ein schnelles Beispiel an:

```jsx
const Greeting = ({ name }) => (
    <h1>Hallo {name}!</h1>
);

Greeting.propTypes = {
    name: PropTypes.string.isRequired,
};

Greeting.defaultProps = {
    name: 'Gast',
};
```

Wir markieren die `name`-Prop der Komponente als `string.isRequired`, wir erwarten also, dass die Prop immer übergeben wird und dass sie auch immer ein String ist. Anschließend definieren wir einen Standardwert für die `name`-Prop. Dieser wird immer dann verwendet, wenn kein Wert für die entsprechende Prop übergeben wird.

```jsx
<Greeting name="Manuel" />
```

Verursacht also die Ausgabe: **Hallo Manuel!**

```jsx
<Greeting />
// oder:
const user = {};
<Greeting name={user.name} />
```

Nutzt wegen der fehlenden bzw. undefinierten `name`-Prop hingegen den `defaultValue`, in diesem Fall **Gast** und zeigt in beiden Fällen an: **Hallo Gast!** React ist dabei klug genug und erkennt bei fehlender aber als `isRequired` markierter Prop ob ein `defaultValue` existiert und zeigt eine Warnung nur dann an, wenn eine Prop fehlt und auch nicht gleichzeitig ein `defaultValue` definiert wurde.

{% hint style="info" %}
Beim **Deployment in Production** lohnt es sich das **Babel-Plugin-Transform-React-Remove-Prop-Types** zu verwenden. Dies spart noch einmal ein paar Bytes Bandbreite, da die `propType`-Definitionen aus dem Build entfernt werden, da diese ohnehin **nur im Development-Modus** berücksichtigt werden.

Das Plugin findet ihr unter:   
https://github.com/oliviertassinari/babel-plugin-transform-react-remove-prop-types

Installieren könnt ihr es auf der Kommandzeile mittels:  
`npm install --save-dev babel-plugin-transform-react-remove-prop-types` bzw.  
`yarn add --dev babel-plugin-transform-react-remove-prop-types`
{% endhint %}

## Flow

Anders als die **React PropTypes** ist **Flow** ein **statischer Typechecker** für **sämtliches** JavaScript, nicht allein nur für React-Komponenten. Wie React selbst wird auch **Flow** von Facebook entwickelt und fügt sich dadurch schön nahtlos in die meisten React Setups ein. Bis **Babel 6** war es sogar Teil des `babel-preset-react` Pakets wurde also sozusagen zur Verwendung mit React „mit-installiert“ und konnte ohne jeglichen zusätzlichen Aufwand einfach verwendet werden.

Seit **Babel 7** ist **Flow** in ein eigenes **Babel Preset** ausgelagert worden, dass sich aber ebenso einfach über `npm install @babel/preset-flow` \(bzw. analog dazu `yarn add @babel/preset-flow`\) installieren lässt. Anschließend muss dann lediglich noch das entsprechende `@babel/preset-flow` als Preset in die Babel-Config eingetragen werden. Das **Preset** wird benötigt um die **Flow-Syntax**, die kein valides JavaScript wäre, im **Build-Prozess** aus dem entsprechenden Files zu entfernen, so dass es beim Aufruf im Browser nicht zu Syntax-Fehlern kommt.

Neben dem Babel Preset wird außerdem noch die **Flow Executable** benötigt, die sich in ihrer jeweils aktuellsten Version mittels `npm install flow-bin` \(bzw. `yarn add flow-bin`\) installieren lässt. Die **Flow Executable** führt dann das eigentliche **Typechecking** durch.

Nachdem **Flow** installiert und das **Babel Preset** eingerichtet wurde, wird noch eine **Flow-Config** benötigt. Diese erstellt ihr ganz einfach über den Aufruf von `./node_modules/flow init` im Terminal in eurem Projektverzeichnis.

**Tipp:** um zu vermeiden jedes Mal `./node_modules` voranzustellen wenn Flow aufgerufen werden soll, könnt ihr euch einen Eintrag in den `script`-Teil eurer `package.json` machen:

```javascript
{
  "scripts": {
    [...]
    "flow": "flow"
  }
}
```

Dies sorgt dafür, dass ihr Flow anschließend über npm oder Yarn aufrufen könnt:

```bash
npm run flow init
```

oder mit Yarn:

```bash
yarn flow init
```

Nachdem ihr `flow init` aufgerufen habt, solltet ihr in eurem Projektverzeichnis eine neue Datei `.flowconfig` sehen, die erst einmal ziemlich leer aussieht, die von Flow aber benötigt wird. In diese Datei könnt ihr später Optionen setzen oder angeben welche Dateien mit Flow geprüft werden sollen oder welche eben nicht.

Ihr habt eure Babel-Config aktualisiert, das `flow-bin` Package in euer Projekt installiert und die `.flowconfig` angelegt? Super. Dann kann es richtig losgehen. Um zu verifizieren dass alles korrekt eingerichtet wurde, könnt ihr einmal flow aufrufen. Wenn ihr den flow-Eintrag von oben in eurer `package.json` hinzugefügt habt könnt ihr das mit dem Befehl `yarn flow` in eurem Terminal. Ist alles korrekt eingerichtet, seht ihr eine Meldung wie die folgende:

```bash
No errors!
Done in 0.57s.
```

Dies bedeutet Flow hat eure Files geprüft und keine Fehler gefunden. Wie auch, haben wir doch noch gar keine Files mit Typechecking erstellt.

Die Standard-Einstellungen von Flow sehen vor, dass nur Files gecheckt werden, die Flow mit einem entsprechenden Kommentar im Code signalisieren, dass diese Typechecks beinhalten. Dazu fügt ihr einfach oben in jedem beliebigen JavaScript-File folgende Zeile ein:

```javascript
// @flow
```

Schauen wir uns das obige Beispiel noch einmal an. Diesmal mit **Flow** als Typechecker anstelle von **PropTypes**:

```jsx
// @flow
import * as React from 'react';
import * as ReactDOM from 'react-dom';
​
type PropsT = {
  date: Date,
  description?: string,
  ticketsUrl?: string,
  title: string,
};

class EventOverview extends React.Component<PropsT> {
  render() {
    const { date, description, ticketUrl, title } = this.props;
    return (
      <div>
        <h1>{title}</h1>
        <h2>{date.toLocaleString()}</h2>
        {description && (
          <div className="description">{description}</div>
        )}
        {ticketsUrl && <a href={ticketsUrl}>Tickets!</a>}
      </div>
    );
  }
}
​
ReactDOM.render(
  <EventOverview date={new Date()} title="React lernen und verstehen" />,
  document.getElementById('root')
);
```

Anders als bei den **PropTypes** definieren wir hier zuerst eine **Type Definition** mit dem Namen `PropsT`. Der Name kann hier grundsätzlich frei gewählt werden. Oft werden `T` oder `Type` an den Namen der Type-Definitions angehängt, um es für Entwickler gleich ersichtlich zu machen, dass es sich dabei um eben solche handelt. Aber rein aus technischer Sicht ist das nicht notwendig. Den eben definierten Type übergeben wir dann in Form eines sogenannten „Generic Type“ an die Komponente: 

```jsx
class EventOverview extends React.Component<PropsT>
```

Type Definitions können auch inline definiert werden. Allerdings wirkt sich das ab einer gewissen Anzahl auch auf die Lesbarkeit aus. In unserem Beispiel sähe die dann so aus:

```jsx
class EventOverview extends React.Component<{
  date: Date,
  description?: string,
  ticketsUrl?: string,
  title: string,
}> {
  […]
}
```

Doch schauen wir uns die **Type-Definition** einmal genauer an. Wie schon bei den **PropTypes** legen wir hier fest welche **Props** eine Komponente übergeben bekommen kann und von welchem Typen diese sein müssen. Da wäre eine `date` Prop, die aus einer `Date`-Instanz bestehen muss und erforderlich ist. Als nächstes dann `description` und `ticketsUrl`, die durch ein `?` nach ihrem Namen als **optional** gekennzeichnet wurden und jeweils, sollten sie übergeben werden, vom Typ `string` sein müssen. Zuletzt wird eine `title` Prop erwartet, die ebenfalls ein `string` sein muss, aber nicht optional ist. Anders als bei **PropTypes** müssen hier nicht die erforderlichen Props mittels `isRequired` gekennzeichnet werden, sondern im Gegenteil die optionalen Props mittels Fragezeichen als optional markiert werden.

**Stateless Functional Components** können in gleicher Form gleich der Übergabe der Props als Funktionsargument typisiert werden:

```jsx
const EventOverview = (props: PropsT) => ([…]);
```

bzw. in destrukturierter Form:

```jsx
const EventOverview = ({ date, description, ticketUrl, title }: PropsT) => (/*…*/);
```

oder als Inline-Definition:

```jsx
const EventOverview = ({
    date,
    description,
    ticketUrl,
    title
}: {
    date: Date,
    description?: string,
    ticketsUrl?: string,
    title: string
}) => {
  […]
};
```

Doch das ist noch nicht alles. Flow kann eben, anders als **PropTypes**, sämtliches JavaScript checken, nicht bloß Props von React-Komponenten. Dies bedeutet, dass auch der State einer Komponente typisiert werden kann. Dazu ist ein zweiter Parameter in den sog. **Generics** vorgesehen:

```jsx
// @flow
import * as React from 'react';
import * as ReactDOM from 'react-dom';
​
type PropsT = {
  date: Date,
  description?: string,
  ticketsUrl?: string,
  title: string,
};

type StateT = {
  isBookmarked: boolean,
};

class EventOverview extends React.Component<PropsT, StateT> {
  state = {
    isBookmarked: false,
  };
  […]
}
```

Anders als in bisherigen Beispielen im Buch haben die Imports hier eine etwas andere Form. Statt:

```jsx
import React from 'react';
```

… wurde React in diesem Kapitel folgendermaßen importiert:

```jsx
import * as React from 'react';
```

Dies führt dazu, dass gleichzeitig auch die von React mitgelieferten **Type Definitions** mit importiert wurden. Dies ist notwendig, wenn wir bspw. ein React-Element aus einer Funktion zurückgeben und dieses typisieren wollen.

## TypeScript

**TypeScript** wird von Microsoft entwickelt und ist ein sogenanntes typisiertes **Superset** von JavaScript, was bedeutet, dass es nicht direkt im Browser ausgeführt werden kann sondern zuvor in einem Zwischenschritt von einem Compiler in „echtes“ JavaScript kompiliert wird. **TypeScript** sieht auf den ersten Blick erst einmal ähnlich aus wie **Flow** und funktioniert auch ähnlich. Während Flow allerdings lediglich ein reiner **Typechecker** ist bringt **TypeScript** als Superset noch etwas mehr mit. So war es lange vor **ES2015** schon möglich Klassen und Imports in **TypeScript** zu verwenden.

In der JavaScript-Community erfreut sich **TypeScript** immer wachsender Beliebtheit und auch in Verbindung mit React ist es immer häufiger zu finden. Aus diesem Grund möchte ich das hier nicht ganz unerwähnt lassen, wobei ich hier gleichzeitig nicht all zu sehr in die Tiefe gehen möchte, da **TypeScript** allein genug Material für ein eigenes Buch hergeben würde.

Im Bezug auf React wichtig zu wissen ist, dass TypeScript-Files üblicherweise eine `.ts` Datei-Endung haben, enthält eine Datei auch JSX, muss die Datei zwingend mit `.tsx` enden.

Mit dem Release von Babel 7 wurde auch die Integration vereinfacht und es benötigt nun nicht mehr zwangsweise den **TypeScript** Compiler \(`tsc`\) sondern kann in Form eines Babel Plugins verwendet werden. Das Plugin wird mit dem Babel Preset `@babel/preset-typescript` installiert.

