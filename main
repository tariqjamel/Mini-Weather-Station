#include <WiFi.h>
#include <HTTPClient.h>
#include <Wire.h>
#include <Adafruit_Sensor.h>
#include <Adafruit_BMP085.h>

// Wi-Fi credentials
const char* ssid = "WIFI_NAME";
const char* password = "WIFI_PASSWORD";

// InfluxDB settings
const char* influxDBURL = "http://LAPTOP_IP:8086/api/v2/write?org=dot&bucket=weather-data&precision=s";
const char* influxDBToken = "iNFLUX_DB_TOKEN";

// BMP180 sensor object
Adafruit_BMP085 bmp;

void connectToWiFi() {
  Serial.print("Connecting to WiFi: ");
  Serial.println(ssid);
  WiFi.begin(ssid, password);

  unsigned long startTime = millis();
  while (WiFi.status() != WL_CONNECTED && millis() - startTime < 15000) {
    delay(500);
    Serial.print(".");
  }

  if (WiFi.status() == WL_CONNECTED) {
    Serial.println("\n✅ WiFi connected!");
    Serial.print("IP address: ");
    Serial.println(WiFi.localIP());
  } else {
    Serial.println("\n❌ WiFi connection failed.");
  }
}

void setup() {
  Serial.begin(115200);
  delay(1000);
  Wire.begin(21, 22); // SDA, SCL for ESP32
  //Connect SDA to pin 21 and SCL to pin 22
  connectToWiFi();

  if (!bmp.begin()) {
    Serial.println("❌ Could not find BMP180 sensor. Check wiring!");
    while (1);
  } else {
    Serial.println("✅ BMP180 sensor initialized.");
  }
}

void loop() {
  if (WiFi.status() != WL_CONNECTED) {
    Serial.println("WiFi disconnected, reconnecting...");
    connectToWiFi();
  }

  float temperature = bmp.readTemperature();             // °C
  float pressure = bmp.readPressure() / 100.0F;          // hPa
  float altitude = bmp.readAltitude(1013.25);            // meters (standard sea-level pressure)

  Serial.println("\n--- Sensor Readings ---");
  Serial.print("Temperature: "); Serial.print(temperature); Serial.println(" °C");
  Serial.print("Pressure: "); Serial.print(pressure); Serial.println(" hPa");
  Serial.print("Altitude: "); Serial.print(altitude); Serial.println(" m");

  // InfluxDB Line Protocol format
  String payload = "weather,device=esp32 ";
  payload += "temperature=" + String(temperature, 2) + ",";
  payload += "pressure=" + String(pressure, 2) + ",";
  payload += "altitude=" + String(altitude, 2);

  HTTPClient http;
  http.begin(influxDBURL);
  http.addHeader("Authorization", "Token " + String(influxDBToken));
  http.addHeader("Content-Type", "text/plain");

  int httpResponseCode = http.POST(payload);

  Serial.print("POST Status: ");
  Serial.println(httpResponseCode);
  if (httpResponseCode == 204) {
    Serial.println("✅ Data sent: " + payload);
  } else {
    Serial.println("❌ Failed to send data");
    Serial.println("Response: " + http.getString());
  }

  http.end();
  delay(10000); // Send every 10 seconds
}

