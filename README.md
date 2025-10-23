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


