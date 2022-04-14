# T800
---
Um filtro de pacotes inteligente para dispositivos embarcados que utilizam a Stack TCP/IPv4 LwIP.

## Toolchain necessária
- Python3
- Framework ESP-IDF feito pela empresa Espressif
- *iperf v2*
- *NMap*

## Como utilizar T800

Para nossos experimentos, o dispositivo utilizado foi a ESP32. É válido ressaltar que as portas TCP 6767, 6768 e 5001 não podem estar sendo utilizadas pelo Sistema Operacional durante a execução dos experimentos.

Ademais, é necessário rodar qualquer aplicação no ESP-IDF que se conecte ao Wi-Fi para descobrir o IP da ESP32 na sua rede. Logo em seguida, apenas é necessário dar *build* e *flash* no projeto na placa:

```bash
get_idf
cp -r ./t800 $IDF_PATH/components/
cp -r ./tflite-lib $IDF_PATH/components/
cp -r ./esp-nn $IDF_PATH/components/
cp lwip/CMakeLists.txt $IDF_PATH/components/lwip/CMakeLists.txt
cp lwip/lwip/src/core/ipv4/ip4.c $IDF_PATH/components/lwip/lwip/src/core/ipv4/ip4.c
cp lwip/lwip/src/include/lwip/ip4.h $IDF_PATH/components/lwip/lwip/src/include/lwip/ip4.h
cd ./t800_experiment
idf.py -p <YOUR_ESP32_PORT> flash
```

Finalmente, o experimento pode acontecer:
1. rode `sudo python attacker.py` em uma janela de terminal
2. rode `idf.py -p <YOUR_ESP32_PORT> monitor` em outra janela de terminal, em paralelo.

Obs: Durante o experimento o pino D5 será 0 quando o experimento não estiver rodando e 1 durante sua execução.

Depois de realizar o experimento, um  arquivo `data.csv` será gerado com todos os dados coletados durante o experimento.

## Como o T800 funciona?
O T800 necessita ser previamente configurado na própria aplicação em que se deseja utilizá-lo. Para tal, é necessário alocar uma *struct* do tipo `t800_config_t` com o modo de utilização do filtro e a função de classificação de pacotes. O modo de utilização pode ser do tipo `SELECTED` ou `UNINITIALIZED`. Já a função de classificação é dada por algum modelo do Tensorflow, convertido para o Tensorflow Lite e carregada previamente para a ESP32. É possível utilizar até mesmo um modelo mais simples com estruturas condicionais.

Em seguida, apenas é necessário chamar a função `init_t800()`, passando como argumento a *struct* de configuração. Essa função, é responsável por inicializar definitivamente o sistema do T800. 

Dessa forma, como o T800 se comunica diretamente com a stack TCP/IPv4 do LwIP, a partir do momento que pacotes começarem a chegar no dispositivo(ESP32), a função `run_t800()` é invocada. Tal função recebe como parâmetros os cabeçalhos IP e TCP do pacote que chegou na placa. Essa função, tem como propósito verificar o modo de funcionamento do T800 naquele momento e invocar a função de classificação já previamente selecionada.

## Como obter métricas computacionais?

Para nossos experimentos, foi utilizado um servidor UDP feito em python que recebe os dados da ESP32 de tempos em tempos (em nossos experimentos, foi estabelecido um tempo de 1 segundo).

É válido ressaltar que a coleta de dados é feita em uma *thread* separada da execução dos experimentos, garantindo, assim, a paralelização do experimento e da coleta dos dados.

## Como gerar um tráfego de rede na ESP32?

### Tráfego benigno
Para isso, o T800 possui um *cliente iperf* implementado de forma independente do sistema computacional do usuário que recebe os experimentos, no *servidor iperf*. Tal fato, garante que os dados coletados pelo * cliente iperf* são confiáveis e representam exatamente o que foi causado pelo experimento.

Para iniciar o * cliente iperf* na ESP32, é necessário chamar a função `iperf_tcp_server()` com um único parâmetro: o socket UDP. Dentro dessa função, o *cliente iperf* será configurado e os pacotes do *servidor iperf* serão recebidos.

Para configurar o *cliente iperf* é necessário apenas configurar o socket TCP que estabelecerá a conexão com o *servidor iperf*. Tal fato é realizado com a invocação da função `iperf_setup_tcp_server()`, passando como parâmetro uma *struct* `sockaddr_in` que servirá com o propósito de configurar o socket do *cliente iperf*.

É válido ressaltar que enquando o *cliente iperf* está rodando, outra *thread* está responsável em coletar os dados dos recursos computacionais da ESP32.

### Tráfego Malicioso

Para o controle de tráfego malicioso, dentro do script em python, que possui toda a lógica de recebimento de dados e a comunicação com o socket do *cliente iperf* da ESP32, também é executado o comando *NMap*.

O *NMap* roda conjuntamente com o *servidor iperf*, no lado do atacante(usuário que deseja receber as métricas da placa), e com o auxílio de algumas flags é possível controlar seu tráfego, como por exemplo a flag `-T`.