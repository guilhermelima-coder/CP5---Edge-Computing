# 🌐 Projeto IoT – Monitoramento de Temperatura, Umidade e Luminosidade com ESP32
📌 Descrição do Projeto

Este projeto tem como objetivo implementar sensores de temperatura e umidade (DHT11) e luminosidade (LDR) no ESP32, integrando-os a um broker MQTT para que os dados possam ser lidos em tempo real por meio do aplicativo MyMQTT.

Além da leitura, também é possível enviar comandos via MQTT para acionar ou desligar um LED conectado ao ESP32, simulando um atuador.

---

## 🧰 Componentes Utilizados
Quantidade	Componente	Função
1	ESP32	Microcontrolador principal
1	DHT11	Sensor de temperatura e umidade
1	LDR (Sensor de Luminosidade)	Sensor de luminosidade ambiente
1	Resistor 10kΩ	Pull-down do LDR
1	LED + Resistor 220Ω	Atuador de simulação
—	Cabo USB	Comunicação e alimentação
—	Protoboard e jumpers	Montagem do circuito

---

## 📝 Esquemático de Ligação
![arduino-nano-esp32-dht11-lcd-i2c-wiring-diagram](https://github.com/user-attachments/assets/464e19e7-bc53-4cd8-be0b-c4a9e65228df)
![arduino-nano-esp32-dht11-relay-wiring-diagram](https://github.com/user-attachments/assets/6160b548-9ea1-44a4-b653-a56576a584f0)
<img width="750" height="330" alt="ESP32-Wi-Fi-Weather-Station-using-DHT11-and-BMP180-Sensor-Circuit-Diagram" src="https://github.com/user-attachments/assets/7b5d59f6-865c-46d8-956e-8310848af067" />
![featured-image-esp32-mqtt-publish-subscribe](https://github.com/user-attachments/assets/4302209b-0458-44e4-843d-43a035ebc34f)
<img width="3632" height="1277" alt="Huzzah32_Blink_CircuitDiagramAndSchematic_Fritzing" src="https://github.com/user-attachments/assets/d6d78835-631d-4152-9304-d94a2ae7fe8b" />

---

## 🔌 Conexões:
| Componente | Pino no Sensor | Pino ESP32 | Descrição |
|-------------|-------------|-------------|-------------|
| DHT11   | VCC   | 3V3   | Alimentação 3.3V   |
| DHT11   | GND   | GND   | Terra   |
| DHT11   | DATA   | D4 (GPIO4)   | Comunicação de dados   |
| LDR + 10kΩ   | Divisor Tensão   | D34 (GPIO34)  | Leitura analógica do LDR   |
| LED   | Anodo (+)   | D2 (GPIO2)   | Saída digital de controle   |
| LED   | Catodo (–)   | GND   | Terra   |

---

## ⚙️ Configuração do Ambiente
1. Instalação do Arduino IDE

- Baixar e instalar Arduino IDE.

- Adicionar suporte ao ESP32:

  - Arquivo > Preferências > URL adicionais de placas:

      https://dl.espressif.com/dl/package_esp32_index.json


  - Ferramentas > Placa > Gerenciador de Placas → Pesquisar e instalar ESP32.

2. Bibliotecas Necessárias

Instale as seguintes bibliotecas via Gerenciador de Bibliotecas:

- DHT sensor library (by Adafruit)

- Adafruit Unified Sensor

- WiFi

- PubSubClient (para MQTT)

---

## 📡 Configuração do Broker MQTT

Para este projeto, utilizamos um broker MQTT gratuito:

- Broker: test.mosquitto.org

- Porta: 1883

- Tópico de publicação (sensores):

  - esp32/temperatura

  - esp32/umidade

  - esp32/luminosidade

- Tópico de subscrição (atuador):

  - esp32/led

Você também pode usar um broker local, como Mosquitto.

---

## 📲 Configuração do MyMQTT
<img width="1200" height="600" alt="opengraph" src="https://github.com/user-attachments/assets/34028401-f7bb-490e-9508-f8cc3b0eb710" />

![300x0w](https://github.com/user-attachments/assets/d590905e-0c83-44e5-9d0d-423655017629)

<img width="715" height="787" alt="image" src="https://github.com/user-attachments/assets/20c50126-1996-4418-837a-75d0a2f1740f" />

1. Instale o aplicativo MyMQTT no seu smartphone.

2. Vá em Configurações > Broker:

- Host: test.mosquitto.org

- Port: 1883

- Client ID: (qualquer nome único, ex: esp32_user)

- QoS: 0

3. Adicione os tópicos:

- esp32/temperatura

- esp32/umidade

- esp32/luminosidade

- esp32/led (para enviar comandos “on” e “off”)

---

## 🧠 Código Fonte (ESP32)
```cpp
#include <WiFi.h>
#include <PubSubClient.h>
#include "DHT.h"

#define DHTPIN 4
#define DHTTYPE DHT11
#define LDRPIN 34
#define LEDPIN 2

const char* ssid = "NOME_DA_SUA_REDE";
const char* password = "SENHA_DA_SUA_REDE";
const char* mqtt_server = "test.mosquitto.org";

WiFiClient espClient;
PubSubClient client(espClient);
DHT dht(DHTPIN, DHTTYPE);

void callback(char* topic, byte* message, unsigned int length) {
  String msg;
  for (int i = 0; i < length; i++) {
    msg += (char)message[i];
  }
  if (String(topic) == "esp32/led") {
    if (msg == "on") digitalWrite(LEDPIN, HIGH);
    else if (msg == "off") digitalWrite(LEDPIN, LOW);
  }
}

void reconnect() {
  while (!client.connected()) {
    if (client.connect("ESP32Client")) {
      client.subscribe("esp32/led");
    } else {
      delay(5000);
    }
  }
}

void setup() {
  Serial.begin(115200);
  pinMode(LEDPIN, OUTPUT);
  dht.begin();

  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.println("Conectando ao WiFi...");
  }

  client.setServer(mqtt_server, 1883);
  client.setCallback(callback);
}

void loop() {
  if (!client.connected()) {
    reconnect();
  }
  client.loop();

  float temperatura = dht.readTemperature();
  float umidade = dht.readHumidity();
  int luminosidade = analogRead(LDRPIN);

  if (!isnan(temperatura) && !isnan(umidade)) {
    client.publish("esp32/temperatura", String(temperatura).c_str());
    client.publish("esp32/umidade", String(umidade).c_str());
  }

  client.publish("esp32/luminosidade", String(luminosidade).c_str());

  delay(3000);
}
```
---

## 🧪 Testes Realizados

- ✅ Conexão WiFi estável no ESP32.

- ✅ Publicação periódica de dados dos sensores no broker MQTT.

- ✅ Visualização dos valores no app MyMQTT em tempo real.

- ✅ Controle do LED via mensagens MQTT (“on” / “off”).

---

## 📂 Organização do Repositório
📁 Projeto_IoT_ESP32
│
├─ 📄 README.md
├─ 📄 main.ino
├─ 📸 imagens/
│   ├─ esquematico.png
│   ├─ mymqtt_config.png
│   └─ resultado_teste.png
└─ 📁 docs/
    ├─ bibliotecas_utilizadas.txt
    ├─ links_de_referencia.md
    └─ instrucoes_instalacao_ide.pdf

---

## 📚 Referências

- Arduino IDE — https://www.arduino.cc

- Mosquitto — https://mosquitto.org

- MyMQTT — Google Play

- ESP32 — Documentação oficial: https://www.espressif.com

---

# 🏁 Conclusão

Este projeto demonstra de forma prática como:

- Integrar sensores com um microcontrolador ESP32;

- Utilizar protocolos IoT (MQTT) para comunicação em tempo real;

- Visualizar dados e atuar remotamente usando um aplicativo móvel.

A base desenvolvida pode ser facilmente expandida para incluir novos sensores, atuadores e dashboards profissionais.

---

## 👥 Projeto IoT desenvolvido por:
  ## Guilherme Eduardo de Lima
  ## Guilherme de Paula
  ## Enzo de Faria
  ## Matheus Gomes

