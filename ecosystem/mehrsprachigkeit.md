# Mehrsprachigkeit

Eins der Themen, das neben **Routing** und **State Management** immer wieder aufkommt ist **Mehrsprachigkeit**. Wie bekomme ich meine Anwendung übersetzt, so dass ein deutscher Benutzer ein deutsches Interface sieht, während andere Benutzer bspw. ein englisches Interface sehen.

Ich möchte in diesem Kapitel nicht generell in die Tiefen der **Internationalisierung** \(oftmals abgekürzt: **i18n**\) einsteigen, denn hier soll es um **React** gehen. Ich setze daher ein gewisses \(geringes\) Grundverständnis voraus, was die Internationalisierung von User Interfaces angeht und konzentriere mich daher insbesondere darauf, wie man selbige in React umsetzt.

Wie Mehrsprachigkeit in sehr simpler Form für sehr einfache Apps mittels **Context API** umgesetzt werden kann, wurde bereits im Kapitel über die **Context API** demonstriert, wo wir eine sehr rudimentäre Form der Mehrsprachigkeit beispielhaft mit eben dieser API umgesetzt haben. Mit zunehmender Komplexität einer Anwendung steigt aber auch die Komplexität an die Mehrsprachigkeit. Plötzlich werden Dinge wichtig wie die Verwendung von Platzhaltern oder die Verwendung des Plurals. Hier ist es dann irgendwann sinnvoll auf Pakete zurückzugreifen, die genau zu diesem Zweck entwickelt worden sind.

Hier gibt es einige bewährte Optionen im **React Ecosystem** auf die wir zurückgreifen können. Zu den bekannteren gehören hier **Lingui**, **Polyglot**, **i18next** oder **react-intl** und auch Facebook hat mit **FBT** ein eigenes Framework für die Internationalisierung von React Anwendungen im Angebot. In diesem Kapitel geht es vorrangig um `i18next` und dessen React Bindings `react-i18next`. 

Warum? Nun, ich habe in verschiedenen Projekten mit `react-intl` und `react-i18next` gearbeitet, habe die anderen Alternativen ausführlich evaluiert und bin der festen Überzeugung, dass **i18next** in vielerlei Hinsicht die beste der genannten Alternativen darstellt. Es existiert eine sehr große und aktive Community rund um **i18next**, es werden neben **React** auch noch viele andere Frameworks, Libraries und Plattformen unterstützt, es läuft ohne großen Aufwand sowohl clientseitig im Browser als auch serverseitig in Node.js und das Übersetzungsformat von **i18next** wird von so ziemlich allen großen online Übersetzungs-Services als Exportformat angeboten. 

Darüber hinaus bot es als erstes Paket bereits wenige Tage nach deren Erscheinen Unterstützung für Hooks und wurde sogar gezielt auf dessen Verwendung optimiert ohne dabei die Abwärtskompatibilität einzubüßen. Dabei bleibt die API überwiegend simpel und einfach zu benutzen und bietet maximale Flexibilität. Um es mit anderen Worten zu sagen: das ganze Paket ist sehr komplett und lässt wenig bis gar keine Wünsche offen.

Um es zu installieren rufen wir wieder unser Terminal auf und geben ein:

```bash
npm install i18next react-i18next
```

bzw. mit Yarn:

```bash
yarn add i18next react-i18next
```

Wir installieren hier mit `i18next` das **Internationalisierungs-Framework** selbst, sowie mit `react-i18next` dessen **React Bindings**, also sozusagen eine Reihe an Komponenten und Funktionen die uns die Arbeit mit i18next in React deutlich erleichtern. So, wie wir das Prinzip schon im Kapitel zu State Management mit `redux` und `react-redux` kennengelernt haben.

Starten wir zunächst einmal indem wir uns zwei Objekte anlegen, die unsere Übersetzungen beinhalten. Eins mit Texten in Deutsch, eins mit Texten in Englisch:

```javascript
const de = {
  greeting: 'Hallo Welt!',
  headline: 'Heute lernen wir Internationalisierung mit React',
};

const en = {
  greeting: 'Hello world!',
  headline: 'Today we learn Internationalization with React',
};
```

Um **i18next** nun in unserer Anwendung zu verwenden, müssen wir es importieren, initialisieren und das React Plugin übergeben. Dies sollte idealerweise am Anfang unserer Anwendung passieren. Möglichst bevor wir unsere App-Komponente an den `ReactDOM.render()`-Aufruf übergeben. 

Dazu importieren wir das `i18next`-Paket selbst, sowie den benannten Export `initReactI18next` aus dem `react-i18next`-Paket:

```javascript
import i18next from 'i18next';
import { initReactI18next } from 'react-i18next';
```

Anschließend nutzen wir die `.use()`-Methode um das React Plugin an **i18next** zu übergeben, sowie die `.init()`-Methode um **i18next** zu initialisieren:

```javascript
i18next
  .use(initReactI18next)
  .init({ ... });
```

Die `init()`-Methode erwartet dabei ein Config-Objekt, das mindestens die beiden Eigenschaften `lng` \(für die ausgewählte Sprache\) sowie `resources` \(für die Übersetzungen selbst\) beinhalten sollte. Darüber hinaus ist es oft sinnvoll eine `fallbackLng` zu erstellen, also eine Fallback-Sprache, die dann benutzt wird wenn eine Übersetzung in der ausgewählten Sprache nicht vorhanden ist. Insgesamt bietet **i18next** hier über 30 verschiedene Konfigurationsoptionen an, doch die genannten drei sind die, die uns für den Moment interessieren:

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

Wir setzen also die Sprache erst einmal auf Englisch, ebenso die Fallback-Sprache. Dann folgt ein `resources`-Objekt das etwas Erläuterung bedarf. Das Objekt hat die Form:

```text
[Sprache].[Namespace].[Translation Key] = Übersetzung
```

Die Sprache dürfte klar sein. Das kann `de` für Deutsch sein, `en` für Englisch oder auch `de-AT` für deutsch mit österreichischer Regionalisierung. Die Spracheigenschaft besitzt als Wert ein Objekt, bestehend aus mindestens einem bis zu theoretisch unbegrenzt vielen **Namespaces**.

Der **Namespace** ist ein zentrales Feature in **i18next** das benutzt werden kann, aber nicht muss. Es erlaubt dem Entwickler größere Übersetzungsdateien in mehrere Teile zu splitten, die bei Bedarf dynamisch nachgeladen werden können. Während dieses Feature bei kleineren Anwendungen nicht unbedingt sinnvoll ist, kann es in größeren und komplexeren Anwendungen dazu benutzt werden um Übersetzungen klein und übersichtlich zu halten, etwa indem jeder größere Seitenbereich einen eigenen **Namespace** erhält.

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



