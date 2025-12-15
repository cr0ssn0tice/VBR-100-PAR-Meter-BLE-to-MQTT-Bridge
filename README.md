# VBR-100-PAR-Meter-BLE-to-MQTT-Bridge
This project connects a VBR-100 PAR Meter (Plant Lighting Meter) to an ESP32 via Bluetooth Low Energy (BLE). It parses the raw data packets to calculate PPFD (Photosynthetic Photon Flux Density) and RGB spectrum analysis, then publishes the results as a JSON object to an MQTT broker over WiFi.
