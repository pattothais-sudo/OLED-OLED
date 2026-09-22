# 🚧 Catraca Automática com ESP32

Projeto desenvolvido utilizando um **ESP32** para controlar uma catraca automática por meio de um **sensor ultrassônico**, **servo motor**, **LED** e **display OLED**.

O sistema identifica a aproximação de um objeto por meio da distância medida pelo sensor ultrassônico. Quando um objeto está próximo, a catraca é aberta automaticamente pelo servo motor e o LED é acionado. Quando o objeto se afasta, a catraca retorna à posição inicial e o LED é desligado.

---

## 📌 Objetivo do projeto

Desenvolver um sistema automatizado capaz de:

* Detectar a aproximação de um objeto;
* Medir a distância utilizando um sensor ultrassônico;
* Abrir a catraca automaticamente;
* Fechar a catraca quando o objeto se afastar;
* Acionar um LED durante a abertura;
* Exibir informações no display OLED;
* Mostrar informações de funcionamento no Monitor Serial.

---

## 🛠️ Tecnologias e bibliotecas utilizadas

* **ESP32**
* **Arduino IDE**
* **C/C++**
* **ESP32Servo**
* **Wire**
* **Adafruit GFX**
* **Adafruit SSD1306**

### Bibliotecas

```cpp
#include <ESP32Servo.h>
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>
```

---

## 🔌 Componentes utilizados

| Componente          | Função                                   |
| ------------------- | ---------------------------------------- |
| ESP32               | Controlador principal do projeto         |
| Servo motor         | Responsável por abrir e fechar a catraca |
| Sensor ultrassônico | Mede a distância do objeto               |
| LED                 | Indica quando a catraca está aberta      |
| Display OLED        | Exibe o estado do sistema                |

---

## 📍 Ligações dos componentes

| Componente                  |      Pino ESP32 |
| --------------------------- | --------------: |
| Servo motor                 |         GPIO 23 |
| TRIG do sensor ultrassônico |         GPIO 12 |
| ECHO do sensor ultrassônico |         GPIO 14 |
| LED                         |          GPIO 4 |
| OLED SDA/SCL                | Comunicação I2C |

O display OLED utiliza comunicação **I2C** e possui o endereço `0x3C`.

---

## ⚙️ Funcionamento

O sistema inicia configurando o servo motor, sensor ultrassônico, LED e display OLED.

Após a inicialização, o OLED apresenta:

```text
Sistema iniciado
```

O sensor ultrassônico realiza medições continuamente para identificar a distância de um objeto.

### 🟢 Abertura da catraca

Quando a distância detectada for **menor ou igual a 90 cm**, o sistema:

1. Posiciona o servo em **90°**;
2. Liga o LED;
3. Define que a catraca está aberta;
4. Exibe `"Abrindo catraca"` no OLED;
5. Informa o funcionamento pelo Monitor Serial.

### 🔴 Fechamento da catraca

Quando a distância for **maior ou igual a 110 cm**, o sistema:

1. Retorna o servo para **0°**;
2. Desliga o LED;
3. Define que a catraca está fechada;
4. Exibe `"Fechando catraca"` no OLED;
5. Informa o funcionamento pelo Monitor Serial.

---

## 📏 Distâncias configuradas

O projeto utiliza duas distâncias diferentes para evitar que a catraca fique abrindo e fechando constantemente quando o objeto estiver próximo do limite.

```cpp
#define DISTANCIA_ABERTURA 90
#define DISTANCIA_FECHAMENTO 110
```

### Abertura

```text
Distância ≤ 90 cm
        ↓
Catraca abre
```

### Fechamento

```text
Distância ≥ 110 cm
        ↓
Catraca fecha
```

Essa diferença entre os valores é conhecida como uma **faixa de histerese**, ajudando a deixar o comportamento do sistema mais estável.

---

## 🔄 Fluxo do sistema

```text
        INÍCIO
           ↓
   Inicializa componentes
           ↓
    Mede a distância
           ↓
   ┌───────┴────────┐
   ↓                ↓
≤ 90 cm          ≥ 110 cm
   ↓                ↓
Abre catraca     Fecha catraca
   ↓                ↓
LED ligado       LED desligado
   ↓                ↓
OLED informa     OLED informa
   └───────┬────────┘
           ↓
      Nova medição
           ↓
         LOOP
```

---

## 🖥️ Display OLED

O display OLED é utilizado para informar o estado atual da catraca.

Algumas mensagens exibidas são:

```text
Sistema iniciado
```

```text
Abrindo catraca
```

```text
Fechando catraca
```

---

## 📟 Monitor Serial

O sistema também envia informações para o Monitor Serial utilizando uma velocidade de **115200 baud**.

Exemplo:

```text
Sistema iniciado!
Distancia: 75.32 cm
Objeto detectado!
Abrindo Servo
LED ligado
```

Quando o objeto se afasta:

```text
Distancia: 120.45 cm
Nenhum objeto proximo.
Fechando Servo
LED desligado
```

---

## 📐 Cálculo da distância

A distância é calculada utilizando o tempo que o sinal ultrassônico leva para retornar ao sensor.

No código:

```cpp
float distancia = (duracao * 0.0343) / 2.0;
```

O valor `0.0343` representa aproximadamente a velocidade do som em centímetros por microssegundo.

A divisão por `2` ocorre porque o sinal realiza um percurso de ida e volta:

```text
Sensor → Objeto → Sensor
```

---

## 🧠 Controle do estado da catraca

O sistema utiliza a variável:

```cpp
bool catracaAberta = false;
```

Ela permite identificar se a catraca está atualmente aberta ou fechada.

### `false`

```text
Catraca fechada
```

### `true`

```text
Catraca aberta
```

Isso evita que o servo receba comandos repetidos desnecessariamente enquanto a catraca já estiver no estado correspondente.

---

## 🚨 Tratamento de erro

Caso o sensor ultrassônico não receba um eco válido, a função retorna `-1`.

Nesse caso, o sistema informa no Monitor Serial:

```text
Nenhum eco recebido.
```

E aguarda uma nova medição.

---

## ▶️ Como executar o projeto

### 1. Instalar a Arduino IDE

Instale a Arduino IDE e configure a placa ESP32.

### 2. Instalar as bibliotecas

No gerenciador de bibliotecas da Arduino IDE, instale:

* ESP32Servo
* Adafruit GFX Library
* Adafruit SSD1306

A biblioteca `Wire` já faz parte do ambiente Arduino/ESP32.

### 3. Montar o circuito

Conecte os componentes aos GPIOs definidos no código:

```text
Servo → GPIO 23
TRIG → GPIO 12
ECHO → GPIO 14
LED → GPIO 4
OLED → I2C
```

### 4. Conectar o ESP32

Conecte o ESP32 ao computador utilizando um cabo USB.

### 5. Compilar e enviar

Selecione a placa ESP32 correspondente na Arduino IDE e envie o código para a placa.

### 6. Abrir o Monitor Serial

Configure o Monitor Serial para:

```text
115200 baud
```

---

## 📂 Estrutura do projeto

```text
Catraca-ESP32/
│
├── Catraca-ESP32.ino
└── README.md
```

---

## 🎯 Resultado esperado

Ao iniciar o sistema, o ESP32 realiza continuamente a leitura da distância.

Quando um objeto se aproxima até **90 cm**, a catraca é aberta e o LED é ligado.

Quando o objeto se afasta para **110 cm ou mais**, a catraca é fechada e o LED é desligado.

O display OLED e o Monitor Serial fornecem informações sobre o estado atual do sistema.

---

## 👩‍💻 Autoria

**Thais Costa Patto de Souza**

Projeto desenvolvido para fins acadêmicos utilizando ESP32 e componentes eletrônicos.

---

## 📄 Licença

Projeto desenvolvido para fins educacionais e acadêmicos.
