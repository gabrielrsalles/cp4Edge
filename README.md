# Projeto de Monitoramento com ESP32, DHT22 e Potenciômetro

Este projeto utiliza um ESP32 para capturar dados de um sensor de temperatura e umidade DHT22, além de ler a posição de um potenciômetro. Os dados são enviados para o ThingSpeak para monitoramento remoto.

## Integrantes

- **Gabriel Ribeiro** - 563584
- **Felipe Viana** - 565341
- **Joan Ferreira** - 562913
- **Felipe Bonilha** - 562356

## Componentes

- **ESP32**: Microcontrolador que gerencia o projeto.
- **DHT22**: Sensor de temperatura e umidade.
- **Potenciômetro**: Sensor de ajuste de posição, utilizado para simular a velocidade de um carro da Fórmula E.

## Bibliotecas Necessárias

- `WiFi.h`: Para conexão com redes Wi-Fi.
- `HTTPClient.h`: Para enviar dados via HTTP.
- `DHT.h`: Para ler os dados do sensor DHT22.

## Circuito

- **DHT22** conectado ao pino GPIO 15 do ESP32.
- **Potenciômetro** conectado ao pino GPIO 34 do ESP32.

## Configuração

1. Conecte o sensor DHT22 ao pino GPIO15 e o potenciômetro ao pino GPIO34.
2. Altere as variáveis de rede Wi-Fi e API do ThingSpeak conforme necessário.
3. Carregue o código no seu ESP32.

## Código

```cpp
#include <WiFi.h>
#include <HTTPClient.h>
#include <DHT.h>

#define DHTPIN 15 // Pino GPIO15 do ESP32 para o DHT22
#define DHTTYPE DHT22 // Tipo de sensor DHT (DHT22)
DHT dht(DHTPIN, DHTTYPE);

#define POT_PIN 34 // Pino GPIO34 do ESP32 para o Potenciômetro

// Credenciais
const char* ssid = "Wokwi-GUEST"; // Rede Wi-Fi
const char* password = ""; // Senha da rede Wi-Fi
const char* apiKey = "OTTZR7G8AB8GUR3R"; // Write API Key
const char* server = "http://api.thingspeak.com"; // Servidor ThingSpeak

void setup() {
  Serial.begin(9600);
  dht.begin();

  // Configuração dos pinos
  pinMode(POT_PIN, INPUT);

  // Inicialização e loop de verificação da rede Wi-Fi
  WiFi.begin(ssid, password);
  Serial.print("Conectando ao WiFi");
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println(" conectado!");
}

void loop() {
  if (WiFi.status() == WL_CONNECTED) {
    // Leitura dos sensores
    float h = dht.readHumidity();
    float t = dht.readTemperature();
    int potValue = analogRead(POT_PIN); // Leitura do valor do Potenciômetro
    float speed = map(potValue, 0, 4095, 0, 100); // Mapeamento do valor do potenciômetro para simular a velocidade de um carro da Fórmula E (0 a 322 km/h)

    if (isnan(h) || isnan(t)) {
      Serial.println("Falha ao ler o sensor DHT11!");
      return;
    }

    Serial.println(t);
    Serial.println(h);

    // Envio de dados para o ThingSpeak
    HTTPClient http;
    String url = String(server) + "/update?api_key=" + apiKey + "&field1=" + String(t) +
                                 "&field2=" + String(h) + "&field3=" + String(speed);
    http.begin(url);

    int httpCode = http.GET();
    if (httpCode > 0) {
      String payload = http.getString(); // Resp
