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

Ein **Store** erwartet bei seiner Erstellung grundsätzlich einen einzigen **Reducer**, jedoch bietet **Redux** hier mit der `combineReducers()`-Funktion eine Möglichkeit um diese **Reducer**-Funktion zur besseren Verständlichkeit und Lesbarkeit in beliebig viele kleine Teilstücke zu splitten und anschließend zu einem großen, gemeinsamen **Reducer** zusammen zu setzen, auch **Root-Reducer** genannt. Wird dann eine Action _dispatched_ wird jeder **Reducer** mit jeweils den gleichen Parametern, nämlich dem **State** und der **Action**, aufgerufen. Diese Methode schauen wir uns in diesem Kapitel später noch etwas genauer an.

Da jeder **Reducer** auf die `type`-Eigenschaft einer **Action** reagiert ist es ein Stück weit zur weit verbreiteten Konvention geworden alle verwendeten Types in gleichnamige Variablen auszulagern, da ein Tippfehler im ersten Moment nicht immer gleich auffällt \(bspw. `USER_ADDDED`\), der JavaScript-Interpreter jedoch einen Fehler wirft beim Zugriff auf eine nicht definierte Variable. So findet man in Apps in denen **Redux** eingesetzt wird zu Beginn einer Datei oft einen Code-Block von folgendem Format:

```javascript
const PLUS = 'PLUS';
const MINUS = 'MINUS';
```

Dies dient eben dazu, um Kohärenz bei den verwendeten **Action Types** sicher zu stellen.

### Einen Store erstellen

Um einen Store zu erstellen der unseren globalen State verwalten wird, importieren wir die `createStore` Funktion aus dem `redux` Paket, rufen diese auf und übergeben ihr eine **Reducer**-Funktion. Die Funktion gibt uns daraufhin das Store-Objekt zurück mit Methoden um mit dem Store zu arbeiten: `dispatch`, `getState` sowie `subscribe`, wobei letztere eine eher untergeordnete Rolle bei der Arbeit mit React spielt, da die Komponenten aus dem `react-redux` Paket sich um das Rerendering von Komponenten kümmern, die von einer Änderung am **State** betroffen sind.

```javascript
import { createStore } from "redux";

const initialState = 0;

const counterReducer = (state = initialState, action) => {
  switch (action.type) {
    case "PLUS": {
      return state + (action.payload || 0);
    }
    case "MINUS": {
      return state - (action.payload || 0);
    }
    default: {
      return state;
    }
  }
};

const store = createStore(counterReducer);
```

Hier haben wir einen ersten und noch sehr simplen Store mit einem Counter-Reducer erstellt. Da wir der `createStore` Funktion hier nur den ersten Parameter übergeben, wird die `counterReducer`-Funktion beim Initialisieren mit `undefined` als vorherigen `state` aufgerufen, weshalb der `initialState` als Standardparameter eingesetzt wird. Dieser entspricht hier dem numerischen Wert `0`. 

Übergeben wir der `createStore` Funktion einen zweiten Parameter, so würde dieser als erster State beim Initialisieren an die Reducer-Funktion übergeben werden:

```javascript
const store = createStore(counterReducer, 3);
```

Hier wäre der Startwert für unseren `state`-Parameter der an die `counterReducer`-Funktion übergeben wird `3` statt `undefined`, der `initialState` Standardparameter würde nicht einspringen und unser Counter würde bei `3` beginnen zu zählen, statt bei `0`.

Als initiale **Action** wird eine Redux-interne Action _dispatched_, die die Form `{type: '@@redux/INIT5.3.p.j.l.8'}` besitzt. Daher greift in unserem `switch`-Block der `default`-Fall und gibt einfach nur den übergebenen `state` \(der in diesem Fall dem `initialState` entspricht\) zurück. 

Ein solcher `default`-Fall ist wichtig. Greift kein anderer `case` im `switch`, muss stets der letzte Stand des States aus der Funktion zurückgegeben werden um unerwünschte Seiteneffekte zu vermeiden. Die `Reducer`-Funktion wird bei **jedem** `dispatch`-Aufruf ausgeführt und ihr Rückgabewert bestimmt immer den **nächsten State!**

Rufen wir nun gleich nach der Initialisierung `store.getState()` auf, erhalten wir unseren `initialState` zurück: `0`:

```javascript
console.log(store.getState()); // 0
```

Wir können nun etwas rumspielen, verschiedene **Actions dispatchen** und schauen wie unser **State** auf die **Actions** jeweils reagiert:

```javascript
store.dispatch({ type: "PLUS", payload: 2 });
console.log(store.getState()); // 2

store.dispatch({ type: "PLUS", payload: 1 });
console.log(store.getState()); // 3

store.dispatch({ type: "MINUS", payload: 2 });
console.log(store.getState()); // 1
```

Hier _dispatchen_ wir zweimal eine **Action** mit dem `type` `PLUS` und einmal eine Action vom `type` `MINUS`. Wir übergeben jeweils eine `payload`, mit der wir angeben um wie viele Ziffern wir den letzten State hoch- oder runterzählen wollen. Unsere **State-Mutation** findet also wie folgt statt:

```javascript
0 (initialer State) + 2 (payload) = 2 (neuer State)
2 (alter State)     + 1 (payload) = 3 (neuer State)
3 (alter State)     - 2 (payload) = 1 (neuer State)
```

Dieser State ist dabei noch denkbar simpel und besteht lediglich aus einem einzigen Wert. Später werden wir uns noch anschauen wie wir komplexeren **State** aus verschiedenen Objekten und mit mehreren **Reducern** erstellen.

### Action Creators vs. Actions

Wer Artikel über **Redux** liest oder auch in die offizielle Doku schaut wird immer wieder mit dem Begriff **Action Creator** konfrontiert. Mir selbst fiel es anfangs etwas schwer die Unterschiede zu verstehen und ich weiß auch von anderen, dass es nicht nur mir so erging. Daher möchte ich an dieser Stelle kurz einen keinen Exkurs einwerfen um die beiden Begriffe **Action Creator** und **Action** voneinander abzugrenzen.

Die **Action** haben wir oben bereits kennengelernt und ist ein einfaches und möglichst _serialisierbares_ **Objekt**, mit dem wir beschreiben wie wir den State verändern wollen. Es enthält immer zwingend eine `type`-Eigenschaft und meist auch eine `payload`.

Ein **Action** _**Creator**_ hingegen ist eine **Funktion**, die schlussendlich eine **Action** **zurückgibt**, also sozusagen eine **Factory**, die eine **Action** _erstellt_ \(daher: _Creator_\). Meist werden **Action Creators** dazu verwendet um in ihnen **Logik zu kapseln**, die zur Erstellung einer **Action** notwendig ist. Manchmal werden sie aber auch einfach verwendet um die doch recht umständliche Natur der **Actions** hinter einer einprägsamen Funktion weg zu abstrahieren. **Action Creators** werden dann an Stelle der ursprünglichen **Action** als Parameter der `dispatch`-Methode aufgerufen.

Typische **Action Creators** aus unserem obigen Beispiel würden dann bspw. so aussehen:

```javascript
const add = (number) => {
  return { type: 'PLUS', payload: number };
};

const subtract = (number) => {
  return { type: 'MINUS', payload: number };
}
```

Oder wer mit den **Shorthand Notationen** aus **ES2015+** vertraut ist kann das Ganze auch noch weiter abkürzen:

```javascript
const add = (payload) => ({ type: 'PLUS', payload });
const subtract = (payload) => ({ type: 'MINUS', payload });
```

Anschließend werden die entsprechenden **Action Creators** als Parameter der `dispatch`-Methode aufgerufen, statt die **Actions** direkt zu übergeben:

```javascript
store.dispatch(add(2));
store.dispatch(add(1));
store.dispatch(subtract(2));
```

Dies geht bei entsprechender Benamung der **Action Creator** Funktionen deutlich zu Gunsten besserer Verständlichkeit des Codes. **Action Creator** sind dabei ein wichtiges Puzzlestück im Redux-Konzept und aus der täglichen Arbeit nicht wegzudenken da sie uns viel Schreibarbeit und Wiederholungen ersparen und somit auch potentielle Fehlerquellen wie bspw. Tippfehler beim `type` einer **Action** verringern \(etwa `PLSU` statt `PLUS`\).

### Komplexe Reducer

Die bisherigen Beispiele dienten dazu mit dem Konzept von **Actions** und **Reducern** vertraut zu werden und um zu verstehen, wie **Actions** verwendet werden, um mit dem **Reducer** den **Store** zu mutieren. Typischerweise besteht der **State** in einer React-Anwendung aber aus deutlich komplexeren Daten und Objekten. Werfen wir also einen Blick auf einen Store, wie er realistisch in einer kleinen Anwendung aussehen könnte.

Als Beispiel soll uns das State-Management für eine fiktive kleine **Todo-App** dienen, die sowohl eine Liste mit Todos verwaltet, als auch einen eingeloggten Benutzer beinhaltet. Unser State halt also die beiden Toplevel-Eigenschaften `todos` \(vom Typ Array\) und `user` \(vom Typ Objekt\). Dies bilden wir in unserem initialen State auch so ab:

```javascript
const initialState = Object.freeze({
  user: {},
  todos: [],
});
```

Zusätzlich, da wir sicherstellen wollen, dass bei jedem Aufruf ein neues State-Objekt erzeugt wird und nicht das alte mutiert wird, umschließen wir unseren initialen State mit einem `Object.freeze()`. Dies sorgt dafür, dass wir einen `TypeError` bekommen, sollte das **State-Objekt** direkt mutiert werden.

Schauen wir uns an wie eine **Reducer**-Funktion aussieht könnte, mit der wir den eingeloggten Benutzer setzen, neue Todos hinzufügen und entfernen sowie ihren Status ändern können:

```javascript
const rootReducer = (state = initialState, action) => {
  switch (action.type) {
    case "SET_USER": {
      return {
        user: {
          name: action.payload.name,
          accessToken: action.payload.accessToken,
        },
        todos: state.todos,
      };
    }
    case "ADD_TODO": {
      return {
        user: state.user,
        todos: state.todos.concat(action.payload),
      };
    }
    case "REMOVE_TODO": {
      return {
        user: state.user,
        todos: state.todos.filter((todo) => todo.id !== action.payload.id),
      };
    }
    case "CHANGE_TODO_STATUS": {
      return {
        user: state.user,
        todos: state.todos.map((todo) => {
          if (todo.id !== action.payload.id) {
            return todo;
          }
          return {
            ...todo,
            done: action.payload.done,
          };
        }),
      };
    }
    default: {
      return state;
    }
  }
};

const store = createStore(rootReducer);
```

Ich möchte hier gar nicht auf alles was hier passiert zu tief ins Detail eingehen, da es hier darum gehen soll wie ein großer, unübersichtlicher **Reducer** in mehrere kleinere und idealerweise übersichtlichere aufgeteilt werden kann. Doch einige Stellen sind für das generelle Verständnis nicht unwichtig. Gehen wir also der Reihe nach den `switch`-Block durch, bei dem uns jeder `case`-Block ein neues State-Objekt zurück gibt.

Angefangen bei `SET_USER`: das hier erzeugte **State-Objekt** ändert das `user`-Objekt und setzt dessen `name`-Eigenschaft auf `action.payload.name`, sowie die `accessToken`-Eigenschaft auf `action.payload.accessToken`. Wer mag, der kann hier stattdessen auch `user` auf `action.payload` setzen, dann würden alle in der entsprechenden Payload der Action übergebenen Eigenschaften im `user`-Objekt landen. Dabei sollte dann aber sichergestellt sein, dass die `action.payload` immer ein Objekt ist, da wir ansonsten die Ausgangsform des `user`-Objekts verändern würden was zu Problemen führen kann, wenn bspw. auch andere Teile des Reducers auf das Objekt zugreifen und das Objekt plötzlich keines mehr ist. In unserem Beispiel ignorieren wir aber alle anderen Eigenschaften indem wir explizit nur `name` und `accessToken` aus der Payload der ausgelösten Action holen. 

Neben dem modifizierten `user` geben wir auch eine `todos`-Eigenschaft zurück, die wir auf `state.todos` setzen, also beim bisherigen Wert belassen. **Dies ist wichtig**, da in unserem State-Objekt die `todos` ansonsten komplett aus dem State entfernt werden würden und wir nun zwar den Benutzer gesetzt, unsere Todos jedoch aus dem State verloren hätten!

Weiter geht es mit `ADD_TODO`: Hier ist es andersherum und wir geben zuerst einmal den `user`-Ast unseres State-Trees unverändert zurück. Anschließend fügen wir das neue Todo-Item mittels `.concat()`-Methode dem `todo`-Array hinzu. Hier ist es wichtig `concat()` und nicht `push()` zu benutzen, da `push()` eine sog. _mutative_ Methode ist, also den bestehenden State verändert statt einen neuen State zu erzeugen. Mittels `state.todos.concat` nehmen wir das aktuelle `todos`-Array als Basis und erzeugen daraus ein neues Array mit dem neuen Todo-Item und geben dieses zurück.

Sehr ähnliches passiert im nächsten Fall: `REMOVE_TODO`. Hier geben wir wieder zuerst den `user`-Ast zurück, ehe wir im `todos`-Array nach dem zu entfernenden Eintrag suchen um diesen herauszufiltern. Das gefilterte Array ist dann unser neuer `todos`-State. Wir nutzen hier die `Array.filter()`-Methode, da diese anders als bspw. `Array.splice()` nicht _mutativ_ ist und ein neues Array erzeugt.

Zuletzt haben wir den `CHANGE_TODO_STATUS` Fall. Mit diesem ändern wir den Status des Todo-Elements, also von _Zu erledigen_ \(`false`\) auf _Erledigt_ \(`true`\) - oder eben andersherum. Dazu geben wir, erneut, das unveränderte `user`-Objekt aus dem vorherigen State zurück und iterieren anschließend mittels `state.todos.map()`durch alle Todos. In der Map-Funktion schauen wir ob die ID des aktuellen Todo-Objekts der ID aus der `action.payload` entspricht. Ist dies nicht der Fall, geben wir einfach jeweilige Todo-Element unverändert zurück. 

Entspricht die ID aus der Payload der ID des aktuellen Todo-Elements, erzeugen wir ein neues Objekt, schreiben alle Eigenschaften mit ihren jeweiligen Werten mittels ES2015+ Spread-Syntax \(`{ ...todo }`\) in das neu erzeugte Objekt und überschreiben die `done`-Eigenschaft mit dem neuen Wert aus der Action-Payload. Wir generieren hier ein neues Objekt statt einfach das bestehende zu überschreiben, da wir ja einen neuen State erzeugen müssen damit unser **Reducer** eine **Pure Function** bleibt. Die Verwendung der `Array.map()`-Methode sorgt bereits dafür, dass wir außerdem ein neues Array erzeugen.

Hier haben wir es lediglich mit zwei Ästen in unserem State-Tree zu tun: `user` und `todos` - und dennoch wird unsere **Reducer**-Funktion bereits sehr lang. Bei steigender Komplexität des States wird die Funktion entsprechend noch länger und vor allem: **fehleranfälliger**. Da wir neben dem veränderten Teil des States auch immer alle anderen Teile zurückgeben müssen - also etwa den unveränderten `user` wenn wir die `todos` modifizieren oder eben die `todos` wenn wir den `user` modifizieren, wird die Funktion sehr schnell unübersichtlich und schwierig zu verwalten.

Aus diesem Grund stellt **Redux** die `combineReducer()`-Methode bereit. Mit ihr wird es möglich unsere **Reducer** in einzelne Teilbereiche aufzuteilen die sich um eine bestimmte Aufgabe kümmern und in eigene Dateien ausgelagert werden können. 

Ausgehend von unserem Beispiel hätten wir hier also die beiden **Reducer-**Funktionen `user` und `todos`. Beide befinden sich in einer eigenen Datei, die die **Reducer-**Funktion als `default` exportiert:

```javascript
// user/reducer.js
const initialState = Object.freeze({});

export default (state = initialState, action) => {
  switch (action.type) {
    case "SET_USER": {
      return {
        name: action.payload.name,
        accessToken: action.payload.accessToken,
      };
    }
    default: {
      return state;
    }
  }
};
```

```javascript
// todos/reducer.js
const initialState = Object.freeze([]);

export default (state = initialState, action) => {
  switch (action.type) {
    case "ADD_TODO": {
      return state.concat(action.payload);
    }
    case "REMOVE_TODO": {
      return state.filter((todo) => todo.id !== action.payload.id);
    }
    case "CHANGE_TODO_STATUS": {
      return state.map((todo) => {
        if (todo.id !== action.payload.id) {
          return todo;
        }
        return {
          ...todo,
          done: action.payload.done,
        };
      });
    }
    default: {
      return state;
    }
  }
};
```



### Asynchrone Actions

Alle **Actions** in vorherigen Beispielen wurden bisher immer **synchron** ausgeführt. In dynamischen Web-Anwendungen in denen die Stärken von **React** am meisten zum Vorschein kommen, haben wir jedoch regelmäßig mit **asynchronen Datenflüssen** zu tun, insbesondere mit Netzwerk-Requests. Hier helfen uns unsere _synchronen_ **Action Creator**-Funktionen nicht wirklich weiter, da die `dispatch`-Methode eines **Stores** ja zwingend eine **Action** erwartet, die, wie wir bereits gelernt haben ein simples und einfaches Objekt mit einer `type`-Eigenschaft ist.

Hier kommt jetzt das **Middleware**-Konzept von **Redux** ins Spiel, und mit ihr insbesondere die **Redux Thunk Middleware**. Doch der Reihe nach.

Die `createStore`-Funktion aus dem `redux`-Paket, die wir etwas weiter oben bereits benutzt haben um einen Store zu erzeugen, verarbeitet bis zu drei Parameter:

* Die **Reducer**-Funktion, die als einziger Parameter zwingend angegeben werden **muss** und sich in Verbindung mit den ausgelösten **Actions** um die Mutation unseres **States** kümmert indem sie mit jeder ausgelösten **Action** einen **State** zurückgibt.
* Einen **initialen State**, der bspw. beim Initialisieren des Stores bereits mit Daten befüllt sein kann. Dieser initiale State wird bei der Initialisierung des Stores auch auch die **Reducer**-Funktion übergeben.
* Eine sog. **Enhancer**-Funktion \(zu dt. etwa: _Erweiterungs-Funktion_\), mit der wir unseren erzeugten Store um eigene Funktionalität erweitern können. Wie etwa die eben angesprochene **Middleware**. 

Bekommt die `createStore`-Funktion zwei Parameter übergeben, behandelt sie den zweiten Parameter als **Enhancer** wenn der Parameter eine **Funktion** ist. Andernfalls wird der zweite Parameter als **initialer State** an die **Reducer**-Funktion übergeben.

Die **Middleware** in **Redux** legt sich dabei um die `dispatch`-Methode, fängt Aufrufe an diese ab, erlaubt es uns die aufgerufene **Action** zu modifizieren und gibt am Ende ihrer Ausführung eine neue `dispatch`-Funktion zurück. Möchten wir nun bspw. asynchrone Funktionen bzw. Promises als Parameter an die `dispatch()`-Methode übergeben, können wir den **Store-Enhancer** nutzen um **Middleware** zu registrieren, die es uns erlaubt genau dies zu tun. Die bekannteste dieser Art ist besagte **Thunk Middleware**.

Zur Installation:

```bash
npm install redux-thunk
```

oder in Yarn:

```bash
yarn add redux-thunk
```

Ist die **Thunk Middleware** installiert, müssen wir sie über die Redux eigene `applyMiddleware`-Funktion beim Enhancer _registrieren_. Dazu importieren wir die Middleware selbst und die applyMiddleware-Funktion direkt aus Redux:

```javascript
import { applyMiddleware, createStore } from 'redux';
import thunk from 'redux-thunk';

// ...

const store = createStore(
  reducer, 
  initialState, 
  applyMiddleware(thunk)
);
```

### 

### Reducer aufteilen und wieder zusammenfügen

### Verwendung mit React



