# maxxicharge-local-ha
Beispiel für die Integration des lokalen Maxxicharge API mit Homeassistant

# Änderungshistorie

- 29.11.2024: Das Release "CCU-FW 0.41"  friert den Stand der Dokumentation und Konfiguration für die CCU Firmware 0.41beta bzw. 0.41 ein.
- 30.11.2024: Aktueller Stand auf "main" realisiert die Integration mit CCU Firmware 0.42beta und 0.43beta.
- 20.12.2024: Configuration.Yaml um Log-Level "Info" ergänzt um Problemsuche zu vereinfachen

# Ziel

Das OpenSource SmartHome System "Home Assistant" (HA) bietet umfangreiche Optionen
zur Auswertung von "irgendwelchen" Sensordaten, aber insbesondere mit dem "Energy Dashboard"
eine einfache Möglichkeit den Energiefluß zwischen Netzanschluß, Solaranlage und Speicher zu
visualisieren und statistisch auszuwerten.

Ab CCU Firmware Release 0.41beta bietet Maxxicharge eine vollständig vom Maxxisun Cloudservice unabhängige Schnittstelle
zum "Auslesen" der Betriebsdaten des Maxxicharge per WebHook an. Die entsprechende Maxxisun Dokumentation
findet sich hier: https://maxxisun.app/cloud

Die hier dokumentierte Integration gliedert sich in 3 Teile:

## 1. Lokale Integration der Maxxicharge Betriebsdaten (Offiziell)

Die Maxxicharge Betriebsdaten werden von dem lokalen API in kurzen Abständen per REST Post an die
in der lokalen Konfiguration hinterlegte API-Route gesendet. Format und Inhalt findet sich in der 
Maxxisun Dokumentation und wird hier nicht wiederholt. 

Zugehörige Dateien mit weiterer Dokumentation:

- configuration.yaml
- z_conf_template.yaml

**Hinweis:** Damit die Daten per REST Post an den Webhook Endpunkt in HA übernommen werden, muss die Funktion auf http://maxxi.local
aktiviert werden:

- "Cloudservice" auf "Nein"
- "Lokalen Server nutzen" auf "Ja"
- "Api-Route" wie in der Anleitung von Maxxisun beschrieben / angepasst an die eigene Konfiguration / eintragen

### API-Route
**Generisch:**
- `http://<dein_ha>.local:8123/api/webhook/<deine_webhook_id>`
  
**Oder mit IP-Adresse statt lokalem Namen:**
- `http://<IP-Adresse>:8123/api/webhook/<deine_webhook_id>`

**Erklärung:**
- `<dein_ha>` ist der lokale Name deiner Homeassistant Instanz. Wenn man beim Default bleibt, ist dieser "homeassistant".
- `<IP-Adresse>` ist die IP-Adresse deiner Homeassistant Instantz im lokalen Netzwerk (die direkte Eingabe der IP-Adresse ist immer
dann notwendig, wenn sich Homeassistant NICHT über die http://<dein_ha>.local:8123 URL lokal aufrufen lässt.
- `<deine_webhook_id>` ist die Id aus dem Konfigurationsfile

**Beispiel bei Nutzung des standardmässigen Homeassistant URL Namens:**
  
- `http://homeassistant.local:8123/api/webhook/maxxicharge_local_id`


**Hinweis:** Homeassistant empfiehlt für Webhook IDs einen Namen mit der Qualität eines Passwortes zu verwenden. Im Beispiel
ist aber ein recht leicht zu erratender sprechender Name verwendet. Ein kryptischer Webhook Name ist auf jeden Fall notwendig,
wenn der Webhook von ausserhalb des lokalen Netzwerks aufgerufen werden soll, und der Webhook auch steuernde Funktionen
für HA hat. Dies ist bei der Integration mit "maxxi.local" beides nicht der Fall. Der Webhook ist als "local" konfiguriert
und liefert auch "nur" Sensordaten an HA - steuert also nichts Kritisches. Im übrigen kann ja jeder Anwender einen "kryptischen"
WebHook Namen verwenden - der muss dann allerdings an beiden Stellen (maxxi.local Konfiguration) und HA Konfiguration identisch sein.

## 2. Auslesen der aktuellen Maxxicharge Konfiguration (Inoffiziell)

Für die aktuelle Konfiguration des Maxxicharge Systems wird derzeit von Maxxisun keine offizielle
Schnittstelle (API) zur Verfügung gestellt. Um die Daten trotzdem in HA zu integrieren, bietet sich
die HA "Scrape" Integration an, mit der man Daten aus Webseiten extrahieren kann. Da die lokale
Maxxicharge Webseite (http://maxxi.local) keine Authorisierung benötigt und sich nur dann ändern
kann, wenn sich die Firmware Version ändert, ist "Scraping" ein relativ verlässlicher Weg,
um an die gewünschten Daten zu kommen. 

**Wichtig:** Wenn Maxxisun eine neue Firmware Version bereitstellt und diese vom Kunden
installiert wird, kann sich die lokale Webseite ändern und die Scraping Funktion damit unbrauchbar
werden.

Zugehörige Dateien mit weiterer Dokumentation:

- configuration.yaml
- z_conf_scrape.yaml

## 3. Ändern der aktuellen Maxxicharge Konfiguration (Inoffiziell)

Auch für das Ändern der Maxxicharge Konfiguration wird derzeit von Maxxisun keine offizielle
Schnittstelle (API) zur Verfügung gestellt. Man kann die Daten aber sehr einfach über HTTP POST
Aufrufe mit entsprechenden Query Parametern lokal ändern. 

**Wichtig:** Auch dies ist keine offizielle Schnittstelle und die Technik der http://maxxi.local
Webseite kann sich in zukünfigten CCU Firmwareversionen so ändern, dass der Ansatz nicht mehr
funktioniert.

**Achtung:** Im Unterschied zum Web-UI erfolgt keine Plausibilitäts- oder Grenzwertprüfung des
übergebenen Wertes. Dies obliegt dem Aufrufer. Evtl. Probleme / Fehler bei der Übergabe
ungeeigneter Werte sind daher in alleiniger Verantwortung des Aufrufers der Schnittstelle.

Zugehörige Dateien mit weiterer Dokumentation:

- configuration.yaml
- z_conf_rest_command.yaml

**Weitere Tipps:**

Wenn die "REST" Aufrufe gemäss Konfiguration zu Verfügung stehen müssen folgende weiteren Schritte
erfolgen, um die änderbaren Feldern in Betrieb zu bekommen:

a) Es müssen "input" "Helfer" Felder in HA erstellt werden. Dies kann per YAML oder per UI in 
der Konfiguration erfolgen.

YAML Beispiel:

```
input_number:
  maxxi_local_max_power:
    name: MaxxiCharge Lokal Maximale Leistung
    min: 300
    max: 1800
    step: 10
    mode: box
    unit_of_measurement: "W"
    icon: mdi:meter-electric
  maxxi_local_offline_power:
    name: MaxxiCharge Lokal Offline Leistung
    min: 100
    max: 600
    step: 10
    mode: box
    unit_of_measurement: "W"
    icon: mdi:transmission-tower-off
  maxxi_local_min_soc:
    name: MaxxiCharge Lokal Minimaler Ladestand
    min: 2
    max: 99
    step: 1
    mode: box
    unit_of_measurement: "%"
    icon: mdi:battery
  maxxi_local_max_soc:
    name: MaxxiCharge Lokal Maximaler Ladestand
    min: 10
    max: 100
    step: 1
    mode: box
    unit_of_measurement: "%"
    icon: mdi:battery
```
Die weiteren Felder habe ich per UI als "Helfer" unter "Einstellungen -> Geräte & Dienste -> Tab "Helfer" erstellt.

**Min/Max Werte:** Die Werte habe ich aus dem Lokalen UI übernommen. Damit ist sichergestellt, dass zumindest bei
UI Änderungen keine von Maxxisun nicht vorgesehenen Werte eingestellt werden können.

Für die beiden als Helfer angelegten Eingabefelder sind die Werte:

- "Ausgabe korrigieren" min: -500  max: 500
- "Reaktionstoleranz" min: 3  max: 50

b) Bei Änderungen eines der Felder im UI muss der entsprechende "REST" Aufruf per Automatisierung erfolgen (jedenfalls
habe ich noch keinen anderen Weg gefunden.)

Beispiel YAML für so eine Automatisierung:

```
alias: MaxxiLocal - Maximale Leistung ändern
description: Maximale Leistung per REST Call Lokal ändern
trigger:
  - platform: state
    entity_id:
      - input_number.maxxi_local_max_power
    for:
      hours: 0
      minutes: 0
      seconds: 5
condition: []
action:
  - data:
      wert: "{{states(\"input_number.maxxi_local_max_power\")}}"
    action: rest_command.maxxicharge_maximale_leistung
mode: single
```

Analoge Automatisierungen dann für alle anderen UI Felder (wenn man die Daten dort ändern will).

**Use Cases:**

Meiner Meinung nach sind Änderungen per UI aber nur als Test und kleine Spielerei zu sehen.
Relevant und nützlich sind Änderungen per Automatisierung. Ein Beispiel:

- Setze den "Ausgabe korrigieren" Wert tagsüber (bei guter Solarproduktion) auf einen negativen Wert,
  um den Netzbezug zu minimieren. Nachts wird der Werte auf eine positiven Wert gesetzt, um die Netzeinspeisung
  aus dem Speicher zu minimieren.

# Allgemeines

**Was ist hier nicht dokumentiert:**

Die von den Sensoren (Abschnitt 1.) gelieferten Leistungswerte sind der aktuelle Wert in W.
Um diese im Standard Energiedashboard vom HA nutzen zu können, müssen entsprechende "Helper" Entitäten
mittels "Integralsensor" - (Linkes Riemann Summenintegral) angelegt werden (W -> kWh). Zu dem Thema gibt es diverse
Videos auf YT - auch in deutsch (z.B. HA Youtuber: simon42,...). 

In HA gibt es dann neben den Statistiken des Energiedashboards auch noch Möglichkeiten explizite
Tages-, Monats-, Jahreswertsummenentitäten anzulegen. Auch dazu sind die diversen YT Videos zum Thema zu empfehlen.

Und sicher noch viel, viel mehr.

**Disclaimer:**

Die Nutzung der Konfiguration erfolgt auf eigene Verantwortung und ohne die implizite oder explizite
Zusicherung von bestimmten Eigenschaften und Funktionalitäten, sowie ohne Gewähr und Support.

Ich bin selbst relativ neu mit dem Homeassistant SmartHome System unterwegs. Die Realisierung der Anbindung
ist daher möglicherweise nicht konform zu Homeassistant "Best Practices". Weiterhin nutze ich nur Standardkonfigurationsmethoden.
Die vermutlich elegantere Lösung mittels HACS und ohne Änderung der Configuration.YAML war mir 
bisher nicht möglich. Wenn jemand eine HACS basierte Integration realisieren möchte, bin ich durchaus
daran interessiert dort unterstützend/lernend mitzuarbeiten.

Verbesserungsverschläge / Tipps sind natürlich willkommen. Diese dann entweder per PR oder als GitHub Issue.

**Hinweis:**
Technische Details zum den einzelnen Funktionen und zur Konfiguration sind als Kommentare im YAML eingefügt.
Grundsätzliches Wissen über HomeAssistant und die "configuration.yaml" setze ich allerdings voraus.
Wenn die fehlen, bitte selbst danach suchen - es gibt viele Videos, Foren,.. und auch viel in Deutsch.
