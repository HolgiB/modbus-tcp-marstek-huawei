Huawei Sun & Marstek Venus Modbus TCP → MQTT Bridge (Node-RED Flows)

Dieses Repository enthält reine Node-RED Flow-Exports (JSON) zur Anbindung von Huawei & Marstek Venus Wechselrichtern und Batteriesystemen über Modbus TCP.

Die Flows richten sich explizit an erfahrene Node-RED Anwender, die:
ihre Laufzeitumgebung selbst betreiben
MQTT & Modbus TCP bereits kennen
keine Docker- oder Python-Wrapper benötigen

🎯 Ziel
stabile Modbus-TCP-Abfrage (≥ 60 s Polling)
saubere MQTT-Topics
transparente Registerverarbeitung
vollständige Kontrolle im Flow

Kein:
modbus2mqtt
YAML / CSV Config-Parser
gepinnte Python-Versionen
versteckte Defaults

Voraussetzungen
Laufende Node-RED Instanz

Installierte Nodes:
node-red-contrib-modbus
node-red-node-mqtt

MQTT Broker (z. B. Mosquitto)
Huawei Sun Wechselrichter mit aktiven Modbus TCP Adapter
beziehungsweise
Marstek Venus Speicher mit aktueller Firmware (> V1.47)

Einleitung: 
Hintergrund für die Erstellung dieser Node-RED Flows war die Hauptproblematik, dass Modbus TCP immer eher umständlich über irgendwelche Docker Container mit Python und ModbusTCP to MQTT erfolgt. Dies war initial mit meinem Huawei Sun Inverter mit Luna Speicher und Modbus TCP Adapter der Fall. Der von mir verwendete Docker Container konnte sauber die Daten auslesen, aber war relativ fix in den ausgelesenen Werten. Anpassungen gingen nur durch Neubau des Docker Images. Nach dem Kauf des Marstek Venus C V3.0 hab ich begonnen diesen in meine Heimautomation zu integrieren. Der Umweg über die Local API mit JSON Requests und einem UDP Port war eher sperrig und gelegentlich auch etwas instabil. Deshalb versuchte ich diverse Home Assistant Integrationen. Dahinter verbirgt sich häufig ein Docker Container der über HACS installiert wird. Mit der Installation über HACS hat man gleich die passenden Komponenten in Home Assistant inklusive Sensoren. Leider musste ich schnell lernen das mein Venus C V3.0 teilweise andere Register Adressen hat als ältere Marstek Speicher. Also machte ich mich weiter auf die Suche, testete gut drei unterschiedliche Home Assistant Integrationen / ModbusTCP-MQTT Bridges. Alle funktionierten mal mehr oder weniger gut. Beim Versuch die ausgelesenen Register anzupassen und meinen eigenen Docker Container zu bauen fiel mir erstmal auf was für ein Chaos dort herrscht. Manche ModbusTCP-MQTT Python Pakete wurden nicht mehr gepflegt, es gab Forks davon. Stellenweise Konfigurationsdateien in YAML. Stellenweise Konfigurationsdateien im CSV Format. Manche Dockerfiles pinnen dann eine fixe Version von ModbusTCP-MQTT. Meist wird gnadenlos per PIP als Root installiert anstelle ein VENV zu verwenden, aber genug des Nerd Genörgels. Final bin ich dann unterstützt von ChatGPT doch bei Node-RED gelandet. Nach einigen Fehlschlägen hatte ich dann lauffähige Flows für meinen Marstek Venus 3.0. Angespornt durch die erfolgreiche Anbindung des Marstek hab ich dann noch kurzerhand meinen Huawei Sun per Modbus TCP angebunden und den alten Dockercontainer eliminiert.

Anpassungen an den Flows / Lessons learned:
Ich habe mal alle Dokumente die mir bezüglich der Register für die beiden Produkte in die Finger gefallen sind in den Ordner Misc gepackt. Optimalerweise sollte man immer möglichst viele Register bei Modbus TCP in einem Rutsch auslesen und dann im Nachgang in die einzelnen Bytes zerlegen und konvertieren. Modbus TCP reagiert häufig auch sehr sensibel auf zu häufige Anfragen. Quasi alle Register alle 5 Sekunden auszulesen führt zumindest beim Huawei Adapter unweigerlich zu Lesefehlern. Ich habe mich aus Gründen der einfacheren Umsetzung gegen das Auslesen größerer Registerblöcke entschieden und lese ganz gezielt nur einzelne Registeradressen aus. Dies allerdings auch nur alle 60 Sekunden. Für die meisten Anwendungen sollte das auch ausreichen wie die Datenerfassung per InfluxDB, die Visualisierung in Grafana oder die einfache Darstellung als Home Assistant Dashboard. Die Register des Huawei Sun dürften recht stabil sein, was Werte, Bedeutung und Format betrifft. Beim Marstek wäre ich mir da nicht sicher. Ich würde bei der Implementierung neuer Register immer Schritt für Schritt einzelne Register auslesen und in Node-RED mittels Debug Nodes auf Plausibilität prüfen. Man hat ja meist eine App zum Vergleichen ob die ausgelesenen Werte Sinn machen (Solarleistung, SOC, etc). Manchmal erlebt man dann auch einen Überraschung und die geladene Kapazität wird wie beim Marstek einfach über die Maximalkapazität (ohnehin nur ein programmierter Fixwert) über den SOC ausgerechnet. Das müsst ihr dann halt in Node-RED nachbauen. Ich nutze bewusst nur lesenden Registerzugriff. Sobald ihr zum Beispiel die Einspeiseleistung per ModbusTCP steuern wollt, wäre ich sehr vorsichtig bei der Umsetzung. Lieber erstmal kleine Werte probieren bevor ihr den Wechselrichter auf volle 3 kW Einspeiseleistung stellt ;-)

Apropos Einspeiseleistung: Wer den Marstek ohne gekoppelten Stromzähler / IR-Lesekopf bezüglich Lade- und Entladeleistung steuern möchte, sollte sich mal das Unimeter Projekt anschauen:
https://github.com/sdeigm/uni-meter

Ich nutze für Steuerung meines Marstek die Local API mit Kopplung an ein MQTT Topic.

MQTT Topics Huawei Sun:
"huawei/pv-power" 	-
"huawei/grid-power"      - Leistung Stromzähler (- = Netzbezug / + = Einspeisung)
"huawei/battery-1-soc"   - SOC Batterie #1
"huawei/battery-1-temp"  - Temperatur Batterie #1
"huawei/inverter-temp"   - Interne Inverter Temperatur
"huawei/battery-1-power" - Lade- und Endladeleistung der Batterie

MQTT Topic Marstek Venus:
"marstek-modbus/battery/soc" 		  - SOC Batterie
"marstek-modbus/battery/capacity" 	  - Maximalkapazität der Batterie (Fixwert)
"marstek-modbus/system/temp" 		  - System Temperatur
"marstek-modbus/grid_power"               - Leistung Akku (- = Netzbezug / + = Einspeisung)
"marstek-modbus/battery/capacity_charged" - Geladene Batteriekapazität 

Referenzen:
https://debacher.de/wiki/Sun2000_Modbus_Register
https://github.com/ViperRNMC/marstek_venus_modbus
https://www.forwardme.de/2025/04/03/marstek-venus-speicher-emulierter-shelly-synology-diskstation-ueberschussladen/

⚠️ Hinweise
Huawei Register variieren je nach Firmware & Modell
Polling unter 30 s kann zu Verbindungsabbrüchen führen
Nutzung auf eigenes Risiko

🤝 Contributions
Pull Requests willkommen für:
neue Register
andere Huawei Modelle
Verbesserungen der Flow-Struktur
Dokumentations-Updates

📜 Lizenz
MIT License
Keine Garantie. Keine Haftung.
Verwende es, ändere es, veröffentliche es weiter.