# Rendering von Elementen

In einigen der vorherigen Artikel habe ich sie wie selbstverständlich bereits einige Male erwähnt, doch was genau sind **React Elemente** eigentlich?

React Elemente sind der kleinstmögliche Baustein in einer React Anwendung. Anhand der Elemente beschreibt ihr, was der Benutzer später auf dem Bildschirm zu sehen bekommt. Trotz ihres gleichen Namens unterscheiden sie sich von DOM Elementen in einem wesentlichen Punkt: sie sind lediglich ein einfaches Objekt und damit auch günstig \(im Sinne der Performance\) zu erstellen.

{% hint style="warning" %}
**Vorsicht:** React **Elemente** werden oftmals mit React **Komponenten** durcheinander geworfen oder im Sprachgebrauch analog verwendet. Das ist aber nicht korrekt! **Elemente** sind das, aus dem **Komponenten** „gemacht sind“. Komponenten werden im nächsten Kapitel noch ausführlich behandelt, bevor es damit weitergeht, solltest du jedoch zuerst dieses Kapitel über **Elemente** gelesen haben.
{% endhint %}

