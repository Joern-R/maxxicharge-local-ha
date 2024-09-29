# maxxicharge-local-ha
Beispiel für die Integration des lokalen Maxxicharge API mit Homeassistant

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


## 2. Auslesen der aktuellen Maxxicharge Konfiguration (Inoffiziell)

Für die aktuelle Konfiguration des Maxxicharge Systems wird derzeit von Maxxisun keine offizielle
Schnittstelle (API) zur Verfügung gestellt. Um die Daten trotzdem in HA zu integrieren bietet sich
die HA "Scrape" Integration an, mit der man Daten aus Webseiten extrahieren kann. Da die lokale
Maxxicharge Webseite (http://maxxi.local) keine Authorisierung benötigt und sich nur dann ändern
kann, wenn sich die Firmware Version ändert, ist die "Scraping" ein relativ verlässlicher Weg
um an die gewünschten Daten zu kommen. 

**Wichtig:** Wenn Maxxisun eine neue Firmware Version bereitstellt und diese vom Kunden
installiert wird, kann sich die lokale Webseite ändert und die Scraping Funktion damit unbrauchbar
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

Zugehörige Dateien mit weiterer Dokumentation:

- configuration.yaml
- z_conf_rest_command.yaml


# Allgemeines

**Disclaimer:**

Die Nutzung der Konfiguration erfolgt auf eigene Verantwortung und ohne die implizite oder explizite
Zusicherung von bestimmten Eigenschaften und Funktionalitäten, sowie ohne Gewähr und Support.

Ich bin selbst relativ neu mit dem Homeassistant SmartHome System unterwegs. Die Realisierung der Anbindung
ist daher möglicherweise nicht konform zu Homeassistant "Best Practices". Weiterhin nutze ich nur Standardkonfigurationsmethoden.
Eine möglicherweise elegantere Lösung mittels HACS und ohne Änderung der Configuration.YAML war mir 
bisher nicht möglich. Wenn jemand eine HACS basierte Integration realisieren möchte, bin ich durchaus
daran interessiert dort unterstützend/lernend mitzuarbeiten.

Verbesserungsverschläge / Tipps sind natürlich willkommen. Diese dann entweder per PR oder als GitHub Issue.

**Hinweis:**
Technische Details zum den einzelnen Funktionen und zur Konfiguration sind als Kommentare im YAML eingefügt.
Grundsätzliches Wissen über HomeAssistant und die "configuration.yaml" dort setze ich allerdings voraus.
Wenn die fehlen, bitte selbst danach suchen - es gibt viele Videos, Foren,.. und auch viel in Deutsch.
