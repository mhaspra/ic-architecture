# Architekturentscheidungen

## ADR-01: Kafka als Integrationsplattform & Entkopplungsplattform
### Kontext
Wir integrieren Daten aus verschiedenen Legacy Applikationen. Diese erfüllen die Qualitätsanforderungen nicht.
Insbesondere haben sie **langsame Antworzeiten** und **keine 24/7 Verfügbarkeit.**\
Deswegen müssen die Daten repliziert werden um die Systeme zu entkoppeln.

Dafür wurden folgende Pattern betrachtet:

| Pattern | Braucht extra Datenspeicherung in den Client Applikationen | Elastisch | Fehleranfällig bei Ausfall Legacy System | Datenänderungen schnell verfügbar | Neue Applikationen können replizierte Daten brauchen (Event Sourcing) | Existieren Tools für die Integration | Team für Infrastruktur
|---|---|---|---|---|---|---|---|
| **Datenintegration mittels Kafka** | Nein, nur für explizite Use Cases | Ja | Nein | Ja | Ja | Ja viele Kafka-Connect Tools | Ja | Ja
| **Datenreplikation mit Batches** | Ja, für jedes lesende System | Nein, Batches laufen irgendwann > 24h | Ja. Batches mit Resart-Möglichkeit sind sehr komplex | Nein, höchstens mit Microbatching. Das ist komplex | Nein | Nein | Nein 
| **Datenreplikation mit Events und Rücklesepattern** | Ja, für jedes lesende System | Ja | Nein | Ja | Nein | Nein | Ja, minimal.

### Entscheidung
Wir integrieren die Daten mittels Kafka. Dies hat folgende Vorteile:
- Es braucht keinen extra Datenspeicher für Client Applikationen
- Neue Applikationen können jederzeit mit den Daten arbeiten und - falls nötig - mittels Event Sourcing jeden Zustand herstellen
- Neue Daten sind "soffort" verfügbar
  Die Datenspeicherung ist elastisch
- Es existieren gute Tools um Daten automatisch in Kafka und wieder raus zu laden
- Wir sind damit cloud ready. Zum Beispiel AWS MKS oder einfach Kafka auf K8s
### Konsequenzen
Es braucht **Kafka Know-How** und ein **Infrastruktur Team für die Betreuung**.\
Ausserdem eine weitere Entscheidung welches Produkt eingesetzte wird: AWS MKS, Confluent, ...

## ADR-02: Wir schneiden das System in - nicht zu kleine - Microservices
### Kontext
Um Features unabhängig und schnell entwickeln, testen und in Produktion bringen zu können müssen wir die Applikation sinnvoll schneiden.
### Entscheidung
Diese sollen zu Beginn möglichst gross sein und nur geschnitten werden wenn:
- Mehrere Teams daran entwickeln. -> Max. 1 Team pro Microservice
- Unterschiedliche Änderungsanforderungen einen Schnitt erzwingen.
- Unterschiedliche Anforderungen bezüglich Skalierbarkeit, Ressourcen oder Security bestehen.

### Konsequenzen
Wir beginnen mit relativ "grossen" Applikationen. Diese müssen gut strukturiert sein um einen späteren Schnitt zu ermöglichen. 

## ADR-03: Docker K8s als Runtime Umgebung
### Kontext
Wir müssen unsere Microservices irgendwo deployen.

### Entscheidung
Jeder Microservice wird als Dockercontainer in K8s deployed.\
So haben wir ein einheitliches Deployment Modell für alle Komponenten (inklusive Infrastruktur).\
Wir sind damit cloud ready.

### Konsequenzen
Alles muss dockerisiert werden. Das Ist aber onehin der de-facto Standart\
Wir müssen uns für eine K8s installation entscheiden und diese betreuen.

## ADR-04: Kafka Streams zur Datenaufbereitung
### Kontext
Wir werden für die Übersicht die Partner und die Contract Daten joinen müssen.

### Entscheidung
Dafür und für andere Use-Cases nehmen wir Kafka-Streams.\
Das ist:
- Mächtig
- Schnell
- Deklarativ
- Macht State Mgmt automatisch in Kafka

### Konsequenzen
Brauchen Kafka-Streams Know-How.

## ADR-05: Spring Boot als Backend Framework
### Kontext
Für die Microservices brauchen wir ein Backend Framework für die "Standardaufgaben"\
Dieses soll möglichst in Java sein.\
Es existieren verschiedene - mehr oder weniger - populäre Frameworks.\
Spring, OpenLiberty, Wildfly, Micronaut...

### Entscheidung
Wir nehmen Spring Boot.\
Es ist populär, hat eine aktive Community, Dokumentation und Hilfestellung ist reichlich vorhanden. Zudem bietet es gute Unterstützung für Kafka, Kafka-Streams & Elasticsearch.
### Konsequenzen
Keine relevanten, das können wir.

## ADR-06: Angular als Frontend Framework
### Kontext
Wir stellen die Daten in einer Webapplikation zur Verfügung.

### Entscheidung
Das machen wir mit Angular. Es ist weit verbreitet, wird aktiv weiterentwickelt und ist relativ einfach zu verstehen.

### Konsequenzen
Keine relevanten, das können wir.
Sollten wir zum Schluss kommen, dass wir WebComponents einsetzen wollen müssen wir diese Entscheidung nochmals überdenken.\
Dafür gibt es geeignetere Frameworks z.B stenciljs.

## ADR-07: Elasticsearch für den Übersicht Use Case
### Kontext
In der Web-Applikation zeigen wir eine Übersicht aller Kunden mit ihren Verträgen an.\
Diese Liste kann sehr schnell gross und unübersichtlich werden. Deswegen werden wir bald Pageging und ggf. eine Suche unterstützen müssen.\
Dafür sind Kafka Streams Interactive Queries nicht geeignet.

### Entscheidung
Wir replizieren die Übersichtsdaten dafür vom Kafka Topic in Elasticsearch.

### Konsequenzen
Die Replikation muss gebaut werden.\
Das bedeutet im Betrieb eine zusätzliche Komplexität. Insbesonders wenn Fehler und Dateninkonsistenzen gefunden werden müssen.

## ADR-08: Wir setzen Backend-For-Frontend ein
### Kontext
Die Web Applikation muss ihre Daten von irgendwo beziehen.\
Unser System ist aber - in Zukunft - in unterschiedliche Microservices geschnitten.

### Entscheidung
Wir setzen von Anfang an ein Backend-For-Frontend (BFF) ein.\
Das hat folgende Vorteile:
* Im BFF können wir dediziert auf die Anforderungen des Frontends eingehen wie zum Beispiel
  * Parallelisierung von Aufrufen um die Frontend Aufrufe zu verringern
  * Stabilere und besser aufs FE zugeschnittene Schnittstellen anbieten (z.B gruppieren aus verschiedenen Bounded-Context)
  * Schnelleres Prototyping


### Konsequenzen
Wir haben eine Release Unit mehr zu betreuen.

## ADR-9: Onion Architektur mit Ports & Adaptern als Applikationsarchitektur
### Kontext
Um die Applikationen verständlich und wartbar zu machen brauchen wir einen einheitlichen Architekturstil.

### Entscheidung
Wir setzen die Onion Architektur mit Ports & Adaptern ein.\
Damit entkoppeln wir low level Infrastruktur Code von high-level Domain Code.

### Konsequenzen
Architekturstil muss allen bekannt sein.\
Details müssen in den Teams festgelegt werden.