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
4. **Link:** `https://platform.favoriot.com/tutorial/v2/?python#edge-gateway`

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
   #include <MQTT.h>
   #include <WiFiClientSecure.h>
   #include <Adafruit_APDS9960.h>
   #include <Adafruit_BME280.h>
   #include <Adafruit_MPU6050.h>
   #include "FavoriotCA.h"
   
   // Initialize sensors
   Adafruit_APDS9960 apds;
   Adafruit_BME280 bme;
   Adafruit_MPU6050 mpu;
   sensors_event_t a, g, temp;
   
   // Wi-Fi and MQTT details
   const char ssid[] = "YOUR_WIFI_SSID";
   const char password[] = "YOUR_WIFI_PASSWORD";
   const char GatewayDeveloperID[] = "YOUR_GATEWAY_DEVELOPER_ID";
   const char GatewayAccessToken[] = "YOUR_GATEWAY_ACCESS_TOKEN";
   const char publishTopic[] = "YOUR_MQTT_PUBLISH_TOPIC";
   const char SubscribeTopic[] = "YOUR_MQTT_SUBSCRIBE_TOPIC";
   
   WiFiClientSecure wifiClient;
   MQTTClient mqttClient(4096);  // Increase buffer size for larger payload
   
   unsigned long lastProximityMillis = 0;
   unsigned long lastEnvironmentMillis = 0;
   unsigned long lastMotionMillis = 0;
   
   void connectToWiFi() {
     Serial.print("Connecting to Wi-Fi ");
     WiFi.begin(ssid, password);
     while (WiFi.status() != WL_CONNECTED) {
       Serial.print(".");
       delay(500);
     }
     Serial.println(" connected!");
   }
   
   void messageReceived(String &topic, String &payload) {
     Serial.println("Incoming message on topic " + topic + ": " + payload);
   }
   
   void connectToMQTT() {
     while (!mqttClient.connected()) {
       Serial.print("Connecting to MQTT...");
       if (mqttClient.connect("ESP32Client", GatewayAccessToken, GatewayAccessToken)) {
         Serial.println(" connected");
         mqttClient.subscribe(String(SubscribeTopic));
         Serial.println("Subscribed to status topic: " + String(SubscribeTopic));
       } else {
         Serial.print(" failed, status code: ");
         Serial.println(mqttClient.lastError());
         delay(2000);
       }
     }
   }
   
   void setup() {
     Serial.begin(115200);
   
     // Initialize sensors
     if (!apds.begin()) {
       Serial.println("Failed to find APDS9960 chip");
     }
     apds.enableProximity(true);
   
     if (!bme.begin()) {
       Serial.println("Failed to find BME280 chip");
     }
   
     if (!mpu.begin()) {
       Serial.println("Failed to find MPU6050 chip");
     }
   
     // Connect to Wi-Fi and MQTT
     connectToWiFi();
     wifiClient.setCACert(rootCACertificate);
     mqttClient.begin("mqtt.favoriot.com", 8883, wifiClient);
     mqttClient.onMessage(messageReceived);
     connectToMQTT();
   }
   
   void loop() {
     if (WiFi.status() != WL_CONNECTED) {
       connectToWiFi();
     }
   
     if (!mqttClient.connected()) {
       connectToMQTT();
     }
   
     mqttClient.loop();
   
     unsigned long currentMillis = millis();
   
     // Proximity sensor data every 5 seconds
     if (currentMillis - lastProximityMillis >= 5000) {
       lastProximityMillis = currentMillis;
   
       int proximity = apds.readProximity();
       String payload = "{\"uid\":\"" + String("apds1234") + "\",\"data\":{\"proximity\":\"" + String(proximity) + "\"}}";
   
       Serial.println("\nSending Proximity data...");
       Serial.println("Data to Publish: " + payload);
   
       if (mqttClient.publish(String(publishTopic), payload)) {
         Serial.println("Proximity data sent successfully");
       } else {
         Serial.println("Failed to send Proximity data");
       }
     }
   
     // Environmental sensor data every 5 seconds
     if (currentMillis - lastEnvironmentMillis >= 5000) {
       lastEnvironmentMillis = currentMillis;
   
       float humidity = bme.readHumidity();
       float temperature = bme.readTemperature();
       float pressure = bme.readPressure() / 100.0;
       float altitude = bme.readAltitude(1013.25);
   
       String payload = "{\"uid\":\"" + String("bme1234") + "\",\"data\":{";
       payload += "\"humidity\":\"" + String(humidity) + "\",";
       payload += "\"temperature\":\"" + String(temperature) + "\",";
       payload += "\"pressure\":\"" + String(pressure) + "\",";
       payload += "\"altitude\":\"" + String(altitude) + "\"}}";
   
       Serial.println("\nSending Environmental data...");
       Serial.println("Data to Publish: " + payload);
   
       if (mqttClient.publish(String(publishTopic), payload)) {
         Serial.println("Environmental data sent successfully");
       } else {
         Serial.println("Failed to send Environmental data");
       }
     }
   
     // Motion sensor data every 5 seconds
     if (currentMillis - lastMotionMillis >= 5000) {
       lastMotionMillis = currentMillis;
   
       mpu.getEvent(&a, &g, &temp);
   
       String payload = "{\"uid\":\"" + String("mpu1234") + "\",\"data\":{";
       payload += "\"acceleration_x\":\"" + String(a.acceleration.x) + "\",";
       payload += "\"acceleration_y\":\"" + String(a.acceleration.y) + "\",";
       payload += "\"acceleration_z\":\"" + String(a.acceleration.z) + "\",";
       payload += "\"gyro_x\":\"" + String(g.gyro.x) + "\",";
       payload += "\"gyro_y\":\"" + String(g.gyro.y) + "\",";
       payload += "\"gyro_z\":\"" + String(g.gyro.z) + "\"}}";
   
       Serial.println("\nSending Motion data...");
       Serial.println("Data to Publish: " + payload);
   
       if (mqttClient.publish(String(publishTopic), payload)) {
         Serial.println("Motion data sent successfully");
       } else {
         Serial.println("Failed to send Motion data");
       }
     }
   
     delay(100);  // Small delay for loop stability
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


<p align="center"><img src="https://github.com/user-attachments/assets/86964cc6-dd9d-47f8-a5bd-cb974b9153a3" width="990"></a></p>


<p align="center"><img src="https://github.com/user-attachments/assets/97d8c4c7-f336-4623-9462-0da928444ea5" width="990"></a></p>

   

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
