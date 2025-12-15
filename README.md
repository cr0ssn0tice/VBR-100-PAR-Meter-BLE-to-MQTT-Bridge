# VBR-100-PAR-Meter-BLE-to-MQTT-Bridge
This project connects a VBR-100 PAR Meter (Plant Lighting Meter) to an ESP32 via Bluetooth Low Energy (BLE). It parses the raw data packets to calculate PPFD (Photosynthetic Photon Flux Density) and RGB spectrum analysis, then publishes the results as a JSON object to an MQTT broker over WiFi.

This allows for seamless integration into Smart Home systems like Home Assistant, ioBroker, or Node-RED.

🚀 FeaturesBLE Client: The ESP32 acts as a central device, connecting directly to the VBR-100.
  Data Parsing: Decodes the proprietary byte stream into human-readable values: 
  PPFD ($\mu mol/m^2/s$)Spectrum Analysis: Red, Green, Blue components (absolute values and percentages).
  MQTT Integration: Sends data as a clean JSON string.
  Rate Limiting: Sends updates every 15 seconds (configurable) to prevent MQTT flooding, while maintaining a stable BLE connection.
  Auto-Reconnect: Automatically handles WiFi, MQTT, and BLE connection drops.
  
🛠 Hardware Required
  ESP32 Development Board (e.g., ESP32-WROOM-32).
  VBR-100 PAR Meter (Bluetooth enabled).

📦 DependenciesThis project uses the Arduino IDE. You need to install the following library via the Library Manager:  
  PubSubClient by Nick O'Leary (for MQTT).
  (The BLEDevice library is included in the standard ESP32 board package).
  
⚙️ ConfigurationOpen the .ino file and update the following settings:
  C++
  // --- WIFI & MQTT SETTINGS ---
const char* ssid        = "YOUR_WIFI_SSID";
const char* password    = "YOUR_WIFI_PASSWORD";
const char* mqtt_server = "192.168.1.XX"; // Your MQTT Broker IP
const int   mqtt_port   = 1883;

// --- VBR-100 SETTINGS ---
// Replace this with your specific VBR-100 MAC Address.
// You can find this using a BLE Scanner app on your phone.
static BLEAddress vbr100Address("MACADRESS");

🧠 Protocol Reverse Engineering & Logic
The VBR-100 uses a proprietary GATT service to transmit data. Below is the derivation of the data parsing logic implemented in this project.

UUIDs
Service UUID: 00010203-0405-0607-0809-0a0b0c0dbc10
Notify Characteristic: 00010203-0405-0607-0809-0a0b0c0d2c12

Data Packet Structure (20 Bytes)
The device sends a 20-byte payload via notification. The relevant data is Big Endian.
Byte   Index           Description Interpretation
0-7    Header/Padding  Ignored  
8-9    Integer Part    Whole number part of PPFD
10-11  Fractional Part Thousandths (0..999)
12-13  Red Raw         Relative weight for Red spectrum
14-15  Green Raw       Relative weight for Green spectrum
16-17  Blue Raw        Relative weight for Blue spectrum
18-19  Footer/Check    Ignored

Calculation Logic
The raw hex data is converted into physical values using the following formulas:

1.  **Total PPFD Calculation** (combining integer and fractional parts):
    $$PPFD_{total} = IntegerPart + \frac{FractionalPart}{1000}$$

2.  **Spectrum Weighting** (sum of raw RGB values):
    $$Sum_{RGB} = R_{raw} + G_{raw} + B_{raw}$$

3.  **Absolute Component Calculation** (distributing PPFD based on color weight):
    $$Red_{\mu mol} = PPFD_{total} \times \frac{R_{raw}}{Sum_{RGB}}$$

4.  **Percentage Calculation**:
    $$Red_{\%} = \frac{R_{raw}}{Sum_{RGB}} \times 100$$
    

📡 MQTT Output
The device publishes to the topic: grow/vbr100/state
{
  "ppfd": 125.500,
  "r": 73.919,
  "g": 38.780,
  "b": 12.801,
  "r_pct": 58.9,
  "g_pct": 30.9,
  "b_pct": 10.2
}

ppfd: Total Photosynthetic Photon Flux Density.
r, g, b: The contribution of Red, Green, and Blue light to the total PPFD.
*_pct: The percentage distribution of the spectrum.

📜 License
This project is open-source. Feel free to modify and distribute.
