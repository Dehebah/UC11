#include "DHTesp.h"
#include "LiquidCrystal_I2C.h"
#define I2C_ADDR    0x27
#define LCD_COLUMNS 20
#define LCD_LINES   4
#include "WiFi.h"
#include "HTTPClient.h"


const char* ssid = "Wokwi-GUEST";
const char* password = "";


const int LED1 = 4;
const int LED2 = 2;
const int DHT_PIN = 15;

DHTesp dhtSensor;

LiquidCrystal_I2C lcd(I2C_ADDR, LCD_COLUMNS, LCD_LINES);

void setup() {
  Serial.begin(115200);
  
  lcd.init();
  lcd.backlight();
  lcd.print("   Conectando   ");
  lcd.setCursor(0, 1);
  lcd.print("      WiFi       ");
   delay(3000);
  //Configuração do WIFI
  Serial.println(); 
  Serial.print("Conectando Wifi ");
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
   delay(500);
    Serial.print(".");
  }
  Serial.println();
  Serial.print("Conectado a Rede");
  lcd.setCursor(0, 0);
  lcd.print(" Status da Rede");
  lcd.setCursor(0, 1);
  lcd.print("Conectado a Rede ");
   delay(3000);
  Serial.println();
  Serial.println("--------------------------");

  lcd.setCursor(0, 0);
  lcd.print(" Atividade UC11");
  lcd.setCursor(0, 1);
  lcd.print("SA2  Projeto IOT");
   delay(3000);
  
  
  dhtSensor.setup(DHT_PIN, DHTesp::DHT22);
  pinMode(LED1, OUTPUT); // Certifique-se de definir o pino do LED1 como saída
  pinMode(LED2, OUTPUT); // Certifique-se de definir o pino do LED2 como saída

  
}

void loop() {
  

 TempAndHumidity data = dhtSensor.getTempAndHumidity(); // Leitura dos dados do sensor
  
  float temperatureValue = data.temperature; // Atribuição da temperatura
  float humidadeValue = data.humidity; // Atribuição da Humidade

  Serial.println("Temperatura: " + String(data.temperature, 0) + "°C");
  Serial.println("Umidade: " + String(data.humidity, 0) + "%");
  Serial.println("--------------------------");

  lcd.setCursor(0, 0);
  lcd.print("   Temp.: " + String(data.temperature, 0) + "\xDF"+"C  ");
  lcd.setCursor(0, 1);
  lcd.print("  Umidade:  " + String(data.humidity, 0) + "% ");
  //lcd.print("Wokwi Online IoT");

  if (data.temperature >= 35) { 
    digitalWrite(LED1, HIGH);
  } else {
    digitalWrite(LED1, LOW);
  }

  if (data.humidity >= 70) { 
    digitalWrite(LED2, HIGH);
  } else {
    digitalWrite(LED2, LOW);
  }

  delay(1000);
  
  //Comunicação via Protocolo HTTP
  String url = "https://api.thingspeak.com/update?api_key=V0RB36C8PU5SIXXX&field1=" + String(temperatureValue) + "&field2=" + String(humidadeValue);

  //Objeto HTTP
  HTTPClient Client;

  Client.begin(url);
  int httpCode = Client.GET();
  Serial.print("Código Recebido da API: ");
  Serial.println(httpCode);

  if (httpCode >= 200 && httpCode <= 299) {
    String payload = Client.getString();
    Serial.println(payload);
  } else {
    Serial.println("Problemas ao efetuar o request");
  }
}




