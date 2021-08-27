01 Kickoff Meeting

## Anwensend
- Jan Unterbrink
- Armin Schlegel
## Agenda
- Arbeitsweise
	- Kanban / Scrum
		* Kanban
			* September ist eine Messe
			* Bis Mitte September Projekt (Planning)
		* aktuell 5 Tage
		* parallel
	- Dailies
	- Kommunikation
		- Teams
			Teams Kommunikation
	- Kernarbeitszeiten?
		09:00 - 15:00
- Wohin kommt der Code?
	- Github
- Wo / Wie und in welcher Ausführlichkeit soll dokumentiert werden
	- in repo

## Was tut Ci4Rails
	- EdgeFarm Devices
		- Hardware mit einem Jocto und Sensorik
		- Firmware updates Jocto + Sensorik
		- Livecycle vom Device
	- EdgeFarm Applications
		- YAML -> Applikationen -> starten auf Applikation Runtime
			- Runtime:
				- Cloud
				- Edge
			- Kubernetes orientiert
	- EdgeFarm Data:
		- Datenfoo (ML und so)
		- Bentos???
	- nats.io

	- cli tool: edgefarm
		orientiert an kubectl

## Probleme:
	- IoT Edge skalliert nicht
		- n devices
		- max. 100 Deployments
		
## Status Firma
- aktuell keine 

## Programmiersprache
	- fast alles in GO bis auf Kundenprogramme (Python)
	- März oder Februar erst richtig angefangen zu arbeiten

##  KubeEdge Probleme
	- KubeEdge TLS Connection closed
	- KubeEdge entfernt nodes, welche noch laufen

	- Nodes werden nicht hinzugefügt

	- Cloudcore Parse messge: .....
	- Cloudcore schmeißt nodes weg, wenn mehr als 10

## 3 Umgebungen
	- dev
	- test
	- prod

	serivces werden auch hochgezogen

	tennants eventuell über vCluster (über synchronisation)
	
## Ziel
- KubeEdge stabil laufen
