# Mehrsprachigkeit

Eins der Themen, das neben **Routing** und **State Management** immer wieder aufkommt, ist **Mehrsprachigkeit**. Wie bekomme ich meine Anwendung übersetzt, so dass ein deutscher Benutzer ein deutsches Interface sieht, während andere Benutzer bspw. ein englisches Interface sehen.

Ich möchte in diesem Kapitel nicht generell in die Tiefen der **Internationalisierung** \(meist abgekürzt: **i18n**\) einsteigen, denn hier soll es um **React** gehen. Ich setze daher ein gewisses \(geringes\) Grundverständnis voraus, was die Internationalisierung von User Interfaces angeht und konzentriere mich daher insbesondere darauf, wie man diese in React umsetzen kann.

Wie Mehrsprachigkeit in sehr simpler Form für einfache Apps mittels **Context API** umgesetzt werden kann, wurde bereits im Kapitel über die **Context API** demonstriert, wo wir eine sehr rudimentäre Form der Mehrsprachigkeit beispielhaft mit eben dieser API umgesetzt haben. Mit zunehmender Komplexität einer Anwendung steigt aber auch die Komplexität an die Mehrsprachigkeit. Plötzlich werden Dinge wichtig wie die Verwendung von Platzhaltern oder die Verwendung des Plurals. Hier ist es dann irgendwann sinnvoll, auf Pakete zurückzugreifen, die genau zu diesem Zweck entwickelt worden sind.

Hier gibt es einige bewährte Optionen im **React-Ecosystem,** auf die wir zurückgreifen können. Zu den bekannteren gehören hier **Lingui**, **Polyglot**, **i18next** oder **react-intl** und auch Facebook hat mit **FBT** ein eigenes Framework für die Internationalisierung von React-Anwendungen im Angebot. In diesem Kapitel geht es vorrangig um `i18next` und dessen React-Bindings `react-i18next`. 

Warum? Nun, ich habe in verschiedenen Projekten mit `react-intl` und `react-i18next` gearbeitet, habe die anderen Alternativen ausführlich evaluiert und bin der festen Überzeugung, dass **i18next** in vielerlei Hinsicht die beste der genannten Alternativen darstellt. Es existiert eine sehr große und aktive Community rund um **i18next**, es werden neben **React** auch noch viele andere Frameworks, Libraries und Plattformen unterstützt, es läuft ohne großen Aufwand sowohl clientseitig im Browser als auch serverseitig in Node.js und das Übersetzungsformat von **i18next** wird von so ziemlich allen großen online Übersetzungs-Services als Exportformat angeboten. 

Darüber hinaus bot es als erstes Paket bereits wenige Tage nach deren Erscheinen Unterstützung für Hooks und wurde sogar gezielt auf dessen Verwendung optimiert, ohne dabei an Abwärtskompatibilität einzubüßen. Dabei bleibt die API überwiegend simpel und einfach zu benutzen und bietet maximale Flexibilität. Um es mit anderen Worten zu sagen: Das ganze Paket ist sehr komplett und lässt wenig bis gar keine Wünsche offen.

### Setup von i18next

Um es zu installieren, rufen wir wieder unser Terminal auf und geben ein:

```bash
npm install i18next react-i18next
```

bzw. mit Yarn:

```bash
yarn add i18next react-i18next
```

Wir installieren hier mit `i18next` das **Internationalisierungs-Framework** selbst, sowie mit `react-i18next` dessen **React Bindings**, also sozusagen eine Reihe an Komponenten und Funktionen, die uns die Arbeit mit i18next in React deutlich erleichtern. Nach dem Prinzip, das wir schon im Kapitel zu State Management mit `redux` und `react-redux` kennengelernt haben.

Starten wir, indem wir uns zunächst zwei Objekte anlegen, die unsere Übersetzungen beinhalten. Eins mit Texten in Deutsch, eins mit Texten in Englisch:

```javascript
const de = {
  greeting: 'Hallo Welt!',
  headline: 'Heute lernen wir Internationalisierung mit React',
  messageCount: '{{count}} neue Nachricht',
  messageCount_plural: '{{count}} neue Nachrichten',
};

const en = {
  greeting: 'Hello world!',
  headline: 'Today we learn Internationalization with React',
  messageCount: '{{count}} new message',
  messageCount_plural: '{{count}} new messages',
};
```

Um **i18next** nun in unserer Anwendung zu verwenden, müssen wir es importieren, initialisieren und das React Plugin übergeben. Dies sollte idealerweise am Anfang unserer Anwendung passieren. Möglichst, bevor wir unsere App-Komponente an den `ReactDOM.render()`-Aufruf übergeben. 

Dazu importieren wir das `i18next`-Paket selbst, sowie den benannten Export `initReactI18next` aus dem `react-i18next`-Paket:

```javascript
import i18next from 'i18next';
import { initReactI18next } from 'react-i18next';
```

Anschließend nutzen wir die `.use()`-Methode, um das React Plugin an **i18next** zu übergeben, sowie die `.init()`-Methode, um **i18next** zu initialisieren:

```javascript
i18next
  .use(initReactI18next)
  .init({ ... });
```

Die `init()`-Methode erwartet dabei ein Config-Objekt, das mindestens die beiden Eigenschaften `lng` \(für die ausgewählte Sprache\) sowie `resources` \(für die Übersetzungen selbst\) beinhalten sollte. Darüber hinaus ist es oft sinnvoll, eine `fallbackLng` zu erstellen, also eine Fallback-Sprache, die dann benutzt wird, wenn eine Übersetzung in der ausgewählten Sprache nicht vorhanden ist. Insgesamt bietet **i18next** hier über 30 verschiedene Konfigurationsoptionen an, doch die genannten drei sind die, die uns für den Moment interessieren:

```javascript
i18next
  .use(initReactI18next)
  .init({
    lng: 'en',
    fallbackLng: 'en',
    resources: {
      en: {
        translation: en,
      },
      de: {
        translation: de,
      }
    },
  });
```

Wir setzen also die Sprache erst einmal auf Englisch, ebenso die Fallback-Sprache. Dann folgt ein `resources`-Objekt, das etwas Erläuterung bedarf. Das Objekt hat die Form:

```text
[Sprache].[Namespace].[Translation Key] = Übersetzung
```

Die Sprache dürfte klar sein. Das kann `de` für Deutsch sein, `en` für Englisch oder auch `de-AT` für deutsch mit österreichischer Regionalisierung. Die Eigenschaft besitzt als Wert ein Objekt, bestehend aus mindestens einem bis zu theoretisch unbegrenzt vielen **Namespaces**.

Der **Namespace** ist ein zentrales Feature in **i18next,** das benutzt werden kann, aber nicht muss. Es erlaubt dem Entwickler, größere Übersetzungsdateien in mehrere Teile zu splitten, die bei Bedarf dynamisch nachgeladen werden können. Während dieses Feature bei kleineren Anwendungen nicht unbedingt sinnvoll ist, kann es in größeren und komplexeren Anwendungen dazu benutzt werden, Übersetzungen klein und übersichtlich zu halten, etwa indem jeder größere Seitenbereich einen eigenen **Namespace** erhält. Übersetzungsdateien für diesen Seitenbereich könnten dann separat gepflegt und nur dann geladen werden, wenn diese auch benötigt werden.

In **i18next** muss zwingend immer mindestens _ein_ **Namespace** benutzt werden. Standardmäßig heißt dieser `translation`, dies kann aber durch die `defaultNS`-Option im Konfigurationsobjekt der `.init()`-Methode geändert werden. Der **Namespace** ist dabei selbst wiederum ein Objekt, das letztendlich die eigentlichen Übersetzungen enthält in der Form `translationKey: value`, also etwa `greeting: 'Hallo Welt!'`. 

Der Wert selbst _kann_ wiederum ein Objekt sein: 

```javascript
{
  greeting: {
    morning: 'Guten Morgen!',
    evening: 'Guten Abend!',
  }
}
```

Oder in verkürzter Form:

```javascript
{
  'greeting.morning': 'Guten Morgen!',
  'greeting.evening': 'Guten Abend!',
}
```

Auch das liegt aber im Ermessen des Entwicklers.

### Verwendung von Übersetzungen in React-Komponenten

Ist **i18next** erst einmal korrekt eingerichtet und die Übersetzungen angelegt, können wir auch schon damit weitermachen, unsere Komponenten zu übersetzen. Auch hier bietet uns **i18next** volle Flexbilität: Wir bekommen eine `withTranslation`-HOC für die Verwendung mit **Klassen-Komponenten**, einen `useTranslation`-Hook zur Verwendung in **Function Components** und für Fälle, in denen wir Komponenten innerhalb von Übersetzungen nutzen, wollen gibt uns `react-i18next` auch eine entsprechende `Trans`-Komponente an die Hand. 

#### Verwendung mit Klassen-Komponenten und withTranslation\(\)-HOC

Wir gehen sie der Reihe nach durch und fangen mit der `withTranslation()`-Funktion an. Diese erzeugt uns eine **HOC** an die wir die zu übersetzende Komponente übergeben und bekommen daraufhin die Props `t` und `i18n` in diese hereingereicht. Dazu importieren wir die Funktion zuerst als benannten Export aus dem `react-i18next`-Paket:

```javascript
import { withTranslation } from 'react-i18next';
```

Anschließend rufen wir die Funktion auf, erhalten eine HOC und übergeben dieser die Komponente, in der wir auf die übersetzten Werte zugreifen möchten. Nutzen wir **Namespaces,** können wir außerdem den oder die gewünschten **Namespaces** an die `withTranslation()`-Funktion übergeben:

```javascript
// Ohne Namespaces (hier wird der Default-Wert benutzt):
const TranslatedComponent = withTranslation()(TranslatedComponent);

// Mit nur einem Namespace:
const TranslatedComponent = withTranslation('namespace')(Component);

// Mit mehreren Namespaces:
const TranslatedComponent = withTranslation([
  'namespace1', 
  'namespace2'
])(TranslatedComponent);
```

In der Praxis hat es sich für mich bewährt, einzelne Komponenten in eigene Dateien auszulagern und die `withTranslation()`-HOC beim Export um die Komponente zu legen:

```jsx
// Greeting.js
class Greeting extends React.Component {
  render() {
    const { t } = this.props;
    return (
      <h1>{t('greeting')}</h1>
    );
  }
}

export default withTranslation()(Greeting);
```

So können wir dann beim Import gleich die übersetzte Komponente importieren:

```javascript
import Greeting from './Greeting.js';
```

Hier importieren wir dann entsprechend die Komponente mit den durch i18next erweiterten Props `t`, `i18n` und `tReady`.

Die `t`-Funktion ist dabei die zentrale Funktion für alles was mit Übersetzungen zu tun hat. Sie bekommt einen **Translation Key** übergeben und gibt uns den übersetzten Wert zurück, basierend auf unserer aktuell eingestellten Sprache:

```jsx
<h1>{t('greeting')</h1> 
// -> <h1>Hello world!</h1>
```

Nutzen wir Platzhalter oder Plurale in unseren Übersetzungen, können diese als zweiter Parameter übergeben werden:

```javascript
<p>{t('messageCount', { count: 3 })}</p>
// -> <p>3 new messages</p>
```

Die `i18n` Prop enthält die zuvor initialisierte **i18next**-Instanz. Sie bietet uns einige Eigenschaften und Methoden, die für unsere Übersetzungen relevant sein können. Die beiden wichtigsten sind dabei sicherlich:

*  `i18n.language`, um die aktuell ausgewählte Sprache auszulesen
* `i18n.changeLanguage()`, um die ausgewählte Sprache zu ändern

Um bspw. in einer Komponente die Sprache von `en` in `de` zu ändern, kann hier der Aufruf von `i18n.changeLanguage('de')` benutzt werden.

#### Verwendung mit Function Components und useTranslation\(\)-Hook

Natürlich können nicht nur **Klassen-Komponenten** mit der `withTranslation()`-HOC benutzt werden sondern auch **Function Components**. Allerdings können wir in letzteren auch den `useTranslation()`-Hook verwenden, der die Komponente meist noch etwas übersichtlicher werden lässt. Dazu importieren wir den Hook wie auch die HOC ebenfalls aus dem `react-i18next`-Paket:

```javascript
import { useTranslation } from 'react-i18next';
```

Der **Hook** lässt uns dann per **Destructuring Assignment Syntax** aus ES2015+ die beiden Eigenschaften `t` und `i18n` aus ihm extrahieren:

```jsx
const Greeting = () => {
  const { t, i18n } = useTranslation();
  return (
    <div>
      <h1>{t('greeting')}</h1>
      <button onClick={() => i18n.changeLanguage('de')}>de</button>
      <button onClick={() => i18n.changeLanguage('en')}>en</button>
    </div>
  );
};
```

Wie auch bereits in der `withTranslation()`-HOC ist `t` hier die Funktion, um Übersetzungen über ihren **Translation Key** auszugeben und `i18n` die **i18next**-Instanz. Der Hook stellt also exakt die gleiche Funktionalität bereit wie auch die HOC, ist in **Function Components** aber deutlich übersichtlicher, weil expliziter. Um bestimmte **Namespaces** zu verwenden, können dem Hook wie schon der `withTranslation()`-Funktion ein String oder ein Array mit Strings übergeben werden:

```javascript
const { t } = useTranslation('namespace');
const { t } = useTranslation(['namespace1', 'namespace2']);
```

Wird kein Namespace übergeben, greift auch hier die Default-Einstellung.

### Komplexe Übersetzungen mit der Trans-Komponente

In einigen Fällen kann es notwendig sein, React-Komponenten in Übersetzungen zu verwenden. Das ist bspw. der Fall, möchte man die `Link`-Komponente vom **React Router** nutzen, um innerhalb einer Übersetzung auf eine andere URL zu verlinken. Das ist nur durch die Verwendung der `t()`-Funktion nicht ohne weiteres möglich. Hierzu stellt uns `react-i18next` die `Trans`-Komponente bereit. Die Verwendung ist nicht immer ganz einfach zu verstehen, aber sie ist ein sehr mächtiges Werkzeug.

Nehmen wir also an, wir wollen innerhalb unserer Übersetzung die `Link`-Komponente verwenden. Der Code sieht vor der Übersetzung also in etwa so aus:

```jsx
<p>
  <label>
    <input type="checkbox" name="terms" />
    Ich akzeptiere die <Link to="/terms">AGB</Link>
  </label>
</p>
```

Die Verwendung der Link-Komponente in einer Übersetzung würde hier nicht funktionieren, da Übersetzungen grundsätzlich Strings sind und wir keine Möglichkeit hätten, festzulegen, dass es sich bei `<Link>` um die Link-Komponente aus dem React-Router Paket handelt.

Die Lösung ist hier nun die Verwendung der `Trans`-Komponente. Übersetzungen, die mit dieser Komponente verwendet werden, können nummerierte Platzhalter besitzen. An ihrer Stelle werden Komponenten eingesetzt, die an gleicher Position in der `Trans`-Komponente verwendet werden. 

Schauen wir uns das obige Beispiel also einmal unter Verwendung von `Trans` an:

```jsx
// Übersetzungen:
const de = {
  terms: 'Ich akzeptiere die <1>AGB</1>.',
};

const en = {
  terms: 'I accept the <1>Terms and Conditions</1>.',
};

// JSX:
<p>
  <label>
    <input type="checkbox" />
    <Trans i18nKey="terms">
      I accept the <Link to="/terms">Terms and Conditions</Link>.
    </Trans>
  </label>
</p>
```

Hier umschließen wir unseren Text mit einem `Trans`-Element. Der Text dient dabei lediglich als Platzhalter, der verwendet wird, wenn es zu dem in der `i18nKey`-Prop angegebenen Wert keine entsprechende Übersetzung gibt. Nun geht es ans Zählen. Wo welcher Text wie ersetzt wird und an welche Stelle eine Komponente tritt, ergibt sich aus dem Index-Wert der Kind-Elemente. Analog dazu, wie auch `React.createElement()` funktioniert.

Im obigen Beispiel ergibt sich also folgendes Zähl-Ergebnis:

```text
0: I accept the
1: <Link to="/terms">Terms and Conditions</Link>
2: .
```

An die Stelle `<1>Terms and Conditions</1>` tritt also hier das `<Link>`-Element, da es im Children-Array den Index-Wert 1 besitzt.

Nun kann das gerade in komplexeren Strukturen, in denen es vielleicht sogar mehrere Links gibt, ziemlich umständlich werden, das zu zählen. Hier bietet `react-i18next` zwei Möglichkeiten, um die Platzhalter automatisch zu generieren. Beide werden als Option beim Initialisieren von **i18next** gesetzt.

Die erste Option ist das Setzen von `saveMissing` auf `true` und die Verwendung einer `missingKeyHandler`-Funktion. Im folgenden Beispiel nutzen wir ein simples `console.log()`, um fehlende Übersetzungen in die Browser-Konsole zu loggen:

```javascript
i18next
  .use(initReactI18next)
  .init({
    saveMissing: true,
    missingKeyHandler: (language, namespace, key, fallbackValue) => {
      console.log('Fehlende Übersetzung:', fallbackValue);
    },
    // weitere Optionen
  });
```

Wird nun ein `i18nKey` benutzt, der nicht existiert, wird stattdessen der Fallback-Wert, also der Wert aus dem Trans-Element in die Browser Konsole geschrieben:

{% hint style="info" %}
Fehlende Übersetzung: I accept the &lt;1&gt;Terms and Conditions&lt;/1&gt;.
{% endhint %}

Für diese kann nun eine entsprechende Übersetzung angelegt werden. Der **React Router** `Link` wurde in der Ausgabe bereits durch den passenden Platzhalter-Index ersetzt, in diesem Fall also **1**.

Als zweite Option können wir statt des `missingKeyHandler` die `debug`-Option auf `true` setzen. 

```javascript
i18next
  .use(initReactI18next)
  .init({
    debug: true,
    // weitere Optionen
  });
```

Wir erhalten dann eine Reihe an Debugging-Informationen, wie eben auch einen Hinweis auf fehlende Übersetzungen. Die Ausgabe ist dann etwa:

{% hint style="info" %}
i18next::translator: missingKey de translation terms I accept the &lt;1&gt;Terms and Conditions&lt;/1&gt;.
{% endhint %}

Die Ausgabe ist hier etwas unfänglicher als bei der eigenen `missingKeyHandler`-Variante, dafür reicht es hier aber auch aus, lediglich eine Option auf `true` zu setzen.

Innerhalb von `<Trans>`-Elementen kann übrigens auch die `t`-Funktion wie gewohnt verwendet werden. So wäre etwa auch Folgendes möglich:

```jsx
<Trans i18nKey="terms">
  I accept the <Link to="/terms" title={t('termsTitle')}>Terms and Conditions</Link>.
</Trans>
```

### Fazit

Mit den in diesem Kapitel gezeigten Beispielen ist es relativ einfach, seine Anwendung gleich mehrsprachig zu entwickeln. In der Praxis hat sich i18next dabei für mich persönlich als sehr universelles und vollständiges Tool erwiesen. Die Integration in verscheidene Libraries und Frameworks funktioniert einwandfrei, es bietet alle Funktionen, die ein solches i18n-Framework benötigt und mit nur wenigen Funktionen und Komponenten \(`i18n.changeLanguage()`, `t`, `<Trans>`\) ist es darüber hinaus relativ leicht zu erlernen.

Einmal korrekt aufgesetzt und eingerichtet, hier empfehle ich einen Blick auf die [vollständigen Optionen von i18next](https://www.i18next.com/overview/configuration-options), realisiert man dabei Mehrsprachigkeit in seiner React-Anwendung, ohne dass es sich wie ein Mehraufwand anfühlt. Dafür sorgen auch Online-Services wie [Locize.com](https://locize.com/) oder [Lokalise.co](https://www.lokalise.co), um nur einige Beispiele zu nennen, mit denen die Erstellung von Übersetzungen verwaltet und zum Teil sogar automatisiert outgesourced werden kann.

