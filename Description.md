
### 1. Full Connection Table

| Component | ESP32 Pin | Connection Detail |
| :--- | :--- | :--- |
| **MLX90614 (SDA)** | **Pin 21** | Connect Data pin of Temp sensor. |
| **MLX90614 (SCL)** | **Pin 22** | Connect Clock pin of Temp sensor. |
| **MAX30102 (SDA)** | **Pin 21** | Share Data pin with Temp sensor. |
| **MAX30102 (SCL)** | **Pin 22** | Share Clock pin with Temp sensor. |
| **MAX4466 Mic (Out)** | **Pin 34** | Connect Mic output to Analog Pin 34. |
| **Danger Button** | **Pin 4** | Connect between Pin 4 and 3.3V. |
| **Water Probes** | **Pin 5** | One to Pin 5, one to 3.3V. |
| **Fire Loop** | **Pin 23** | Wire loop from 3.3V to Pin 23. |
| **VCC / GND** | **3.3V / GND** | All sensors share power and ground. |

---

### 2. Short Description
**The Autonomous Child Safety Hub** is an advanced IoT protection system. It tracks environment hazards (Fire/Water), biological health (**MLX90614** Temp & **MAX30102** Pulse), and uses an automated voice processing system to convert and speak Bangla phrases in English instantly.

---

### 3. How It Works
* **Health Tracking:** The **MLX90614** reads temperature via infrared, and the **MAX30102** monitors blood pulse.
* **Fire Logic:** Pin 23 is a loop. If fire burns the wire, the 3.3V signal is lost (LOW), triggering **"BABY ON FIRE."**
* **Water Logic:** Probes on Pin 5 detect moisture. If water bridges the gap, the signal goes HIGH, triggering **"UNDER WATER."**
* **Auto-Translation & Play:** The website uses `r.continuous = true`. It listens to your phone/PC microphone. When you speak Bangla, it sends the text to Google, gets the English translation, and uses `window.speechSynthesis` to **speak the English translation out loud** automatically.
* **Fixed IP:** Access the live dashboard at **192.168.0.9** over the **CHILD SAFETY** WiFi.

---

### 4. Project Output Details
* **WiFi Name:** `CHILD SAFETY`
* **Password:** `234555444`
* **Access URL:** `http://192.168.0.9`
