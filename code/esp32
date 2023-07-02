#include <Wire.h>
#include <Adafruit_GFX.h>
#include <LiquidCrystal_I2C.h>
#include "ThingSpeak.h"
#include <WiFi.h>
#include <OneWire.h>

// Set the ThingSpeak channel and API key information
unsigned long myChannelNumber = ;
const char* myWriteAPIKey = "";
float TempVal = 0.0;

// Set the WiFi network credentials
const char* ssid = ""; // your wifi SSID name
const char* password = ""; // wifi password

// Set the ThingSpeak server address
const char* server = "api.thingspeak.com";
WiFiClient client;
int wait_between_uploads = 10000; // 10 seconds

LiquidCrystal_I2C lcd(0x27, 16, 2);  // set the LCD address to 0x27 for a 16 chars and 2 line display

unsigned long lastDisplayTime = 0;
unsigned long displayInterval = 500;

float calibration_value = 22.34; //21.64
int phval = 0;
unsigned long int avgval;
int buffer_arr[10], temp;
float ph_act;
float EC_val;
float TDS_val;
const byte tds_sensor = A1; // TDS Sensor Pin

namespace device {
  float aref = 4.3;
}

namespace sensor {
  float ec = 0;
  unsigned int tds = 0;
  float waterTemp = 31.4; // Assume water temperature of 28 degrees Celsius
  float ecCalibration = 1.0;
  float temperature = 0; // Placeholder for temperature reading
}
OneWire oneWire(tds_sensor);

const int phSensorRelayPin = D2; // Digital pin connected to the relay controlling pH sensor power
const int ecSensorRelayPin = D3; // Digital pin connected to the relay controlling EC sensor power

void setup() {
  Wire.begin();
  Serial.begin(115200);
  lcd.init();
  lcd.backlight();

  pinMode(phSensorRelayPin, OUTPUT); // Set the relay pin as output
  pinMode(ecSensorRelayPin, OUTPUT); // Set the relay pin as output

  // Disconnect any previous WiFi connection
  WiFi.disconnect();
  delay(10);

  // Connect to the WiFi network
  WiFi.begin(ssid, password);
  Serial.print("Connecting to ");
  Serial.println(ssid);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("");
  Serial.print("ESP32 connected to WiFi: ");
  Serial.println(ssid);
  Serial.println();

  // Initialize the ThingSpeak library with the WiFi client
  ThingSpeak.begin(client);
}

void loop() {

  
  // Power on the pH sensor and wait for stabilization
  digitalWrite(phSensorRelayPin, LOW);

  delay(5000); // Adjust the delay as needed for sensor stabilization

  // Take pH measurement
  readpH();

  
  // Power off the pH sensor
  digitalWrite(phSensorRelayPin, HIGH);

  // Wait for a short duration before proceeding to the EC measurement
  delay(100);

  // Power on the EC sensor and wait for stabilization
  digitalWrite(ecSensorRelayPin, LOW);
  delay(5000); // Adjust the delay as needed for sensor stabilization
  // Take EC measurement
  readTdsQuick();

  // Power off the EC sensor
  digitalWrite(ecSensorRelayPin, HIGH);

  // Send the data to ThingSpeak
  ThingSpeak.writeFields(myChannelNumber, myWriteAPIKey);
  delay(1000);
  
  // Set the values to be sent to ThingSpeak
  ThingSpeak.setField(1, ph_act);
  ThingSpeak.setField(2, TDS_val);
  ThingSpeak.setField(3, EC_val);
  // Print a message to the serial monitor indicating that the data has been uploaded
  Serial.println("Uploaded to ThingSpeak server.");

  // Disconnect the WiFi client
  client.stop();

  // Wait for the specified amount of time before uploading the next set of data
  // ThingSpeak requires a minimum 15 seconds delay between updates on a free account
  Serial.println("Waiting to upload next reading...");
  Serial.println();
  delay(wait_between_uploads);

  lcd.clear();
}

void readpH() {
  for (int i = 0; i < 10; i++) {
    buffer_arr[i] = analogRead(A0);
    delay(30);
  }

  for (int i = 0; i < 9; i++) {
    for (int j = i + 1; j < 10; j++) {
      if (buffer_arr[i] > buffer_arr[j]) {
        temp = buffer_arr[i];
        buffer_arr[i] = buffer_arr[j];
        buffer_arr[j] = temp;
      }
    }
  }

  avgval = 0;
  for (int i = 2; i < 8; i++) {
    avgval += buffer_arr[i];
  }
  float volt = (float)avgval * 3.3 / 4096 / 6;
  ph_act = -5.70 * volt + calibration_value;
  Serial.print("pH Val: ");
  Serial.println(ph_act);
  lcd.setCursor(0, 0);
  lcd.print("PH: ");
  lcd.setCursor(0, 1);
  lcd.print(ph_act);
}

void readTdsQuick() {
  float rawEc = analogRead(tds_sensor) * device::aref / 4096.0;
  float temperatureCoefficient = 1.0 + 0.02 * (sensor::waterTemp - 25.0);
  sensor::ec = (rawEc / temperatureCoefficient) * sensor::ecCalibration;
  sensor::tds = (133.42 * pow(sensor::ec, 3) - 255.86 * sensor::ec * sensor::ec + 857.39 * sensor::ec) * 0.5;
  EC_val = sensor::ec * 1000.0;
  TDS_val = sensor::tds;
  Serial.print(F("TDS:")); Serial.print(TDS_val); Serial.println(("PPM"));
  Serial.print(F("EC:")); Serial.print(EC_val); Serial.println(("µS/cm"));
  Serial.print(F("Temperature:")); Serial.println(sensor::waterTemp, 2);

  lcd.setCursor(5, 0);
  lcd.print("TDS: ");
  lcd.setCursor(5, 1);
  lcd.print(sensor::tds);

  lcd.setCursor(10, 0);
  lcd.print("EC: ");
  lcd.setCursor(10, 1);
  lcd.print(sensor::ec * 1000.0, 1);
}
