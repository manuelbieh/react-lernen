# Tools und Frameworks

Um React herum hat sich mittlerweile ein ganzes Ökosystem an Tools gebildet. Vom Static Site Generator, um simple bis medium-komplexe statische Websites auf Basis von React-Komponenten zu erzeugen, über Prototyping-Tools bis hin zu Tools um die Komponenten der eigenen Anwendung in einer Art Styleguide übersichtlich darzustellen, ist hier vieles vertreten. In diesem Unterkapitel möchte ich eine _kurze_ Übersicht über die bekanntesten Tools und Frameworks aus der React-Welt geben. 

### Storybook

> Storybook is a development environment for UI components. It allows you to browse a component library, view the different states of each component, and interactively develop and test components.

**Storybook** ist ein Tool zur Erstellung von Isolierten UI-Komponenten für React, Vue.js und Angular. **Storybook** dient dabei als eine Art Sandbox-Umgebung, in der Komponenten für sich alleine in sog. **Stories** entwickelt werden können und stellt diese in einem übersichtlichen und einfach zu bedienendem Interface dar. Durch die Isolation wird ein sehr hoher Grad an Abstraktion ermöglicht, der es auch erlaubt Edge-cases darzustellen und zu testen.

**Lizenz:** MIT \(Open Source\)  
**URL:** [https://storybook.js.org/](https://storybook.js.org/)  
**GitHub:** [https://github.com/storybooks/storybook](https://github.com/storybooks/storybook)

### React Styleguidist

> React Styleguidist is a component development environment with hot reloaded dev server and a living style guide that you can share with your team.

**Styleguidist** geht in die selbe Richtung wie Storybook. Auch mit Styleguidist ist es möglich, einen Styleguide aus den UI-Komponenten der eigenen Anwendung zu erstellen. Allerdings ist **Styleguidist** hier etwas impliziter als **Storybook** und so wird der Styleguide schon zu einem recht hohen Maß aus den **PropTypes** von Komponenten sowie aus **JSDoc**-Kommentaren angereichert. Für alles Weitere dienen dann Markdown-Dateien im Ordner der jeweiligen Komponente.

**Lizenz:** MIT \(Open Source\)  
**URL:** [https://react-styleguidist.js.org/](https://react-styleguidist.js.org/)  
**GitHub:** [https://github.com/styleguidist/react-styleguidist](https://github.com/styleguidist/react-styleguidist)

### Docz

> It has never been so easy to document your things!

**Docz** ist ein Tool, das, wie der Name es bereits erahnen lässt, der Dokumentation von Komponenten dient. Auch **Docz** ist letztlich eine Art Styleguide, wie bereits die ersten beiden Tools. Dennoch sticht es irgendwo heraus, denn es ist komplett **MDX**-basiert. MDX ist eine um React-Komponenten erweiterte Version des Markdown-Formats. Komponenten werden in `.mdx`-Dateien beschrieben und können wie in JavaScript selbst importiert und verwendet werden.

**Lizenz:** MIT \(Open Source\)  
**URL:** [https://docz.site/](https://docz.site/)  
**GitHub:** [https://github.com/pedronauck/docz](https://github.com/pedronauck/docz)

### React Cosmos

> Dev tool for creating reusable React components

Noch ein Stück weiter geht **React Cosmos**. Während die ersten drei genannten Tools ihren Fokus auf die Kapselung und die Darstellung reiner UI-Komponenten legen, bricht **React Cosmos** diese Kapselung bewusst auf und erlaubt es auch externe Abhängigkeiten wie React Router oder Redux durch das Konzept von _Fixtures_ und _Proxies_ darzustellen und zu testen.

**Lizenz:** MIT \(Open Source\)  
**GitHub:** [https://github.com/react-cosmos/react-cosmos](https://github.com/react-cosmos/react-cosmos)

### Gatsby

> Gatsby is a free and open source framework based on React that helps developers build blazing fast websites and apps

**Gatsby** gehört in die Kategorie der sog. Static Site Generators, also einem Generator für statische Websites. Diese werden in **Gatsby** aus React-Komponenten und GraphQL erzeugt und bestehen letztendlich aus statischen HTML-Dateien. Dabei erzeugt **Gatsby** auch ein JavaScript-Bundle, das beim Aufruf deiner Seite geladen wird. Ist das Bundle erst einmal geladen, macht **Gatsby** Gebrauch von clientseitigem Rendering, was zur Folge hat, dass die erzeugten Seiten unglaublich schnell \(„blazing fast“\) angezeigt werden, da ab diesem Zeitpunkt der HTTP-Overhead entfällt. **Gatsby** wurde als Open Source Projekt gestartet \(und ist dies auch bis heute\) und erhielt als solches im Mai 2018 eine beeindruckende Finanzierung von 3,8 Mio USD.

**Lizenz:** MIT \(Open Source\)  
**URL:** [https://www.gatsbyjs.org/](https://www.gatsbyjs.org/)  
**GitHub:** [https://github.com/gatsbyjs/gatsby](https://github.com/gatsbyjs/gatsby)

### React-Static

> React-Static is a fast, lightweight, and powerful progressive static site generator based on React and its ecosystem.

**React-Static** ist eine Alternative zu **Gatsby**. Auch **React-Static** ist ein **Static Site Generator,** mit dem statische Websites basierend auf React generiert werden können. Durch seine hohe Finanzierungsrunde ist **Gatsby** sicherlich das bekanntere der beiden Tools, jedoch ist auch **React-Static** eine durchaus ernste Alternative in diesem Bereich mit einer großen und sehr aktiven Community.

**Lizenz:** MIT \(Open Source\)  
**GitHub:** [https://github.com/nozzle/react-static](https://github.com/nozzle/react-static)

### Next.js

Next.js bezeichnet sich selbstbewusst als „The React Framework" für diverse Anwendungsfälle wie statische und dynamische Websites, große wie kleine Unternehmen, mobile oder klassische Websites und vieles mehr. Und in der Tat ist dieses Framework in der React-Community mit über 36.000 Stars auf GitHub ein anerkanntes Tool zur Entwicklung von Anwendungen mit React und eine echte Alternative zu Create React App. Anders als CRA kommt Next.js gleich mit Unterstützung von serverseitig gerenderten Seiten, was es insbesondere für SEO-relevante Themen interessant werden lässt.

**Lizenz:** MIT \(Open Source\)  
**URL:** [https://nextjs.org/](https://nextjs.org/)  
**GitHub:** [https://github.com/zeit/next.js](https://github.com/zeit/next.js)

