# State Management

Mit zunehmender Größe einer Anwendung wächst meist auch deren Komplexität und die Daten, die mit der Anwendung verwaltet werden sollen. Also anders gesagt: der **Application State** wird schwieriger und unübersichtlicher zu verwalten. Wann gebe ich wie, welcher Komponente welche Props hinein. Wie wirken sich diese Props auf den State meiner Komponente aus und was passiert wenn ich den State in einer Komponente modifiziere.

Auch wenn React hier in den letzten Jahren zunehmend deutlich besser geworden ist, gerade mit der neuen **Context API** und dem **useReducer-Hook** einiges zur besseren Übersichtlichkeit von komplexen Datenstrukturen getan hat ist es noch immer nicht ganz einfach immer alle Daten und Datentransformationen im Blick zu haben. Um dieses Problem zu lösen gibt es einige externe Tools für **globales State Management**, die sich im Ecosystem um React herum gebildet haben. 

Zu den bekannteren Tools dieser Art wäre hier einerseits **MobX**, das sich selbst als _„Simple, scalable state management“_ beschreibt, also als Werkzeug für _simples und skalierbares State Management_. Auf der anderen Seite haben wir **Redux**, zweifellos der „Platzhirsch“ in der React-Welt, mitentwickelt unter anderem von Dan Abramov und Andrew Clark, die inzwischen auch dem offiziellen React-Team angehören. Redux bezeichnet sich als _„A predictable state container for JavaScript apps“_  also als _„vorhersehbarer State Container für JavaScript-Anwendungen“_.

In diesem Kapitel soll es insbesondere um **Redux** gehen. Einerseits, weil ich selbst mit **Redux** in vielen Projekten gearbeitet und sehr gute Erfahrungen damit machen durfte, andererseits aber auch weil es mit \(laut npmjs.com\) wöchentlich rund **4 Millionen Installationen** ganz klar deutlich mehr im Mainstream angekommen ist als **MobX**, mit immer respektablen, verglichen jedoch mit **Redux** doch überschaubaren **200.000 Installationen**.

**Redux** erfreut sich dabei noch immer an steigender Beliebtheit und hat wachsende Downloadzahlen zu verzeichnen obwohl es bereits mehrmals von irgendwelchen Propheten tot gesagt wurde. Als die finale **Context API** in **React 16.3.0** veröffentlicht wurde hieß es, dass **Redux** damit obsolet würde \(wurde es nicht\), als mit **React 16.8.0** die **Hooks** und hier insbesondere der stark an **Redux** angelehnte **useReducer-Hook** eingeführt wurde gab es diese Stimmen erneut. 

In der Realität sieht das so aus, dass die Zahl der Installationen auch nach Einführung von **Context** und **Hooks** weiter steigen und **Redux** selbst intern **Gebrauch** dieser neuen Möglichkeiten macht, um einerseits die Performance zu verbessern und andererseits die Verwendung der eigenen API zu vereinfachen. Darüber hinaus hat **Redux** inzwischen ein unheimlich großes Ecosystem an eigenen Addons und Tools um sich versammelt, das auch durch die in React neu hinzugekommenen Funktionen nicht ersetzt wird.

### Einführung in Redux

Bei **Redux** handelt es sich also wie beschrieben um einen _vorhersehbaren State Container_. Doch was bedeutet das genau? An dieser Stelle möchte ich gern etwas weiter ausholen, da ich es für wichtig erachte das Grundprinzip zu verstehen um hinterher Fehler zu vermeiden die schnell gemacht werden, hat man das Prinzip hinter **Redux** nicht zumindest in sehr groben Ansätzen verinnerlicht.

Erst einmal haben wir mit **Redux** ein Tool, das in Teilen angelehnt ist an die Prinzipien der **Flux Architektur**. Diese wurde - wie auch **React** selbst - von **Facebook** entwickelt, um die Entwicklung clientseitiger Web Anwendungen zu vereinfachen. Das Grundprinzip sieht dabei einen **unidirektionalen Datenfluss** vor, bei dem Daten immer nur **in eine Richtung** fließen. Also das Prinzip, wie wir es bereits aus **React** selbst kennen: eine **Aktion** \(bspw. ausgelöst durch einen Button-Klick\) ändert den **State**, die **State-Änderung** löst ein **Rerendering** aus und erlaubt es dann weitere Aktionen auszuführen. 

Nach diesem Prinzip funktioniert auch **Redux**, mit dem entscheidenden Unterschied jedoch, dass der State eben **global** verwaltet wird statt nur innerhalb einer Komponente und somit alle Komponenten, egal wo im Seitenbaum sich diese befinden, auf **sämtliche Daten** zugreifen können. 

Um Redux in einem Projekt zu nutzen, müssen wir es natürlich zuerst wieder einmal über die Kommandozeile installieren:

```bash
npm install redux react-redux
```

bzw. mit Yarn:

```bash
yarn add redux react-redux
```

Installiert werden hier **zwei** Pakete. Die Library `redux` selbst und `react-redux`. Während mit dem `redux` Paket die eigentliche State Management Library installiert wird, werden mit `react-redux` die sog. **Bindings** installiert. Auf gut Deutsch gesagt ist dies einfach nur ein Paket mit einigen **React-Komponenten** die konkret für die Verwendung von **Redux** mit **React** entwickelt und daraufhin optimiert wurden. Keine große Magie. 

Theoretisch wäre auch die Verwendung von Redux alleine möglich, allerdings müssten wir uns dann selbst darum kümmern zu schauen wann Komponenten neu gerendert werden und darum, wie Daten aus einer Komponente in den State Container rein und wieder raus kommen. Da wir das nicht wollen, weil sich jemand anderes der sich viel besser damit auskennt als wir alle das bereits gemacht hat, nutzen wir eben zusätzlich `react-redux`.

### Store, Actions und Reducer

Nicht von den möglicherweise noch unbekannten Begriffen verunsichern lassen. Wir gehen sie der Reihe nach durch und irgendwann macht es Klick.

Alle Daten in **Redux** befinden sich in einem sogenannten **Store**, der sich um die Verwaltung des globalen **States** kümmert. Theoretisch kann eine Anwendung auch mehrere Stores haben, im Konzept von Flux ist das sogar explizit so vorgesehen, in **Redux** ist das jedoch um Komplexität zu reduzieren eher unüblich und so beschränken sich React Anwendungen die **Redux** einsetzen meist auch auf lediglich einen _einzigen_ **Store** als **Single Source of Truth**, also als _die_ einzige wahre Quelle für alle Daten. Der Store stellt Methoden bereit um die sich in ihm befindlichen Daten zu verändern \(`dispatch`\), zu lesen \(`getState`\), und auf Änderungen zu reagieren \(`subscribe`\).

Die einzige Möglichkeit um Daten in einem **Store** zu verändern ist dabei durch das Auslösen \(_„dispatchen“_\) von **Actions**. Auch hier lässt sich **Redux** wieder von Ideen aus der Flux Architektur inspirieren und macht es erforderlich diese **Actions** im Format der **Flux Standard Actions** \(FSA\) zu halten. Eine solche **FSA** bestehen aus einem simplen JavaScript-Objekt das immer **zwingend** eine `type`-Eigenschaft besitzen _muss_ und darüber hinaus die drei weiteren Eigenschaften `payload`, `meta` und `error` besitzen _kann_, wobei uns in erster Linie einmal die `payload` interessieren soll, mit der wir es in 9 von 10 Fällen in denen wir eine **Action** _dispatchen_ zu tun haben werden. 

Die **Payload** stellt sozusagen den _Inhalt_ einer **Action** dar und kann vom simplen Boolean oder String, über numerische Werte, bis hin zu komplexen Arrays oder Objekten beliebige Daten beinhalten die serialisierbar sind, also in Form einer JSON-Repräsentation gespeichert werden können.

Beispiel für eine typische **Action** in **Redux**:

```javascript
{
  type: "USER_ADDED",
  payload: {
    id: 1,
    name: "Manuel"
  }
}
```

Wird eine **Action** durch die vom **Store** bereitgestellte `dispatch`Methode ausgelöst, wird der zum Zeitpunkt des Aufrufs aktuelle **State** zusammen mit der ausgelösten **Action** an die **Reducer** übergeben. Ein **Reducer** ist eine **pure Function**, die wir ebenfalls bereits aus React kennen, und dient dazu um aus dem **aktuellen State** und der jeweiligen Action mit ihren `type` und `payload` Eigenschaften einen **neuen State** zu erzeugen. Zur Erinnerung: eine **pure Function** erzeugt stets die **selbe Ausgabe** bei **gleichen Eingabeparametern**, egal wie oft diese aufgerufen wird. Dieses Verhalten ist es, das sie vorhersehbar und dadurch auch gleichzeitig einfach testbar macht.

Beispiel für einen **Reducer**:

```javascript
const reducer = (state, action) => {
  switch (action.type) {
    case "PLUS": {
      return state + action.payload;
    }
    case "MINUS": {
      return state - action.payload;
    }
    default: {
      return 0;
    }
  }
};
```

Ein **Store** erwartet bei seiner Erstellung grundsätzlich einen einzigen **Reducer**, jedoch bietet **Redux** hier mit der `combineReducers()`-Funktion eine Möglichkeit um diese **Reducer**-Funktion zur besseren Verständlichkeit und Lesbarkeit in beliebig viele kleine Teilstücke zu splitten und anschließend zu einem großen, gemeinsamen **Reducer** zusammen zu setzen, auch **Root-Reducer** genannt. Wird dann eine Action dispatched wird jeder **Reducer** mit jeweils den gleichen Parametern, nämlich dem **State** und der **Action**, aufgerufen.

### Einen Store erstellen



