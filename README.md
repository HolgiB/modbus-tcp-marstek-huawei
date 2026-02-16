# Huawei Sun & Marstek Venus Modbus TCP → MQTT Bridge (Node-RED Flows)

Dieses Repository enthält **reine Node-RED Flow-Exports (JSON)** zur Anbindung von **Huawei Sun** & **Marstek Venus** Wechselrichtern und Batteriesystemen über **Modbus TCP**.

Die Flows richten sich explizit an **erfahrene Node-RED Anwender**, die:

- ihre Laufzeitumgebung selbst betreiben  
- MQTT & Modbus TCP bereits kennen  
- **keine Docker- oder Python-Wrapper** benötigen  

---

## 🎯 Ziel

- stabile Modbus-TCP-Abfrage (**≥ 60 s Polling**)  
- saubere MQTT-Topics  
- transparente Registerverarbeitung  
- vollständige Kontrolle im Flow  

### ❌ Kein Einsatz von

- `modbus2mqtt`  
- YAML / CSV Config-Parser  
- gepinnten Python-Versionen  
- versteckten Defaults  

---

## ✅ Voraussetzungen

- laufende **Node-RED** Instanz  

### Installierte Nodes

- `node-red-contrib-modbus`  
- `node-red-node-mqtt`  

### Hardware / Systeme

- MQTT Broker (z. B. Mosquitto)  
- **Huawei Sun** Wechselrichter mit aktivem Modbus-TCP-Adapter  
- **oder**  
- **Marstek Venus** Speicher mit aktueller Firmware (**> V1.47**)  

---

## 📘 Einleitung

Hintergrund für die Erstellung dieser Node-RED Flows war die Hauptproblematik, dass **Modbus TCP** häufig über Docker-Container mit Python und *ModbusTCP → MQTT* realisiert wird.

Das war initial auch bei meinem **Huawei Sun Inverter mit Luna Speicher und Modbus TCP Adapter** der Fall.  
Der verwendete Docker-Container konnte die Daten zwar zuverlässig auslesen, war jedoch **sehr fix** in den definierten Registern. Anpassungen waren nur durch **Neubau des Docker-Images** möglich.

Nach dem Kauf des **Marstek Venus C V3.0** begann ich, diesen in meine Heimautomation zu integrieren.  
Der Umweg über die **Local API (JSON + UDP-Port)** war sperrig und gelegentlich instabil.

Daraufhin testete ich mehrere **Home-Assistant-Integrationen**, meist über **HACS** installiert.  
Dabei stellte sich schnell heraus, dass mein **Venus C V3.0** teilweise **abweichende Registeradressen** gegenüber älteren Marstek-Modellen nutzt.

Es folgte eine Suche durch diverse:

- Home-Assistant-Integrationen  
- ModbusTCP–MQTT-Bridges  

Alle funktionierten mehr oder weniger gut.

Beim Versuch, selbst Anpassungen vorzunehmen oder einen eigenen Docker-Container zu bauen, zeigte sich ein recht chaotisches Bild:

- nicht mehr gepflegte Python-Pakete  
- diverse Forks  
- Konfiguration teils in **YAML**, teils in **CSV**  
- hart gepinnte Versionen in Dockerfiles  
- Installation per `pip` als **root**, statt sauber mit `venv`  

Final bin ich – unterstützt durch ChatGPT – bei **Node-RED** gelandet.  
Nach einigen Iterationen hatte ich stabile Flows für den **Marstek Venus 3.0**.  
Im Anschluss wurde auch der **Huawei Sun per Modbus TCP** angebunden und der alte Docker-Container vollständig ersetzt.

---

## 🛠 Anpassungen an den Flows / Lessons Learned

- Alle verfügbaren Register-Dokumentationen befinden sich im Ordner **`Misc`**  
- Modbus TCP sollte möglichst **nicht zu häufig** abgefragt werden  
- Idealerweise werden mehrere Register gemeinsam gelesen und anschließend zerlegt  

⚠️ **Wichtig:**  
Zu häufiges Polling (z. B. alle 5 Sekunden) führt insbesondere beim Huawei-Adapter zuverlässig zu Lesefehlern.

### Design-Entscheidungen

- ❌ kein Lesen großer Registerblöcke  
- ✅ gezieltes Lesen einzelner Register  
- ⏱ Polling-Intervall: **60 Sekunden**  

Für typische Anwendungen ist das vollkommen ausreichend:

- InfluxDB  
- Grafana  
- Home-Assistant-Dashboards  

### Register-Stabilität

- **Huawei Sun:** Register gelten als relativ stabil  
- **Marstek Venus:** Register können je nach Firmware variieren  

Empfehlung:  
Neue Register **immer einzeln testen** und mittels **Debug-Nodes** auf Plausibilität prüfen.  
Ein Abgleich mit App-Werten (PV-Leistung, SOC etc.) ist dringend zu empfehlen.

Beispiel Marstek:  
Die geladene Kapazität wird intern aus **SOC und einer festen Maximal-Kapazität** berechnet.  
Diese Logik muss bei Bedarf in Node-RED nachgebildet werden.

🔒 Es wird ausschließlich **lesender Modbus-Zugriff** verwendet.  
Beim **Schreiben von Registern** (z. B. Einspeiseleistung steuern) ist äußerste Vorsicht geboten.

---

## ⚡ Einspeiseleistung & Steuerung

Wer den **Marstek ohne gekoppelten Stromzähler / IR-Lesekopf** betreiben möchte, sollte sich das **Unimeter-Projekt** ansehen:

https://github.com/sdeigm/uni-meter

Die Steuerung meines Marstek erfolgt über die **Local API**, gekoppelt an ein **MQTT Topic**.

---

## 📡 MQTT Topics

### Huawei Sun

```
huawei/pv-power                 - PV-Leistung
huawei/grid-power               - Leistung Stromzähler (- = Netzbezug / + = Einspeisung)
huawei/battery-1-soc            - SOC Batterie #1
huawei/battery-1-temp           - Temperatur Batterie #1
huawei/inverter-temp            - Interne Inverter-Temperatur
huawei/battery-1-power          - Lade- / Entladeleistung Batterie
```

### Marstek Venus

```
marstek-modbus/battery/soc                - SOC Batterie
marstek-modbus/battery/capacity           - Maximalkapazität (Fixwert)
marstek-modbus/system/temp                - Systemtemperatur
marstek-modbus/grid_power                 - Leistung (- = Netzbezug / + = Einspeisung)
marstek-modbus/battery/capacity_charged   - Geladene Batteriekapazität
```

---

## 🔗 Referenzen

- https://debacher.de/wiki/Sun2000_Modbus_Register  
- https://github.com/ViperRNMC/marstek_venus_modbus  
- https://www.forwardme.de/2025/04/03/marstek-venus-speicher-emulierter-shelly-synology-diskstation-ueberschussladen/  

---

## ⚠️ Hinweise

- Huawei-Register variieren je nach Firmware & Modell  
- Polling **unter 30 Sekunden** kann zu Verbindungsabbrüchen führen  
- Nutzung erfolgt **auf eigenes Risiko**  

---

## 🤝 Contributions

Pull Requests sind willkommen für:

- neue Register  
- andere Huawei-Modelle  
- Verbesserungen der Flow-Struktur  
- Dokumentations-Updates  

---

## 📜 Lizenz

**MIT License**

Keine Garantie. Keine Haftung.  
Verwende es, ändere es, veröffentliche es weiter.
