# Monitoramento de Temperatura com Raspberry Pi e MQTT
//

(imagens)

//

Projeto de monitoramento de temperatura utilizando Raspberry Pi, sensores DS18B20 e o protocolo MQTT.

A solução foi desenvolvida como uma prova de conceito para monitorar a temperatura próxima aos aparelhos de ar-condicionado em um ambiente crítico (por exemplo, uma sala de equipamentos ou data center), onde o controle de temperatura é sensivel.

> Este projeto realiza apenas a **coleta e a transmissão** das temperaturas. Ele **não controla** os aparelhos de ar-condicionado.

## Sumário
- [Objetivo](#objetivo)
- [Arquitetura](#Arquitetura)
- [Raspberry Pi Publisher](#Raspberry-Pi-Publisher)
- [Raspberry Pi Broker](#Raspberry-Pi-Broker)
- [Fluxo de comunicação](#Fluxo-de-comunicação)
- [Tecnologias utilizadas](#Tecnologias-utilizadas)
- [Materiais](#Materiais)
- [Preparação dos cartões MicroSD](#Preparação-dos-cartões-MicroSD)
- [Inicialização dos Raspberry Pi](#Inicialização-dos-Raspberry-Pi)
- [Preparação dos Raspberry Pi](#Preparação-dos-Raspberry-Pi)
- [Configuração do Broker MQTT](#Configuração-do-Broker-MQTT)
- [Teste do MQTT](#Teste-do-MQTT)
- [Montagem do sensor DS18B20](#Montagem-do-sensor-DS18B20)
- [Ativação da interface 1-Wire](#Ativação-da-interface-1-Wire)
- [Verificação do sensor](#Verificação-do-sensor)
- [Preparação do ambiente Python](#Preparação-do-ambiente-Python)
- [Teste em Python com temperatura simulada](#Teste-em-Python-com-temperatura-simulada)
- [Publicação da temperatura real](#Publicação-da-temperatura-real)
- [Organização dos tópicos](#Organização-dos-tópicos)
- [Monitoramento das mensagens no Broker](#Monitoramento-das-mensagens-no-Broker)
- [Utilização de mais de um sensor](#Utilização-de-mais-de-um-sensor)

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

- Ler os sensores DS18B20 (neste projeto, os dois sensores DS18B20 estão conectados ao mesmo Raspberry Pi Publisher, compartilhando a mesma linha 1-Wire - );
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
- Eclipse Mosquitto;O projeto cobre a coleta e o transporte das leituras de temperatura via MQTT. Recursos como autenticação, criptografia, alertas e persistência
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

>Defina usuário, senha e credenciais de Wi-Fi próprios. Nenhuma credencial real deve constar neste README ou em arquivos versionados do projeto.

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
>Os endereços IP variam conforme a rede local de cada ambiente. Use o IP retornado pelo comando `hostname -I` no seu próprio ambiente — não há um endereço fixo válido para todos os casos.

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
>[!WARNING] `allow_anonymous true` é uma configuração exclusiva de laboratório/desenvolvimento. Ela permite que qualquer dispositivo na rede publique e assine tópicos sem autenticação. Em uma implantação real (fora de um ambiente de testes isolado), é obrigatório configurar:
> - Autenticação por usuário e senha (`password_file`) ou certificados;
> - Controle de acesso por tópico (`acl_file`);
> - Criptografia via TLS (porta 8883);
> - Restrição de rede/firewall para o Broker.
>
> Não utilize `allow_anonymous` true em redes compartilhadas, expostas à internet ou em produção.

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

> [!WARNING] As cores dos fios podem variar conforme o fabricante do sensor.
> Verifique a identificação dos fios antes de conectar a alimentação.

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

O DS18B20 utiliza o barramento 1-Wire. A biblioteca w1thermsensor documenta o GPIO 4 como pino de dados padrão e o resistor entre a alimentação e a linha DATA.

## Ativação da interface 1-Wire

Depois de revisar a montagem do circuito, ligue o Raspberry Pi Publisher e abra o **Terminal**.

Execute:

```bash
sudo raspi-config
```

No menu de configuração, acesse:

```text
Interface Options > 1-Wire > Yes
```

Finalize a configuração e reinicie o Raspberry Pi:

```bash
sudo reboot
```

A interface 1-Wire permite que o Raspberry Pi reconheça o sensor DS18B20. Nesta montagem, o sinal de dados utiliza o **GPIO 4**, correspondente ao pino físico 7.

## Verificação do sensor

Após a reinicialização, verifique os dispositivos conectados ao barramento 1-Wire:

```bash
ls /sys/bus/w1/devices/
```

O sensor DS18B20 deverá aparecer com um identificador iniciado por `28-`.

Exemplo:

```text
28-00000abcdef1
```

Cada sensor possui um identificador próprio, utilizado posteriormente para diferenciar as medições.

> [!NOTE]
> Um diretório iniciado por `00-` pode indicar problema na montagem, conexão incorreta ou ausência do resistor de pull-up.

Para visualizar os dados brutos enviados pelo sensor:

```bash
cat /sys/bus/w1/devices/28-*/w1_slave
```

Dependendo da versão do Raspberry Pi OS, também poderá existir o arquivo `temperature`:

```bash
cat /sys/bus/w1/devices/28-*/temperature
```

O valor poderá ser apresentado em milésimos de grau Celsius. Por exemplo:

```text
25125
```

Esse valor corresponde a:

```text
25,125 °C
```

Se nenhum sensor for encontrado, verifique:

- Se a interface 1-Wire está ativada;
- Se o fio preto está conectado ao GND;
- Se o fio vermelho está conectado aos 3,3 V;
- Se o fio amarelo está conectado ao GPIO 4;
- Se o resistor de 4,7 kΩ está entre os 3,3 V e a linha de dados;
- Se o sensor está recebendo alimentação.

## Preparação do ambiente Python

No Raspberry Pi Publisher, instale o Python, o `pip` e o suporte a ambientes virtuais:

```bash
sudo apt update
sudo apt install python3 python3-pip python3-venv -y
```

Crie uma pasta para o projeto:

```bash
mkdir -p ~/projeto-temperatura
cd ~/projeto-temperatura
```

Crie um ambiente virtual:

```bash
python3 -m venv .venv
```

Ative o ambiente:

```bash
source .venv/bin/activate
```

Instale as bibliotecas utilizadas pelo projeto:

```bash
python3 -m pip install --upgrade pip
python3 -m pip install paho-mqtt w1thermsensor
```

O ambiente virtual mantém as dependências do projeto separadas dos pacotes do sistema operacional.

Sempre que abrir um novo Terminal para executar o projeto, acesse a pasta e ative novamente o ambiente:

```bash
cd ~/projeto-temperatura
source .venv/bin/activate
```

> [!NOTE]
> Não é necessário utilizar `--break-system-packages`, pois as bibliotecas serão instaladas dentro do ambiente virtual.

## Teste em Python com temperatura simulada

Antes de utilizar a temperatura real do sensor, faça um teste publicando um valor simulado no Broker MQTT.

Crie o arquivo:

```bash
nano teste_mqtt.py
```

Adicione o seguinte código:

```python
import paho.mqtt.client as mqtt

BROKER = "IP_DO_BROKER"
PORTA = 1883
TOPICO = "temperatura/teste"
TEMPERATURA_SIMULADA = 25.8

cliente = mqtt.Client(
    mqtt.CallbackAPIVersion.VERSION2,
    client_id="publisher-teste"
)

try:
    cliente.connect(BROKER, PORTA, 60)
    cliente.loop_start()

    resultado = cliente.publish(
        TOPICO,
        payload=str(TEMPERATURA_SIMULADA),
        qos=0,
        retain=False
    )

    resultado.wait_for_publish()

    print(
        f"Temperatura simulada enviada: "
        f"{TEMPERATURA_SIMULADA} °C"
    )

finally:
    cliente.loop_stop()
    cliente.disconnect()
```

Substitua:

```python
BROKER = "IP_DO_BROKER"
```

Pelo endereço IP do Raspberry Pi que executa o Mosquitto.

Exemplo:

```python
BROKER = "192.168.1.20"
```

Salve o arquivo e execute:

```bash
python3 teste_mqtt.py
```

No Raspberry Pi Broker, acompanhe a mensagem publicada:

```bash
mosquitto_sub \
  -h IP_DO_BROKER \
  -p 1883 \
  -t "temperatura/#" \
  -v
```

Resultado esperado:

```text
temperatura/teste 25.8
```

Se a mensagem for recebida, a aplicação Python conseguiu se conectar e publicar no Broker MQTT.

## Publicação da temperatura real

Depois de validar a comunicação MQTT com a temperatura simulada, crie o programa responsável pela leitura dos sensores DS18B20.

Crie o arquivo:

```bash
nano publisher_temperatura.py
```

Adicione o seguinte código:

```python
import time

import paho.mqtt.client as mqtt
from w1thermsensor import W1ThermSensor

BROKER = "IP_DO_BROKER"
PORTA = 1883
INTERVALO = 2

cliente = mqtt.Client(
    mqtt.CallbackAPIVersion.VERSION2,
    client_id="raspberry-publisher"
)

cliente.connect(BROKER, PORTA, 60)
cliente.loop_start()

try:
    while True:
        sensores = W1ThermSensor.get_available_sensors()

        if not sensores:
            print("Nenhum sensor DS18B20 encontrado.")

        for sensor in sensores:
            temperatura = sensor.get_temperature()
            topico = f"temperatura/{sensor.id}"

            resultado = cliente.publish(
                topico,
                payload=str(temperatura),
                qos=0,
                retain=False
            )

            resultado.wait_for_publish()

            print(
                f"Sensor: {sensor.id} | "
                f"Temperatura: {temperatura:.2f} °C | "
                f"Tópico: {topico}"
            )

        time.sleep(INTERVALO)

except KeyboardInterrupt:
    print("\nPublicação interrompida pelo usuário.")

finally:
    cliente.loop_stop()
    cliente.disconnect()
    print("Conexão com o Broker encerrada.")
```

Substitua:

```python
BROKER = "IP_DO_BROKER"
```

Pelo endereço IP do Raspberry Pi Broker.

Exemplo:

```python
BROKER = "192.168.1.20"
```

Execute o programa:

```bash
python3 publisher_temperatura.py
```

O programa seguirá este fluxo:

1. Conectará o Raspberry Pi Publisher ao Broker MQTT;
2. Procurará os sensores DS18B20 disponíveis;
3. Percorrerá os sensores encontrados;
4. Lerá a temperatura de cada sensor;
5. Criará um tópico usando o identificador do sensor;
6. Publicará a temperatura no tópico correspondente;
7. Repetirá o processo a cada dois segundos.

Para interromper o programa, pressione:

```text
Ctrl+C
```

## Organização dos tópicos

Cada sensor DS18B20 possui um identificador único. O programa utiliza esse identificador para criar automaticamente um tópico para cada sensor.

Formato utilizado:

```text
temperatura/{sensor_id}
```

Exemplos:

```text
temperatura/00000abcdef1
temperatura/00000abcdef2
```

No código, o tópico é criado por esta linha:

```python
topico = f"temperatura/{sensor.id}"
```

Dessa forma, não é necessário informar manualmente o ID de cada sensor.

## Monitoramento das mensagens no Broker

No Raspberry Pi Broker, execute:

```bash
mosquitto_sub \
  -h IP_DO_BROKER \
  -p 1883 \
  -t "temperatura/+" \
  -v
```

O caractere `+` representa um nível do tópico. Nesse caso, representa o identificador de qualquer sensor conectado.

Exemplo de mensagem recebida:

```text
temperatura/00000abcdef1 25.81
```

Para acompanhar também a temperatura simulada e qualquer outro tópico abaixo de `temperatura/`, utilize:

```bash
mosquitto_sub \
  -h IP_DO_BROKER \
  -p 1883 \
  -t "temperatura/#" \
  -v
```

O caractere `#` permite receber todos os tópicos existentes abaixo de `temperatura/`.

## Utilização de mais de um sensor

O DS18B20 utiliza o barramento 1-Wire e cada sensor possui um identificador individual. Por isso, mais de um sensor pode compartilhar as mesmas conexões:

- O VCC pode compartilhar a linha de 3,3 V;
- O GND pode compartilhar a linha de aterramento;
- O DATA pode compartilhar a linha conectada ao GPIO 4;
- Cada sensor continuará possuindo um identificador diferente;
- O programa percorrerá automaticamente os sensores detectados;
- Cada temperatura será publicada em um tópico próprio.

Exemplo com dois sensores:

```text
temperatura/00000abcdef1 25.81
temperatura/00000abcdef2 26.34
```

> [!CAUTION]
> Desligue o Raspberry Pi da fonte antes de conectar outro sensor ou modificar o circuito.