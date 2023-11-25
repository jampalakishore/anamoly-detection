#include <WiFi.h>
#include <PubSubClient.h>

#define WIFISSID "YourWiFiSSID"
#define PASSWORD "YourWiFiPassword"
#define TOKEN "YourUbidotsToken"
#define MQTT_CLIENT_NAME "YourMQTTClientName"

#define VARIABLE_LABEL "..."// Assing the variable label
#define DEVICE_LABEL "..."// Assig the device label
#define VIBRATION_SENSOR_PIN 34
#define THERMISTOR_PIN 35

char mqttBroker[] = "industrial.api.ubidots.com";
char payload[100];
char topic[150];
char str_vibration[10];
char str_temperature[10];

WiFiClient ubidots;
PubSubClient client(ubidots);

void callback(char *topic, byte *payload, unsigned int length) {
  char p[length + 1];
  memcpy(p, payload, length);
  p[length] = NULL;
  Serial.write(payload, length);
  Serial.println(topic);
}

void reconnect() {
  while (!client.connected()) {
    Serial.println("Attempting MQTT connection...");

    if (client.connect(MQTT_CLIENT_NAME, TOKEN, "")) {
      Serial.println("Connected");
    } else {
      Serial.print("Failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again in 2 seconds");
      delay(2000);
    }
  }
}

void setup() {
  Serial.begin(115200);
  WiFi.begin(WIFISSID, PASSWORD);
  pinMode(VIBRATION_SENSOR_PIN, INPUT);
  pinMode(THERMISTOR_PIN, INPUT);

  Serial.println();
  Serial.print("Waiting for WiFi...");

  while (WiFi.status() != WL_CONNECTED) {
    Serial.print(".");
    delay(500);
  }

  Serial.println("");
  Serial.println("WiFi Connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
  client.setServer(mqttBroker, 1883);
  client.setCallback(callback);
}

void loop() {
  if (!client.connected()) {
    reconnect();
  }

  sprintf(topic, "%s%s", "/v1.6/devices/", DEVICE_LABEL);
  sprintf(payload, "%s", "");
  sprintf(payload, "{\"%s\":", VARIABLE_LABEL);

  int vibrationValue = digitalRead(VIBRATION_SENSOR_PIN);
  if (vibrationValue == HIGH) {
    float vibrationSensorValue = random(256);
    dtostrf(vibrationSensorValue, 4, 2, str_vibration);
    sprintf(payload, "%s {\"vibration\": %s}", payload, str_vibration);

    // Read temperature from the thermistor
    int thermistorValue = analogRead(THERMISTOR_PIN);
    float voltage = thermistorValue * (3.3 / 4095.0);
    float resistance = (3.3 - voltage) / (voltage / 10000.0);
    float temperature = 1.0 / ((1.0 / 298.15) + (1.0 / 10000.0) * log(resistance / 10000.0));
    temperature -= 273.15; // Convert Kelvin to Celsius

    dtostrf(temperature, 4, 2, str_temperature);
    sprintf(payload, "%s, \"temperature\": %s}", payload, str_temperature);

    Serial.println("Publishing data to Ubidots Cloud");
    client.publish(topic, payload);
  } else {
    Serial.println("No vibration");
  }

  client.loop();
  delay(500);
}
