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

## 📊 Dashboard em Python (Monitoramento em Tempo Real)

Para complementar o projeto e permitir a visualização dos dados dos sensores em tempo real, foi desenvolvido um Dashboard interativo utilizando Python, com as bibliotecas Dash, Plotly e MQTT.

Este painel exibe gráficos dinâmicos de temperatura, umidade e luminosidade, atualizados automaticamente a partir das mensagens recebidas via broker MQTT.

---

## ⚙️ Estrutura e Funcionamento

O código do dashboard estabelece uma conexão MQTT com o mesmo broker usado pelo ESP32.
Sempre que uma nova leitura é publicada nos tópicos:

esp32/temperatura

esp32/umidade

esp32/luminosidade
o dashboard recebe esses valores, armazena-os temporariamente e atualiza os gráficos em tempo real.

A interface foi criada com o Dash (Plotly) e organizada em três painéis principais:

Gráfico de Temperatura (°C) — mostra a variação da temperatura ambiente ao longo do tempo.

Gráfico de Umidade (%) — exibe as mudanças na umidade relativa do ar.

Gráfico de Luminosidade (LDR) — representa a intensidade luminosa captada pelo sensor.
Os gráficos são atualizados a cada 5 segundos por meio do componente dcc.Interval().

---

## 🧠 Código Fonte (DASHBOARD)
```cpp
import dash
from dash import dcc, html
import dash_bootstrap_components as dbc
from dash.dependencies import Output, Input
import plotly.graph_objs as go
import paho.mqtt.client as mqtt
from collections import deque
import threading
import datetime
import json
 
# ---------- CONFIGURAÇÕES MQTT ----------
MQTT_BROKER = "44.223.0.185"
MQTT_PORT = 1883
TOPICS = ["esp32/temperatura", "esp32/umidade", "esp32/luminosidade"]
 
# ---------- VARIÁVEIS GLOBAIS ----------
data_temp = deque(maxlen=20)
data_hum = deque(maxlen=20)
data_ldr = deque(maxlen=20)
 
timestamps_temp = deque(maxlen=20)
timestamps_hum = deque(maxlen=20)
timestamps_ldr = deque(maxlen=20)
 
last_update = "Aguardando dados..."
 
# ---------- CALLBACK MQTT ----------
def on_connect(client, userdata, flags, rc):
    print("Conectado ao broker MQTT com código:", rc)
    for topic in TOPICS:
        client.subscribe(topic)
 
def on_message(client, userdata, msg):
    global last_update
    payload = msg.payload.decode()
 
    try:
        data = json.loads(payload)
        time_str = data.get("timestamp", datetime.datetime.now().strftime("%H:%M:%S"))
        value = float(data.get("value", 0))
 
        if msg.topic == "esp32/temperatura":
            data_temp.append(value)
            timestamps_temp.append(time_str)
        elif msg.topic == "esp32/umidade":
            data_hum.append(value)
            timestamps_hum.append(time_str)
        elif msg.topic == "esp32/luminosidade":
            data_ldr.append(value)
            timestamps_ldr.append(time_str)
 
        last_update = time_str
        print("Payload recebido:", payload)
 
    except Exception as e:
        print("Erro ao processar mensagem MQTT:", e)
        print("Payload recebido:", payload)
 
def mqtt_thread():
    client = mqtt.Client()
    client.on_connect = on_connect
    client.on_message = on_message
    client.connect(MQTT_BROKER, MQTT_PORT, 60)
    client.loop_forever()
 
# ---------- INICIAR THREAD MQTT ----------
threading.Thread(target=mqtt_thread, daemon=True).start()
 
# ---------- DASHBOARD ----------
app = dash.Dash(__name__, external_stylesheets=[dbc.themes.BOOTSTRAP])
app.title = "Vinheria Agnello - IoT Dashboard"
 
app.layout = dbc.Container([
    html.Br(),
    html.H1("🍷 Vinheria Agnello — Dashboard IoT",
            style={"textAlign": "center", "color": "#800020", "fontWeight": "bold"}),
 
    html.Hr(),
 
    dbc.Row([
        dbc.Col(html.Div([
            html.H5("Última atualização:", style={"color": "#555"}),
            html.P(id="update-time", style={"fontSize": "18px", "fontWeight": "bold", "color": "#800020"})
        ]), width=4),
    ], justify="center"),
 
    dbc.Row([
        dbc.Col(dcc.Graph(id="temp-graph"), md=4),
        dbc.Col(dcc.Graph(id="hum-graph"), md=4),
        dbc.Col(dcc.Graph(id="ldr-graph"), md=4),
    ]),
 
    dcc.Interval(
        id="interval-component",
        interval=5000,  # Atualiza a cada 5 segundos
        n_intervals=0
    )
], fluid=True)
 
# ---------- CALLBACKS ----------
@app.callback(
    [Output("temp-graph", "figure"),
     Output("hum-graph", "figure"),
     Output("ldr-graph", "figure"),
     Output("update-time", "children")],
    Input("interval-component", "n_intervals")
)
def update_graph(n):
    # ----- Gráfico de Temperatura -----
    fig_temp = go.Figure(go.Scatter(
        x=list(timestamps_temp),  # tempo no eixo X
        y=list(data_temp),        # valor no eixo Y
        mode="lines+markers",
        name="Temperatura (°C)",
        line=dict(color="#800020")
    ))
    fig_temp.update_layout(
        title="Temperatura",
        xaxis=dict(title="Tempo"),
        yaxis=dict(title="°C"),
        template="plotly_dark"
    )
 
    # ----- Gráfico de Umidade -----
    fig_hum = go.Figure(go.Scatter(
        x=list(timestamps_hum),
        y=list(data_hum),
        mode="lines+markers",
        name="Umidade (%)",
        line=dict(color="#FFD700")
    ))
    fig_hum.update_layout(
        title="Umidade",
        xaxis=dict(title="Tempo"),
        yaxis=dict(title="%"),
        template="plotly_dark"
    )
 
    # ----- Gráfico de Luminosidade -----
    fig_ldr = go.Figure(go.Scatter(
        x=list(timestamps_ldr),
        y=list(data_ldr),
        mode="lines+markers",
        name="Luminosidade",
        line=dict(color="#9370DB")
    ))
    fig_ldr.update_layout(
        title="Luminosidade",
        xaxis=dict(title="Tempo"),
        yaxis=dict(title="LDR"),
        template="plotly_dark"
    )
 
    return fig_temp, fig_hum, fig_ldr, last_update
 
 
if __name__ == "__main__":
    app.run(debug=True, host="0.0.0.0", port=8050)
```
---

## 💻 Tecnologias Utilizadas

Dash / Plotly → Criação da interface e dos gráficos interativos.

Dash Bootstrap Components → Estilização com tema escuro e design responsivo.

Paho-MQTT → Comunicação com o broker MQTT em tempo real.

Threading e JSON → Processamento paralelo das mensagens e estruturação dos dados recebidos.

🔌 Execução do Dashboard

Certifique-se de ter o Python 3 instalado no sistema.

Instale as dependências necessárias executando no terminal:

pip install dash plotly dash-bootstrap-components paho-mqtt


Verifique se o ESP32 já está publicando dados no broker MQTT.

Salve o arquivo do dashboard como dashboard.py.

Execute o dashboard com o comando:

python dashboard.py


Após a inicialização, o terminal exibirá uma mensagem semelhante a:

Running on http://127.0.0.1:8050/


Acesse o endereço no navegador (geralmente http://localhost:8050) para visualizar o dashboard em tempo real.

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

