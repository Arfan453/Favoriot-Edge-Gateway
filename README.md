# Sending Hibiscus Sense ESP32 Data to Favoriot Edge Gateway

This guide provides step-by-step instructions for configuring a Hibiscus Sense ESP32 to send sensor data to the Favoriot Edge Gateway.

---

## Prerequisites

1. **Hibiscus Sense ESP32 Board**:
   - Ensure you have a functional Hibiscus Sense ESP32 board.
2. **Favoriot Edge Gateway**:
   - Access to Favoriot Edge Gateway with the necessary configurations.
3. **Arduino IDE Installed**:
   - Install Arduino IDE. [Download Here](https://www.arduino.cc/en/software)
4. **Favoriot Account**:
   - Ensure you have an API key and access to Favoriot services.

---

## Step 1: Set Up Favoriot Edge Gateway

1. Log in to your Favoriot account.
2. Navigate to **Edge Gateway** settings.
3. Note the Gateway URL, Port, and MQTT Topic details.

---

## Step 2: Install Required Libraries in Arduino IDE

1. Open the Arduino IDE.
2. Install the following libraries via the Library Manager:
   - **WiFi**: For ESP32 Wi-Fi connectivity.
   - **PubSubClient**: For MQTT communication.
   - **Adafruit Sensor Libraries** (if using sensors like BME280).

---

## Step 3: Configure Arduino Sketch

1. Open a new sketch in Arduino IDE.
2. Add the following code:

   ```cpp
   #include <WiFi.h>
   #include <PubSubClient.h>

   // Wi-Fi credentials
   const char* ssid = "YOUR_WIFI_SSID";
   const char* password = "YOUR_WIFI_PASSWORD";

   // Favoriot Edge Gateway configurations
   const char* mqtt_server = "YOUR_GATEWAY_URL";
   const int mqtt_port = YOUR_GATEWAY_PORT;
   const char* mqtt_topic = "YOUR_GATEWAY_TOPIC";

   WiFiClient espClient;
   PubSubClient client(espClient);

   void setup() {
       Serial.begin(115200);

       // Connect to Wi-Fi
       WiFi.begin(ssid, password);
       while (WiFi.status() != WL_CONNECTED) {
           delay(500);
           Serial.print(".");
       }
       Serial.println("WiFi connected");

       // Set MQTT server
       client.setServer(mqtt_server, mqtt_port);

       // Connect to MQTT broker
       while (!client.connected()) {
           if (client.connect("ESP32Client")) {
               Serial.println("MQTT connected");
           } else {
               Serial.print("Failed, rc=");
               Serial.print(client.state());
               delay(2000);
           }
       }
   }

   void loop() {
       if (!client.connected()) {
           // Reconnect if disconnected
           while (!client.connected()) {
               if (client.connect("ESP32Client")) {
                   Serial.println("Reconnected to MQTT");
               } else {
                   delay(2000);
               }
           }
       }

       // Prepare sensor data (example temperature and humidity)
       float temperature = 25.0; // Replace with actual sensor data
       float humidity = 60.0;    // Replace with actual sensor data

       String payload = "{";
       payload += "\"temperature\": " + String(temperature) + ", ";
       payload += "\"humidity\": " + String(humidity);
       payload += "}";

       // Publish data to Favoriot Edge Gateway
       client.publish(mqtt_topic, payload.c_str());

       delay(60000); // Send data every 60 seconds
   }
   ```

3. Replace the placeholders (`YOUR_WIFI_SSID`, `YOUR_GATEWAY_URL`, etc.) with your actual configurations.

---

## Step 4: Upload Code to ESP32

1. Connect the Hibiscus Sense ESP32 to your computer via USB.
2. Select the appropriate **Board** and **Port** from the Arduino IDE.
3. Click **Upload** to upload the sketch to the board.
4. Open the **Serial Monitor** to verify the connection and data transmission.

---

## Step 5: Verify Data in Favoriot Platform

1. Log in to your Favoriot account.
2. Navigate to the **Edge Gateway Logs**.
3. Verify that data from the Hibiscus Sense ESP32 is being received and logged.

---

## Troubleshooting

- **Wi-Fi Connection Issues**:
  - Verify the SSID and password.
- **MQTT Connection Issues**:
  - Check the Gateway URL, port, and topic.
- **No Data in Favoriot**:
  - Verify the MQTT payload and ensure the Favoriot Edge Gateway is configured correctly.

---

## Resources

- [Favoriot API Documentation](https://docs.favoriot.com/)
- [ESP32 Arduino Documentation](https://docs.espressif.com/projects/arduino-esp32/en/latest/)

---

## License

This project is licensed under the MIT License. See the LICENSE file for details.
