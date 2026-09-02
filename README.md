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

```
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
