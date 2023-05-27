# Projeto de Alimentador para Cães ou Gatos

Esse trabalho aborda o desenvolvimento de um alimentador automático para animais domésticos utilizando ESP-32 com RTC.
Autores: Samuel Barros Souza   (RA:20.01044-3);
         Lucas Granja Bernardo (RA:19.00305-6);

# 🚀 Introdução

A proposta desse projeto é o desenvolvimento de um sistema automático para liberação de comida em horários determinados via código. Como também, é a implementação de um botão que permita que o usuário despeje a comida para seu animal de estimação quando desejar. 

# 💻 Componentes Eletrônicos e Esquema Elétrico 

O software utilizado para o desenvolvimento do esquemático foi o Altium, e é possível observar na figura abaixo as ligações.

![image](https://github.com/samuellbs/Alimentador_PET/assets/103770785/8689b729-66f0-4c9b-b2c6-f63c6623cd45)

A placa final desenvolvida não utilizou um diodo em paralelo com o motor, mas é recomendável utilizar para evitar a tensão reversa no transistor. 

A tabela abaixo indica os componentes utilizados, bem como os preços deles (25/05/2023) como forma de realizar um levantamento de custo.

![image](https://github.com/samuellbs/Alimentador_PET/assets/103770785/cd5911a6-a360-4e6c-b6f2-1e72f2a021fb)


#  ⚙️ Funcionamento do sistema

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

# 📄 Bibliotecas Diplay OLED I2C

Inicialmente, na IDE Arduino, selecione Ferramentas --> Gerenciar Bibliotecas. Logo depois, digite SSD1306 na barra de pesquisa e procure pela biblioteca SSD1306 do Adafruit e instale. Por fim, repita o processo e instale também as bibliotecas GFX e BusIO, ambas do Adafruit.

# 📄 Biblioteca Real Time Clock (RTC)

Acesse o link https://github.com/adafruit/RTClib e baixe o zip da biblioteca. Logo após, clique em Sketch --> Incluir Biblioteca --> Adicionar Biblioteca .zip e selecione o arquivo baixado.

# ⌨️ Software Versão 0

```
/*    Programa: Alimentador Automático PET 
 *    Autores: Samuel Barros Souza   (RA: 20.01044-3)
 *             Lucas Granja Bernardo (RA: 19.00305-6)
 *             
 *    Versão: 0
 *    Concluído em: 10/05/2023
 *    Breve Descrição:
 *        Este programa realiza a liberação de ração para animais
 *        domésticos, utilizando horário real de Brasília ou pela 
 *        definição do usuário.
 */
 
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>
#include "RTClib.h"
#include <Fonts/FreeSerif9pt7b.h>


#define SCREEN_WIDTH 128 // OLED display width, in pixels
#define SCREEN_HEIGHT 64 // OLED display height, in pixels
// Declaration for an SSD1306 display connected to I2C (SDA, SCL pins)
#define OLED_RESET     -1 // Reset pin # (or -1 if sharing Arduino resetpin)
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, OLED_RESET);


#define button_pin 18       // pino do botão
#define motor_pin_one 32   // pino 1 do motor
#define motor_pin_two 33   // pino 2 do motor



//RTC_DS1307 rtc; //RTC da classe DS1307
RTC_DS3231 rtc; //RTC da classe DS3132
  


int leituraButton;
int hora, minuto, segundo;
char strBuf[50];         // para montar a string com duas casas decimais



void setup()
{
  Serial.begin(115200);
  
  pinMode(motor_pin_one, OUTPUT);
  pinMode(motor_pin_two, OUTPUT);
  
  rtc.begin();
  //rtc.adjust(DateTime(F(__DATE__), F(__TIME__)));  // Atualiza o horário automaticamente, necessário na primeira compilação e depois comentar
  
  if (! rtc.begin()) {                         //Se o RTC nao for inicializado, faz
    Serial.println("RTC NAO INICIALIZADO");    //Imprime o texto
    while (1);                                 //Trava o programa
  }
  delay(100);                                  
  
  //rtc.adjust(DateTime(2023, 4, 19, 20, 59, 00));//Ajusta o tempo do RTC para a data e hora definida pelo usuario.
  
  if (!display.begin(SSD1306_SWITCHCAPVCC, 0x3C)) {     // Identificar o Display
    Serial.println(F("SSD1306 allocation failed"));
    for (;;); // Caso não encontre, loop infinito
  }
  
}

void loop() {

 
 OLED_display(); 


}


void OLED_display () {


/*      Descrição de funcionamento:
 *       
 *          Essa é a função principal do código e é responsável por 
 *          escrever no display o horário e monitorar as funções do
 *          botão e do tempo.
 */
   
  DateTime agora = rtc.now();    
  hora = agora.hour();        // definição da hora atual
  minuto = agora.minute();    // definição do minuto atual
  segundo = agora.second();   // definição do segundo atual

  
  digitalWrite(motor_pin_one, LOW);   // motor desligado
  digitalWrite(motor_pin_two,LOW);   // motor desligado
  
  display.setFont(&FreeSerif9pt7b);   // definição da fonte de letra para display
  display.clearDisplay();
  // Display Text
  display.setTextSize(1);             // definição tamanho de letra para display
  display.setTextColor(WHITE);        // definição de cor de letra para display
  display.setCursor(0, 24);
  display.println("Horario");
  sprintf(strBuf, "%02d : %02d : %02d\n", hora, minuto, segundo);  // imprime no display "Horario: hora : minuto : segundo")
  display.println(strBuf);
  Serial.println(strBuf);
//  display.print(hora, DEC);
//  display.print(" : ");
//  display.print(minuto);
//  display.print(" :  ");
//  display.println(segundo);
  display.display();
  display.clearDisplay();
  
  trata_botao();          // monitoramento da função do botão  
  trata_tempo();          //monitoramento da função do tempo
}



void OLED_event () {


/*      Descrição de funcionamento: 
 *        
 *          Essa função é responsável por realizar a mudança de funcionamento do sistema
 *          quando há eventos importantes (botão HIGH ou tempo HIGH). Dessa maneira, ela
 *          controla o motor para despejar ração quando necessário.
 *  
 */

  display.setFont(&FreeSerif9pt7b);   // fonte de letra do display
  display.clearDisplay();
  display.setTextSize(1);             // tamanho de letra do display
  display.setTextColor(WHITE);        // cor de letra do display
  display.setCursor(0, 24);
  display.println("Hora da comida");
  display.display();
  trata_motor();                      // monitoramento da função do motor
  display.clearDisplay();

}


void trata_tempo() {


/*      Descrição de funcionamento: 
 *        
 *          Essa função é responsável por monitorar a igualdade do horário real com as
 *          definições abaixo para despejar a ração.
 *          Os horários de 07:30 e 19:30 foram definidos pelos autores do projeto e 
 *          são totalmente arbitrários.
 *  
 */
  if (hora == 07 && minuto == 30 && segundo == 00){

    OLED_event();
    delay(10000);
   
    
  }

  else if (hora == 19 && minuto == 30 && segundo == 00){

    OLED_event();
    delay(10000);
   
  }
}
/*
  else if (hora == 17 && minuto == 00 && segundo == 0) {

    OLED_event();
    delay(10000);
    
  }

else if (hora == 21 && minuto == 00 && segundo == 0){

    OLED_event();
    delay(10000);
    
  }
  digitalWrite(motor_pin,LOW);
}
*/


void trata_botao() {


/*      Descrição de funcionamento: 
 *        
 *          Essa função é responsável por monitorar o acionamento do botão e acionar o motor para despejar ração.
 *  
 */
 
   leituraButton = digitalRead(button_pin);  // variável para leitura (HIGH ou LOW) do botão 
   
  if (leituraButton == HIGH){

    OLED_event();
    delay(10000);
  
  }
  
  leituraButton = 0;
}

void trata_motor() {

/*      Descrição de funcionamento: 
 *        
 *          Essa função é responsável por acionar o motor quando desejado, conforme 
 *          os eventos de tempo ou botão.
 *          
 *  
 */
 
    digitalWrite(motor_pin_one, HIGH);
    digitalWrite(motor_pin_two, LOW);         // motor no sentido anti-horário (10)
    delay(3000);
   
    digitalWrite(motor_pin_one, LOW);
    digitalWrite(motor_pin_two, HIGH);      // motor no sentido horário (01)
    delay(2000);
    
    digitalWrite(motor_pin_one, HIGH);
    digitalWrite(motor_pin_two, LOW);       // motor no sentido anti-horário (10)
    delay(3000);
    
    
    digitalWrite(motor_pin_one, LOW);
    digitalWrite(motor_pin_two, LOW);     // motor desligado (00 ou 11)
  }
```

