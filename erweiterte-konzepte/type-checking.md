# Type Checking

Typechecking ist eine einfache Möglichkeit um potentielle Fehler in einer Anwendung zu vermeiden. Das Prinzip dabei ist ganz einfach: Komponenten sollten „Pure“ sein, wir wir schon in der Einführung gelernt haben. Sie sollten also möglichst keine Seiten-Effekte besitzen und vor allem sollten sie bei den **gleichen Eingabeparametern** \(was im Fall von Komponenten die **Props** und deren daraus abgeleiteter **State** ist\) auch die **identische Ausgabe** erzeugen.

Das bedeutet, dass es möglichst vorhersehbar und sehr strikt sein sollte welche **Props** in eine Komponente hereingereicht werden können und welche von ihr verarbeitet werden. Um dies sicherzustellen können wir uns das sog. Typechecking zu nutze machen. JavaScript ist prinzipiell eine untypisierte Sprache. Eine Variable die mal ein String war, kann problemlos in eine Number oder gar ein Objekt umgewandelt werden, ohne dass der JavaScript-Interpreter ein Problem damit hat.

Auch wenn dies bei der Entwicklung mitunter sehr praktisch ist weil wir uns nicht festlegen müssen, öffnet das die Tür für einige Fehler und macht es nötig, regelmäßig manuell auf den korrekten Typen zu prüfen. Wollen wir bspw. auf eine tief verschachtelte Eigenschaft `user.settings.notifications.newMessages`zugreifen, sollten wir zuvor prüfen ob `user` überhaupt ein Objekt und nicht `null` ist, anschließend sollten wir prüfen ob das gleiche für `settings` zutrifft, usw. Andernfalls könnten wir es mit einem Type Error zu tun haben:

{% hint style="danger" %}
TypeError: Cannot read property 'settings' of undefined
{% endhint %}

Typechecking kann uns hier also helfen derartige potentielle Fehler schon vorher zu entdecken. Dazu gibt es neben **Flow** und **TypeScript**, die statische Typisierung ermöglichen, mit den sogenannten **PropTypes** auch eine recht simple React-eigene Lösung. Während Flow und TypeScript generelle Typisierung in JavaScript ermöglichen, beschränken sich die React PropTypes rein auf React-Komponenten und finden außerhalb keine Anwendung. Wer also gefallen findet an Typisierung, sollte durchaus mal einen Blick auf **Flow** oder **TypeScript** wagen.

## PropTypes

PropTypes reichen zurück bis in frühe Versionen von React, lange bevor React seine heutige Popularität erreicht hat und wurden in React 15.5. in ein eigenes `prop-types` Package ausgelagert. Während man seine PropTypes vorher einfach mittels bspw. `React.PropTypes.string` definieren konnte, erfolgt der Zugriff nun über das zuvor importierte `PropTypes` Modul: `PropTypes.string`.

PropTypes legen fest welche Form eine Prop annehmen darf und ob diese optional ist oder immer erforderlich ist. Gibt es Abweichungen, weist uns React **im Development-Modus** darauf hin. Bei einer korrekt veröffentlichten App, die die Production-Version von React nutzt und mit der Umgebungsvariable `process.env.NODE_ENV=production` gebaut wurde bekommen wir diese Warnungen nicht zu sehen.

Doch wie sieht die Verwendung von PropTypes nun aus? Hier müssen wir unterscheiden zwischen der **Class Component** und der **Stateless Functional Component**. 

Bei der **Class Component** sind die PropTypes  eine statische Eigenschaft der Klasse

Warning: Failed prop type: Invalid prop \`foo\` of type \`string\` supplied to \`App\`, expected \`number\`.

https://github.com/oliviertassinari/babel-plugin-transform-react-remove-prop-types

## Flow

## TypeScript

