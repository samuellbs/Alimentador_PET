# Projeto de Alimentador para Cães ou Gatos

Esse trabalho aborda o desenvolvimento de um alimentador automático para animais domésticos utilizando ESP-32 com RTC.
Autores: Samuel Barros Souza   (RA:20.01044-3);
         Lucas Granja Bernardo (RA:19.00305-6);

# 🚀 Introdução

A proposta desse projeto é o desenvolvimento de um sistema automático para liberação de comida em horários determinados via código. Como também, é a implementação de um botão que permita que o usuário despeje a comida para seu animal de estimação quando desejar. 

# Componentes Eletrônicos e Esquema Elétrico 

O software utilizado para o desenvolvimento do esquemático foi o Altium, e é possível observar na figura abaixo as ligações.

![image](https://github.com/samuellbs/Alimentador_PET/assets/103770785/8689b729-66f0-4c9b-b2c6-f63c6623cd45)

A placa final desenvolvida não utilizou um diodo em paralelo com o motor, mas é recomendável utilizar para evitar a tensão reversa no transistor. 

A tabela abaixo indica os componentes utilizados, bem como os preços deles (25/05/2023) como forma de realizar um levantamento de custo.

![image](https://github.com/samuellbs/Alimentador_PET/assets/103770785/cd5911a6-a360-4e6c-b6f2-1e72f2a021fb)


# Funcionamento do sistema

O projeto apresenta um display OLED I2C que indica o horário atual. Dessa forma, quando o botão é apertado, ocorrem duas situações simultâneas. A primeira é a saturação do transistor BC548 permitindo que o LED acenda. Como também, o pino D18 do ESP32 está "monitorando" o acionamento do botão, para que exiba no display "Hora da comida" e acione o motor para liberação da comida. É importante ressaltar que a proposta do botão é permitir que o usuário despeje a comida quando quiser.

Por outro lado, como a proposta do sistema é ser um alimentador pet automático, através do código é possível definir horários para que a comida seja despejada no pote. No caso do software original, estão definidos os horários 07:30:00 e 19:30:00. As imagens abaixo são dedicadas para o funcionamento real do projeto, incluindo a placa final soldada, estrutura física e exemplos do funcionamento descrito acima.

![image](https://github.com/samuellbs/Alimentador_PET/assets/103770785/db1bf6cf-6dea-44c6-ac03-a2b49627749c)

https://github.com/samuellbs/Alimentador_PET/assets/103770785/105a38ab-1e81-4e2a-8d41-1488f6c5eada

https://github.com/samuellbs/Alimentador_PET/assets/103770785/1b54e7d9-1f1e-4d01-a8e0-5c319d7db13d

É importante ressaltar que a estrutura física escolhida, não foi desenvolvida pelos autores desse projeto. A escolha foi por uma estrutura da UsinaINFO, que atendia as necessidades.

# Código do sistema

O software foi desenvolvido na Arduino IDE.

# 📋 Pré-requisitos para o Software

O código necessita da instalação de bibliotecas para o funcionamento de componentes do projeto.

# Bibliotecas Diplay OLED I2C

Inicialmente, na IDE Arduino, selecione Ferramentas --> Gerenciar Bibliotecas. Logo depois, digite SSD1306 na barra de pesquisa e procure pela biblioteca SSD1306 do Adafruit e instale. Por fim, repita o processo e instale também as bibliotecas GFX e BusIO, ambas do Adafruit.

# Biblioteca Real Time Clock (RTC)

Acesse o link https://github.com/adafruit/RTClib e baixe o zip da biblioteca. Logo após, clique em Sketch --> Incluir Biblioteca --> Adicionar Biblioteca .zip e selecione o arquivo baixado.
