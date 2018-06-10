# Typechecking mit PropTypes, Flow und TypeScript

**Typechecking** ist eine einfache Möglichkeit um potentielle Fehler in einer Anwendung zu vermeiden. Das Prinzip dabei ist ganz einfach: Komponenten sollten „Pure“ sein, wir wir schon in der Einführung gelernt haben. Sie sollten also möglichst keine Seiten-Effekte besitzen und vor allem sollten sie bei den **gleichen Eingabeparametern** \(was im Fall von Komponenten die **Props** und deren daraus abgeleiteter **State** ist\) auch die **identische Ausgabe** erzeugen.

Das bedeutet, dass es möglichst vorhersehbar und sehr strikt sein sollte welche **Props** in eine Komponente hereingereicht werden können und welche von ihr verarbeitet werden. Um dies sicherzustellen können wir uns das sog. **Typechecking** zu nutze machen. JavaScript ist prinzipiell eine untypisierte Sprache. Eine Variable die mal ein **String** war, kann problemlos in eine **Number** oder gar ein **Object** umgewandelt werden, ohne dass der JavaScript-Interpreter ein Problem damit hat.

Auch wenn dies bei der Entwicklung mitunter sehr praktisch ist weil wir uns nicht festlegen müssen, öffnet das die Tür für einige ärgerliche Fehler und macht es daher nötig, regelmäßig manuell auf den korrekten Typen zu prüfen. Wollen wir bspw. auf eine tief verschachtelte Eigenschaft `user.settings.notifications.newMessages`zugreifen, sollten wir zuvor prüfen ob `user` überhaupt ein Objekt und nicht `null` ist, anschließend sollten wir prüfen ob das gleiche für `settings` zutrifft, usw. Andernfalls könnten wir es mit einem Type Error zu tun haben:

{% hint style="danger" %}
TypeError: Cannot read property 'settings' of undefined
{% endhint %}

**Typechecking** kann uns hier also helfen derartige potentielle Fehler schon vorher zu entdecken. Dazu gibt es neben **Flow** und **TypeScript**, die statische Typisierung ermöglichen, mit den sogenannten **PropTypes** auch eine recht simple React-eigene Lösung. Während **Flow** und **TypeScript** generell statische Typisierung in JavaScript ermöglichen, beschränken sich die React **PropTypes** allein auf React-Komponenten und finden außerhalb von Komponenten keine Anwendung. Wer also Gefallen findet an statischer Typisierung, sollte durchaus mal einen Blick auf **Flow** oder **TypeScript** wagen.

## PropTypes

**PropTypes** reichen zurück bis in ganz frühe Versionen von React, lange bevor React seine heutige Popularität erreicht hat und wurden in React 15.5. aus dem Core heraus und in ein eigenes `prop-types` Package ausgelagert. Während man seine **PropTypes** vorher mittels bspw. `React.PropTypes.string` definieren konnte, erfolgt der Zugriff nun über das zuvor importierte `PropTypes` Modul: `PropTypes.string`. 

Das bedeutet auch, dass es erst als `devDependency` installiert werden muss. Auf der Kommandozeile reicht dafür ein simples:

```text
yarn add --dev prop-types
```

oder:

```text
npm install --save-dev prop-types
```

**PropTypes** dienen dabei als eine Art Interface und legen fest, welche Form eine Prop annehmen darf und ob diese optional ist oder immer erforderlich ist. Gibt es Abweichungen, weist uns React **im Development-Modus** darauf hin. Bei einer korrekt veröffentlichten Anwendung, die die Production-Version von React nutzt und/oder mit der Umgebungsvariable `process.env.NODE_ENV=production` gebaut wurde bekommen wir diese Warnungen **nicht** mehr zu sehen!

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

https://github.com/oliviertassinari/babel-plugin-transform-react-remove-prop-types
{% endhint %}

## Flow

## TypeScript

