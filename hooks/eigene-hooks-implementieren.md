# Eigene Hooks implementieren

Neben den **internen Hooks** wie `useState` oder `useEffect` ist es auch möglich **eigene Hooks** zu erstellen, die selbst wiederum Gebrauch von den **internen Hooks** oder auch von anderen **eigenen Hooks** machen und eigene Logik jeglicher Art in wiederverwendbarer Form kapseln. Sogenannte **Custom Hooks**. Die Entwicklung von solchen **Custom Hooks** ist immer dann besonders sinnvoll wenn die immer gleiche Logik in mehreren Komponenten verwendet werden soll. Auch wenn die Logik in einer Komponente zunehmend komplexer wird kann es sinnvoll sein diese aufzuteilen und in **eigene Hooks** mit leicht verständlichen Namen zu verlagern, um die eigentliche **Function Component** übersichtlicher zu halten.

### Der erste eigene Custom Hook

Fangen wir zum Einstieg mit einem sehr simplen Beispiel an und gehen davon aus, dass wir einen **Custom Hook** implementieren wollen mit dem wir einen Side Effect auslösen und die Hintergrundfarbe ändern wann immer eine Komponente gemounted wird. Ein typischer Name für einen solchen **Custom Hook** – wir erinnern uns daran, dass Hooks immer mit `use` beginnen müssen – wäre bspw. `useBackgroundColor()`. Der Hook erwartet eine gültige CSS-Farbe und setzt diese als Hintergrundfarbe sobald eine Komponente, die Gebrauch von diesem Hook macht, gemounted wird oder einen neuen Wert übergibt an den **Custom Hook** übergibt.

Da wir diese Logik möglicherweise in mehreren Komponenten verwenden möchten und dort nicht jedes mal die selbe Funktionalität der immer gleichen `useEffect`-Funktion implementieren wollen, erstellen wir also den folgenden **Custom Hook** als erste Fingerübung:

```javascript
// useBackgroundColor.js
import { useEffect } from 'react';

const useBackgroundColor = (color) => {
  useEffect(
    () => {
      document.body.style.backgroundColor = color;
      return () => {
        document.body.style.backgroundColor = '';
      }
    }, 
    [color]
  );
}

export default useBackgroundColor;
```

Als Beispiel für die Verwendung erstellen wir eine kleine `Tabs`-Komponente die drei Buttons anzeigt, die beim Klick jeweils einen anderen Content anzeigen. Je nachdem welche Komponente angezeigt wird, wollen wir den Hintergrund unserer Anwendung ändern. Dazu nutzen wir unseren eben erstellten `useBackgroundColor()` Hook. 

Das Ganze gestaltet sich dann wie folgt:

```jsx
// Tabs.js
import React, { useState } from "react";
import ReactDOM from "react-dom";
import useBackgroundColor from "./useBackgroundColor";

const DefaultContent = () => {
  return <p>Default Content</p>;
};

const SpecialContent = () => {
  useBackgroundColor("red");
  return <p>Special Content</p>;
};

const OtherSpecialContent = () => {
  useBackgroundColor("orange");
  return <p>Other Special Content</p>;
};

const Tabs = () => {
  const [tab, setTab] = useState("home");

  return (
    <div className="tabs">
      <div className="tabBar">
        <button onClick={() => setTab("home")}>Home</button>
        <button onClick={() => setTab("special")}>Special</button>
        <button onClick={() => setTab("other")}>Other Special</button>
      </div>
      <div className="tabContent">
        {tab === "home" && <DefaultContent />}
        {tab === "special" && <SpecialContent />}
        {tab === "other" && <OtherSpecialContent />}
      </div>
    </div>
  );
};

ReactDOM.render(<Tabs/>, document.getElementById("root"));
```

Wir haben in diesem Beispiel drei simple Kompontenten implementiert, die unseren Content darstellen: `DefaultContent`, `SpecialContent`, `OtherSpecialContent`. Zwei dieser Komponenten nutzen dabei unseren im ersten Schritt erstellten **Custom Hook** `useBackgroundColor()`  um die globale Hintergrundfarbe in einem `useEffect()`-Hook zu ändern sobald die Komponente gemounted wird.

Alternativ könnten wir an dieser Stelle auch den `useEffect()`-Hook den wir im `useBackgroundColor()`-Hook benutzen auch händisch überall dort implementieren wo wir die Hintergrundfarbe ändern müssen. Dies würde aber eben zu sehr viel Duplikation im Code führen. Stattdessen lagern wir die entsprechende Logik in eine eigene Hook-Funktion aus, machen diese bedingt konfigurierbar indem wir die gewünschte Farbe übergeben und können sie dann in beliebig vielen Function Components verwenden.

Während die einfache Erstellung wiederverwendbarer Komponenten auf Darstellungsebene \(also im User Interface Layer\) also die große Stärke von React und JSX ist, so bieten uns Hooks auf Logik-Ebene erstmals eine Möglichkeit um mit React-Bordmitteln Wiederverwendbarkeit gewährleisten zu können ohne Kompromisse eingehen zu müssen.

### Mit Daten aus Hooks arbeiten

Datenübergabe in **Custom Hooks** ist keine Einbahnstraße. In unserem ersten eigenen Hook haben wir gesehen wie wir Daten \(in diesem Fall eine Farbe\) an den **Custom Hook** übergeben. Nämlich als einfachen Funktionsparameter. Der Hook kann aber ebenso Daten zurückgeben mit denen dann in der Komponente gearbeitet werden kann. In welcher Form Daten aus dem **Hook** zurückgegeben werden ist dabei völlig dem Entwickler überlassen. Vom einfachen String über einen Tupel wie beim internen `useState()`-**Hook** bis hin zu ganzen React-Komponenten oder Elementen oder auch einem Mix aus allem sind der eigenen Fantasie hier keine Grenzen gesetzt.

Nehmen wir an wir wollen auf die Daten einer API zugreifen. Auf welche genauen Daten zugegriffen wird wollen wir parametrisierbar machen. Der Hook soll dafür sorgen uns die Daten zu besorgen – egal in welcher Komponente wir ihn verwenden – und dann an die Komponente die die Daten benötigt zurückgeben. In einem realen Beispiel könnten das z.B. Benutzerdaten von GitHub sein. Dies soll unser nächster eigener Hook werden: `useGitHubUserData`. 

Wir übergeben dem Hook einen GitHub-Benutzernamen und bekommen ein Objekt mit allen relevanten Informationen zu dem Benutzer zurück. Der Hook kümmert sich darum die Daten über die GitHub API zu besorgen und anschließend an die Komponente zurück zu übergeben:

```jsx
// useGitHubAccountData.js
import { useEffect, useState } from "react";
import axios from "axios";

const useGitHubAccountData = (account) => {
  const [accountData, setAccountData] = useState({});

  useEffect(
    () => {
      if (!account) {
        return;
      }

      axios.get(`https://api.github.com/users/${account}`).then((response) => {
        setAccountData(response.data);
      });
    },
    [account]
  );

  return accountData;
};

export default useGitHubAccountData;
```

Auch hier nutzen wir wieder die bereits bekannten Hooks `useEffect()` und `useState()`. Den State \(konkret `accountData`\) nutzen wir um darin die GitHub-Benutzerdaten zu verwalten. Den Effekt führen wir immer dann aus \(und nur dann\) wenn sich der übergebene Benutzername ändert. Zum übergebenen Benutzernamen holen wir uns dann über die GitHub API die Benutzerdaten ab, warten den Response ab und schreiben die Daten aus dem Response per `setAccountData()` in den State. Am Ende des Hooks wird dann der `accountData` State zurückgegeben an die Komponente die den Hook aufgerufen hat.

Die Daten stehen nun in der Komponente zur Verfügung und können in dieser beliebig verwendet werden:

```jsx
// RepoInfo.js
import React from "react";
import ReactDOM from "react-dom";
import useGitHubAccountData from "./useGitHubAccountData";

const RepoInfo = () => {
  const accountData = useGitHubAccountData("manuelbieh");

  return (
    <p>
      Der GitHub-Benutzer {accountData.name} besitzt {accountData.public_repos}{" "}
      öffentliche Repositories.
    </p>
  );
};

ReactDOM.render(<RepoInfo />, document.getElementById("root"));
```

An dieser Stelle können wir nun basierend auf der `RepoInfo`-Komponente hergehen und weitere Funktionalität implementieren. Statt eines festen GitHub-Accounts wollen wir den Benutzer vielleicht einen Account suchen lassen. Dazu legen wir uns mit dem `useState()`-Hook einen neuen **State** an, in den wir den vom Benutzer eingegebenen Account schreiben und geben diesen State an unseren **Custom Hook** weiter.

Da der `useEffect()`-Hook als **Dependency** den Account-Namen hat wird dieser immer dann ausgeführt wenn ein neuer Account-Name übergeben wird. Das bedeutet, dass mit jeder Änderung im Suchfeld ein neuer API Request initiiert wird, der uns die Daten zum eingegebenen Account besorgt:

```jsx
// RepoLookup.js
import React, { useState } from "react";
import ReactDOM from "react-dom";
import useGitHubAccountData from "./useGitHubAccountData";

const RepoLookup = () => {
  const [account, setAccount] = useState("");
  const accountData = useGitHubAccountData(account);

  return (
    <div>
      <input value={account} onChange={(e) => setAccount(e.target.value)} />
      {!account ? (
        <p>Bitte einen GitHub-Benutzernamen eingeben</p>
      ) : (
        <p>
          Der GitHub-Benutzer {accountData.name} ({accountData.login}) besitzt{" "}
          {accountData.public_repos} öffentliche Repositories.
        </p>
      )}
    </div>
  );
};

ReactDOM.render(<RepoLookup />, document.getElementById("root"));
```

Kurze Info am Rande: die Public API von GitHub erlaubt nur max. 60 API Requests pro Stunde. Wer hier also ernsthaft rumprobieren will kann sich einen API Token erstellen und diesen per `access_token=xyz` an die URL dranhängen. Der Token kann hier erstellt werden: [https://github.com/settings/tokens](https://github.com/settings/tokens).



