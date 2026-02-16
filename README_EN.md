# Huawei Sun & Marstek Venus Modbus TCP → MQTT Bridge (Node-RED Flows)

This repository contains **pure Node-RED flow exports (JSON)** for connecting **Huawei Sun** and **Marstek Venus** inverters and battery systems via **Modbus TCP**.

The flows are explicitly aimed at **experienced Node-RED users** who:

- operate their runtime environment themselves  
- are already familiar with MQTT & Modbus TCP  
- do **not** require Docker or Python wrappers  

---

## 🎯 Goals

- stable Modbus TCP polling (**≥ 60 s interval**)  
- clean and well-structured MQTT topics  
- transparent register handling  
- full control inside Node-RED flows  

### ❌ Explicitly NOT used

- `modbus2mqtt`  
- YAML / CSV configuration parsers  
- pinned Python versions  
- hidden defaults  

---

## ✅ Requirements

- running **Node-RED** instance  

### Required Nodes

- `node-red-contrib-modbus`  
- `node-red-node-mqtt`  

### Hardware / Systems

- MQTT broker (e.g. Mosquitto)  
- **Huawei Sun** inverter with enabled Modbus TCP adapter  
- **or**  
- **Marstek Venus** battery storage with recent firmware (**> V1.47**)  

---

## 📘 Background

The motivation for creating these Node-RED flows was the general inconvenience of running **Modbus TCP** via Docker containers using Python-based *ModbusTCP → MQTT* bridges.

This was initially the case with my **Huawei Sun inverter with Luna battery and Modbus TCP adapter**.  
While the Docker container reliably read data, it was **highly static** regarding available registers. Any change required rebuilding the Docker image.

After purchasing a **Marstek Venus C V3.0**, I started integrating it into my home automation setup.  
Using the **Local API (JSON over UDP)** turned out to be clumsy and occasionally unstable.

I then tested several **Home Assistant integrations**, typically installed via **HACS**.  
It quickly became apparent that the **Venus C V3.0** uses **different Modbus register addresses** compared to older Marstek models.

This resulted in testing multiple:

- Home Assistant integrations  
- Modbus TCP → MQTT bridges  

All of them worked only partially or inconsistently.

When attempting to modify register mappings or build a custom Docker container, the overall ecosystem appeared rather chaotic:

- unmaintained Python packages  
- multiple forks  
- configuration split between **YAML** and **CSV**  
- Dockerfiles pinning specific dependency versions  
- `pip` installs as **root** instead of using virtual environments  

Eventually, with some help from ChatGPT, I settled on **Node-RED**.  
After several iterations, I ended up with stable flows for the **Marstek Venus 3.0**.  
Encouraged by this success, I also connected the **Huawei Sun inverter via Modbus TCP** and removed the old Docker container entirely.

---

## 🛠 Flow Design / Lessons Learned

- All available register documentation is collected in the **`Misc`** directory  
- Modbus TCP devices are often **very sensitive to polling frequency**  
- Ideally, multiple registers should be read in a single request and processed afterwards  

⚠️ **Important:**  
Polling too frequently (e.g. every 5 seconds) will almost certainly cause read errors, especially with Huawei adapters.

### Design Decisions

- ❌ no large register block reads  
- ✅ targeted reads of individual registers  
- ⏱ polling interval: **60 seconds**  

This is more than sufficient for typical use cases such as:

- InfluxDB data collection  
- Grafana visualization  
- Home Assistant dashboards  

### Register Stability

- **Huawei Sun:** registers are relatively stable in meaning and format  
- **Marstek Venus:** registers may vary depending on firmware version  

Recommendation:  
Always test new registers **one at a time** and verify plausibility using **Debug nodes**.  
Cross-check values with the vendor app (PV power, SOC, etc.).

Marstek example:  
Charged battery capacity is internally derived from **SOC and a fixed maximum capacity value**.  
This logic may need to be replicated manually in Node-RED.

🔒 Only **read-only Modbus access** is used.  
If you plan to **write registers** (e.g. control feed-in power), proceed with extreme caution and start with very small values.

---

## ⚡ Feed-in Power Control

If you want to control the **Marstek Venus without a coupled power meter / IR reader**, take a look at the **Unimeter project**:

https://github.com/sdeigm/uni-meter

I personally control my Marstek via the **Local API**, coupled with an **MQTT topic**.

---

## 📡 MQTT Topics

### Huawei Sun

```
huawei/pv-power                 - PV power
huawei/grid-power               - Grid meter power (- = import / + = export)
huawei/battery-1-soc            - Battery #1 SOC
huawei/battery-1-temp           - Battery #1 temperature
huawei/inverter-temp            - Internal inverter temperature
huawei/battery-1-power          - Battery charge / discharge power
```

### Marstek Venus

```
marstek-modbus/battery/soc                - Battery SOC
marstek-modbus/battery/capacity           - Battery maximum capacity (fixed value)
marstek-modbus/system/temp                - System temperature
marstek-modbus/grid_power                 - Power (- = import / + = export)
marstek-modbus/battery/capacity_charged   - Charged battery capacity
```

---

## 🔗 References

- https://debacher.de/wiki/Sun2000_Modbus_Register  
- https://github.com/ViperRNMC/marstek_venus_modbus  
- https://www.forwardme.de/2025/04/03/marstek-venus-speicher-emulierter-shelly-synology-diskstation-ueberschussladen/  

---

## ⚠️ Notes

- Huawei registers vary by firmware and model  
- Polling **below 30 seconds** may lead to connection drops  
- Use at your own risk  

---

## 🤝 Contributions

Pull requests are welcome for:

- additional registers  
- other Huawei models  
- flow structure improvements  
- documentation updates  

---

## 📜 License

**MIT License**

No warranty. No liability.  
Use it, modify it, redistribute it.
