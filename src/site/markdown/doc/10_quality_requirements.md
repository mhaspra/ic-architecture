# Qualitätsanforderungen
Folgende Qualitätsziele sind für die Lösung wichtig:
1. 7/24 verfügbar
2. Kurze Zugriffszeiten
2. Near Realtime Daten
3. Resilienz gegenüber Fehlern

### 7/24 Verfügbarkeit
Die - mit K8s - gewählte Deployment Umgebung ermöglicht eine 7/24 Verfügbarkeit für alle Komponenten.
* Es können mehrere Replikationen der Microservices betrieben werden.
* Die Datenspeicher Kafka und Elasticsearch sind out of the box Cluster-fähig.
* Der K8s Cluster selbst kann auf allen 3 grossen Cloud Providern betrieben werden. Kafka und Elasticsearch sind dabei Multi-Region fähig.

### Kurze Zugriffszeiten
Durch die Speicherung in Kafka und später in Elasticsearch ist das System von den langsamen Zugriffszeiten der Legacysysteme entkoppelt.\
Sowohl Elasticsearch als auch Kafka haben eine sehr gute Performance und können je nach Resourcenanforderungen elastisch horizontal skaliert werden.

### Near Realtime Daten
Durch den Streaming Ansatz - im Gegensatz zur Replikation mittls Batches - sind die Daten beinahe real-time in Kafka und dann in Elasticsearch vorhanden.

### Resilienz gegenüber Fehlern/Ausfällen
|Event|Einfluss auf die Legacy Systeme|Einfluss auf unser System|
|---|---|---|
|Unterbruch der Verbindung Legacy/Kafka Connect|Keinen, da vom WAL gelesen wird wird das ger nicht bemerkt| Kafka Connect macht automatischen Resume |
|Crash/Shutdown Kafka Connector|Keinen, da vom WAL gelesen wird wird das ger nicht bemerkt| Kafka Connect macht automatischen Resume, **kann aber Duplikate schicken** |
|Ausfall/Wartun Legacy Systeme|Siehe Verbindungsunterbruch| Siehe Verbindungsunterbruch |