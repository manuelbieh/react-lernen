# Context API

Die **Context API** wurde in React sehr lange Zeit eher stiefmütterlich behandelt und wurde erst einmal nur prototypisch implementiert und als experimentell bezeichnet ehe sie in React **16.3.** in grundlegend überarbeiteter Form offizieller Teil von React wurde. 

Sie wurde dafür konzipiert, um Daten innerhalb einer Komponenten-Hierarchie in einer Anwendung an sog. _Konsumenten_ zu verteilen ohne dabei Props an jede einzelne Komponente manuell übergeben zu müssen. Dies kann in einigen Fällen sehr mühsam sein wenn es sich um Daten handelt die von vielen Komponenten innerhalb des Baumes gemeinsam immer wieder verwendet werden. Dazu gehören bspw. Sprach-Einstellungen oder ein globales Style-Schema \(„Theme“\).

In solchen Fällen bietet sich die **Context API** an, die jeweils aus für gewöhnlich einem **Context** _**Provider**_ und beliebig vielen **Context** _**Consumern**_ besteht. Der **Provider** dient hier als eine Art zentrale Instanz für die jeweiligen Daten, der **Consumer** kann dann die entsprechenden Daten _konsumieren_, die vom Provider bereitgestellt werden. Eine Anwendung kann dabei auch beliebig viele Kontexte besitzen \(bspw. einen für die vom Benutzer eingestellte Sprache, einen für das Style-Schema, etc.\) und auch ein Provider selbst kann wiederverwendet werden. Aber eins nach dem anderen.



