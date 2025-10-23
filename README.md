#🌐 Projeto IoT – Monitoramento de Temperatura, Umidade e Luminosidade com ESP32#
📌 Descrição do Projeto

Este projeto tem como objetivo implementar sensores de temperatura e umidade (DHT11) e luminosidade (LDR) no ESP32, integrando-os a um broker MQTT para que os dados possam ser lidos em tempo real por meio do aplicativo MyMQTT.

Além da leitura, também é possível enviar comandos via MQTT para acionar ou desligar um LED conectado ao ESP32, simulando um atuador.

🧰 Componentes Utilizados
Quantidade	Componente	Função
1	ESP32	Microcontrolador principal
1	DHT11	Sensor de temperatura e umidade
1	LDR (Sensor de Luminosidade)	Sensor de luminosidade ambiente
1	Resistor 10kΩ	Pull-down do LDR
1	LED + Resistor 220Ω	Atuador de simulação
—	Cabo USB	Comunicação e alimentação
—	Protoboard e jumpers	Montagem do circuito
