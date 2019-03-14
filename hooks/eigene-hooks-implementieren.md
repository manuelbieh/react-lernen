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





