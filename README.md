# Monitoramento de Temperatura com Raspberry Pi e MQTT

Projeto de monitoramento de temperatura utilizando Raspberry Pi, sensores DS18B20 e o protocolo MQTT.

A solução foi desenvolvida como uma prova de conceito para monitorar a temperatura próxima aos aparelhos de ar-condicionado da sala-cofre do Data Center do Ministério do Trabalho e Emprego (MTE).

> Este projeto realiza apenas a coleta e a transmissão das temperaturas. Ele não controla os aparelhos de ar-condicionado.

## Objetivo

Desenvolver uma solução capaz de:

- Ler a temperatura com sensores DS18B20;
- Enviar as medições pela rede utilizando MQTT;
- Centralizar as mensagens em um Broker Mosquitto;
- Identificar individualmente cada sensor;
- Permitir uma futura integração com Zabbix, Grafana ou outro sistema de monitoramento.

## Arquitetura

O projeto utiliza dois Raspberry Pi:

### Raspberry Pi Publisher

Responsável por:

- Ler os sensores DS18B20;
- Conectar-se ao Broker MQTT;
- Publicar as temperaturas em tópicos MQTT.

### Raspberry Pi Broker

Responsável por:

- Executar o Mosquitto;
- Receber as mensagens do Publisher;
- Distribuir as mensagens aos clientes inscritos.

### Fluxo de comunicação

```text
Sensor DS18B20
       |
       | 1-Wire
       v
Raspberry Pi Publisher
       |
       | MQTT
       v
Raspberry Pi Broker
Mosquitto
       |
       v
Zabbix, dashboard ou outro cliente

```

## Tecnologias utilizadas

- Raspberry Pi OS;
- Python 3;
- MQTT;
- Eclipse Mosquitto;
- Eclipse Paho MQTT;
- Biblioteca w1thermsensor;
- Sensor DS18B20;
- Protocolo 1-Wire.

## Materiais
- 2 Raspberry Pi 3;
- 2 sensores DS18B20;
- 1 protoboard;
- 3 cabos jumper macho-fêmea;
    - 1 cabo preto;
    - 1 cabo vermelho;
    - 1 cabo amarelo.
- 1 resistor de 4,7 kΩ;
- 2 cartões MicroSD de 16 GB ou 32 GB;
- 2 fontes de alimentação compatíveis;
- Cabos HDMI, teclado e mouse para a configuração inicial.

## Preparação dos cartões MicroSD

Primeiramente, faça o download e instale o **Raspberry Pi Imager** em seu computador.

Depois, insira o cartão MicroSD no leitor USB e abra o Raspberry Pi Imager. Selecione as seguintes opções:

- **Raspberry Pi Device:** Raspberry Pi 3;
- **Operating System:** Raspberry Pi OS 64-bit;
- **Storage:** cartão MicroSD que será utilizado.

Nas configurações adicionais do Raspberry Pi Imager, também é possível definir:

- Nome do dispositivo;
- Nome de usuário;
- Senha;
- Rede Wi-Fi;
- País da rede Wi-Fi;
- Fuso horário;
- Layout do teclado;
- Acesso remoto por SSH.

Os cartões MicroSD devem ser preparados separadamente para cada Raspberry Pi.

Sugestão de nomes para identificar os dispositivos:

```text
raspberry-publisher
raspberry-broker
```

## Inicialização dos Raspberry Pi

Após a instalação do Raspberry Pi OS no cartão MicroSD:

1. Remova o cartão MicroSD do leitor USB;
2. Insira o cartão no Raspberry Pi correspondente;
3. Conecte o cabo HDMI ao Raspberry Pi e ao monitor;
4. Conecte o teclado e o mouse;
5. Conecte a fonte de alimentação;
6. Aguarde a inicialização do Raspberry Pi OS;
7. Conecte o Raspberry Pi à mesma rede local que será utilizada pelo outro dispositivo.

Repita o procedimento para o segundo Raspberry Pi.

Para executar os comandos das próximas etapas, abra o aplicativo Terminal do Raspberry Pi OS.

>Depois da configuração inicial, os dispositivos também poderão ser acessados remotamente por SSH, caso essa opção tenha sido habilitada no Raspberry Pi Imager.

## Preparação dos Raspberry Pi

O Raspberry Pi OS foi instalado nos cartões MicroSD utilizando o Raspberry Pi Imager.

Depois da inicialização, atualizar os dois dispositivos:
```bash
sudo apt update

sudo apt full-upgrade -y
```

Para identificar o endereço IP de cada Raspberry Pi:

```bash
hostname -I
```

Para testar a comunicação entre os dispositivos:

```bash
ping -c 4 IP_DO_OUTRO_RASPBERRY
```

## Configuração do Broker MQTT

No Raspberry Pi que funcionará como Broker:

```bash
sudo apt install mosquitto mosquitto-clients -y

sudo systemctl enable mosquitto

sudo systemctl restart mosquitto

sudo systemctl status mosquitto
```

Para permitir conexões pela rede local durante os testes, criar o arquivo:

```bash
sudo nano /etc/mosquitto/conf.d/projeto-temperatura.conf
```

Adicionar:

```text
listener 1883

allow_anonymous true
```

Reiniciar o Mosquitto:

```bash
sudo systemctl restart mosquitto
```
>A conexão anônima deve ser utilizada somente em ambiente de testes. Em uma implantação real, devem ser configurados autenticação, controle de acesso e criptografia.

## Teste do MQTT

No Raspberry Pi Broker, iniciar o Subscriber:

```bash
mosquitto_sub -h IP_DO_BROKER -p 1883 -t "teste/temperatura" -v
```

No Raspberry Pi Publisher, enviar uma mensagem:

```bash
mosquitto_pub -h IP_DO_BROKER -p 1883 -t "teste/temperatura" -m "25.8"
```
Resultado esperado:
```text
teste/temperatura 25.8
```

## Montagem do sensor DS18B20

//

(imagem)

//

### Componentes utilizados

Para realizar a montagem do circuito, foram utilizados:

- 1 Raspberry Pi 3;
- 1 sensor de temperatura DS18B20;
- 1 protoboard;
- 1 resistor de 4,7 kΩ;
- 3 cabos jumper macho-fêmea:
  - 1 preto;
  - 1 vermelho;
  - 1 amarelo.

### Identificação dos fios

Na montagem realizada, os fios do sensor DS18B20 possuem as seguintes funções:

- **Preto:** GND ou aterramento;
- **Vermelho:** VCC ou alimentação de 3,3 V;
- **Amarelo:** DATA ou transmissão dos dados de temperatura.

> [!WARNING]
> As cores dos fios podem variar conforme o fabricante do sensor. Verifique a identificação dos fios antes de conectar a alimentação.

### Resumo das conexões

| Componente | Função | Conexão no Raspberry Pi |
|---|---|---|
| Fio preto | GND | Pino físico 6 |
| Fio vermelho | Alimentação de 3,3 V | Pino físico 1 |
| Fio amarelo | Sinal de dados | GPIO 4, pino físico 7 |
| Resistor de 4,7 kΩ | Pull-up | Entre 3,3 V e a linha de dados |

### Montagem na protoboard

Os fios preto, vermelho e amarelo do sensor devem ser inseridos em trilhas elétricas diferentes da protoboard.

Os furos pertencentes à mesma trilha são eletricamente conectados. Por isso, os componentes não precisam ocupar exatamente os mesmos furos mostrados na imagem, mas precisam compartilhar as trilhas corretas.

#### 1. Conexão do aterramento

1. Insira o **fio preto do sensor** em uma trilha da protoboard;
2. Insira o **jumper preto** em outro furo da mesma trilha;
3. Conecte a extremidade fêmea do jumper a um pino **GND** do Raspberry Pi.

#### 2. Conexão da alimentação

1. Insira o **fio vermelho do sensor** em outra trilha da protoboard;
2. Insira o **jumper vermelho** em outro furo da mesma trilha;
3. Conecte a extremidade fêmea do jumper ao pino de **3,3 V** do Raspberry Pi.

#### 3. Conexão do sinal de dados

1. Insira o **fio amarelo do sensor** em uma terceira trilha da protoboard;
2. Insira o **jumper amarelo** em outro furo da mesma trilha;
3. Conecte a extremidade fêmea do jumper ao **GPIO 4**, correspondente ao pino físico 7 do Raspberry Pi.

O GPIO 4 será utilizado pela interface 1-Wire para receber os dados de temperatura enviados pelo sensor.

#### 4. Instalação do resistor

Instale o resistor de **4,7 kΩ** entre as trilhas dos fios vermelho e amarelo:

1. Conecte uma extremidade do resistor à mesma trilha do **fio vermelho**, ligada aos 3,3 V;
2. Conecte a outra extremidade à mesma trilha do **fio amarelo**, ligada ao GPIO 4.

O resistor funciona como um resistor de **pull-up**, mantendo a linha de dados em um nível lógico estável para a comunicação entre o sensor e o Raspberry Pi.

### Diagrama simplificado

```text
Raspberry Pi                         Sensor DS18B20

3,3 V, pino físico 1 --------------- VCC, fio vermelho
          |
          +---- resistor 4,7 kΩ -----+
                                     |
GPIO 4, pino físico 7 -------------- DATA, fio amarelo

GND, pino físico 6 ----------------- GND, fio preto
```