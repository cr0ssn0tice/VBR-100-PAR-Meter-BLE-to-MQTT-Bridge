### VBR-100-PAR-Meter-BLE-to-MQTT-Bridge
This project connects a VBR-100 PAR Meter (Plant Lighting Meter) to an ESP32 via Bluetooth Low Energy (BLE). It parses the raw data packets to calculate PPFD (Photosynthetic Photon Flux Density) and RGB spectrum analysis, then publishes the results as a JSON object to an MQTT broker over WiFi.

This allows for seamless integration into Smart Home systems like Home Assistant, ioBroker, or Node-RED.

### 🚀 FeaturesBLE Client: The ESP32 acts as a central device, connecting directly to the VBR-100.
  Data Parsing: Decodes the proprietary byte stream into human-readable values: 
  PPFD ($\mu mol/m^2/s$)Spectrum Analysis: Red, Green, Blue components (absolute values and percentages).
  MQTT Integration: Sends data as a clean JSON string.
  Rate Limiting: Sends updates every 15 seconds (configurable) to prevent MQTT flooding, while maintaining a stable BLE connection.
  Auto-Reconnect: Automatically handles WiFi, MQTT, and BLE connection drops.
  
### 🛠 Hardware Required
  ESP32 Development Board (e.g., ESP32-WROOM-32).
  VBR-100 PAR Meter (Bluetooth enabled).

### 📦 DependenciesThis project uses the Arduino IDE. You need to install the following library via the Library Manager:  
  PubSubClient by Nick O'Leary (for MQTT).
  (The BLEDevice library is included in the standard ESP32 board package).
  
### ⚙️ ConfigurationOpen the .ino file and update the following settings:
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

### 🧠 Protocol Reverse Engineering & Logic
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




### Calculation Logic
The raw hex data is parsed and converted using the following logic:

// 1. Parse Raw Bytes (Big Endian)
// Bytes 8-9: Integer part of PPFD
// Bytes 10-11: Fractional part of PPFD (thousandths)
IntegerPart    = (Data[8]  << 8) | Data[9]
FractionalPart = (Data[10] << 8) | Data[11]

// Raw RGB weights
Red_Raw   = (Data[12] << 8) | Data[13]
Green_Raw = (Data[14] << 8) | Data[15]
Blue_Raw  = (Data[16] << 8) | Data[17]

// 2. Calculate Total PPFD
// Combine integer and fractional parts (e.g., 125 + 0.500 = 125.500)
PPFD_Total = IntegerPart + (FractionalPart / 1000.0)

// 3. Calculate Spectrum Distribution
// Sum of all color weights
RGB_Sum = Red_Raw + Green_Raw + Blue_Raw

// Calculate absolute values (umol/m2/s)
Red_Value   = PPFD_Total * (Red_Raw   / RGB_Sum)
Green_Value = PPFD_Total * (Green_Raw / RGB_Sum)
Blue_Value  = PPFD_Total * (Blue_Raw  / RGB_Sum)

// Calculate Percentages (%)
Red_Percent   = (Red_Raw   / RGB_Sum) * 100.0
Green_Percent = (Green_Raw / RGB_Sum) * 100.0
Blue_Percent  = (Blue_Raw  / RGB_Sum) * 100.0


### 📡 MQTT Output
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

### 📜 License
This project is open-source. Feel free to modify and distribute.
