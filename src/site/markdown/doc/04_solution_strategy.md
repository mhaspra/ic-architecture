# Lösungsstrategie

Die Lösung sieht wie folgt aus:

## Frontend
Als Frontend Technologie wird Angular eingesetzt.

## Backend
Das Backend bilden eine kleine Anzahl von - auf Spring Boot basierenden - Microservices.
Eine Onion Architektur mit taktischen Domain Driven Design Pattern gewährleistet eine optimale Wartbarkeit.

## K8s als Deployment Umgebung
Die Backend Microservices werden in einem K8s Cluster deployed. Damit wird eine hohe Zuverlässigkeit, Wartbarkeit und Elastizität erreicht.

## Entkopplung von den Legacy Systemen
Die Entkopplung von den Legacy Systemen wird mittels Kafka erreicht. Das ermöglicht zudem zusätzliche Use Cases auf den integrierten Daten