---
layout: post
title: "Projeto emissor/receptor infravermelho – Arduino"
modified:
categories: blog
excerpt: ""
tags: [Arduino]
image:
  feature:
date: 2014-11-12T15:39:55-04:00
---

O projeto consiste em criar um receptor que capte qualquer sinal IR e que seja capaz de reproduzi-lo ao pressionarmos um botão.

Adicionalmente o sinal captado será exibido no Serial Monitor da IDE do Arduino (caso o Serial Monitor esteja aberto em um PC devidamente conectado na porta serial do Arduino).

# Componentes necessários

* Arduino (utilizei o UNO mas pode ser qualquer outro, inclusive não será necessário alterar o código porque ele está apontando para portas que existem em todos os modelos de Arduino).
* Resistor de 10k omhs
* Receptor IR 

![Receptor IR]({{ site.url }}/images/projeto-receptor-emissor/modulo-receptor-ir-tsop1838.jpg)

* Emissor IR

<img alt="Emissor IR" src="{{ site.url }}/images/projeto-receptor-emissor/modulo-transmissor-ir-keyes-38khz.jpg" width="160">

* Push Button

<img alt="Push Button" src="{{ site.url }}/images/projeto-receptor-emissor/push_button.jpg" width="170">

## Observação:

Tanto o receptor quanto o emissor IR **não precisam** ser iguais aos que eu utilizei! O projeto funcionará com qualquer um, por exemplo, se você quiser comprar apenas o LED Emissor IR (que custa alguns centavos) e ajustá-lo no circuito com os devidos resistores (para que o LED não queime) você pode fazê-lo sem nenhum problema. Eu utilizei esses 2 componentes prontos (com resistores e lindos ledzinhos inclusos naquela plaquinha quadrada no qual o LED vem grudado) para ter menos trabalho. Porém, eles custaram 10 reais cada um.

# O circuito

Não sei fazer nenhum diagrama de circuito elétrico e, sinceramente, estou com preguiça de aprender, portanto aqui vão diversas fotos em ângulos diferentes mostrando como os componentes devem ser conectados no Arduino:

![]({{ site.url }}/images/projeto-receptor-emissor/1.jpg)

![]({{ site.url }}/images/projeto-receptor-emissor/2.jpg)

![]({{ site.url }}/images/projeto-receptor-emissor/3.jpg)

![]({{ site.url }}/images/projeto-receptor-emissor/4.jpg)

Em resumo:

* **Push Button** conectado na porta digital 2.
* **Emissor IR** conectado na porta digital 3.
* **Receptor IR** conectado na porta analógica A5.

Utilizei dois resistores pois eu não tinha um resistor de 10k ohms. Ainda assim, esses resistores que utilizei não são o suficiente e a corrente elétrica poderia danificar o meu push button. Mas como eu sou um cara que tem muitos push buttons pra substituir caso queimem, foi assim mesmo.

# A biblioteca IRremote

Antes de irmos direto para o código devemos incluir uma biblioteca na pasta libraries do Arduino. Nunca fez isso? Eu também nunca tinha feito. Funciona da seguinte forma:

Baixe a biblioteca IRremote através do GitHub do criador da IRremote (Ken Shirriff’s): [https://github.com/shirriff/Arduino-IRremote](https://github.com/shirriff/Arduino-IRremote)

Após baixar o zip descompacte-o dentro da pasta “libraries” da instalação de seu Arduino (na minha máquina está localizada em `C:\Program Files (x86)\Arduino\libraries`).

![]({{ site.url }}/images/projeto-receptor-emissor/pasta_irRemote.png)

**Importante:**
Feito isso, caso você tenha alguma janela da interface do Arduino aberta, feche-a e abra novamente. Toda vez que você abre o aplicativo do Arduino ele compila as bibliotecas localizadas na pasta “libraries” de sua instalação (por isso colocamos a biblioteca IRremote lá).
{:.notice}

**Importante²:**
Fazer apenas isso não foi o suficiente pra mim. Meu código (que está logo abaixo) simplesmente não compilava. Depois de algum tempo pesquisando eu descobri que o problema ocorria devido a um conflito com uma biblioteca existente nessa pasta “libraries” que vem junto na instalação da IDE do Arduino. Então, se você também tiver algum problema na compilação exclua a pasta “RobotIRremote” da pasta “libraries” e seja feliz.
{:.notice}

# O código
Prontinho pra ser copiado, compilado e enviado para o seu Arduino:

```c
/*
 * IRrecord: record and play back IR signals as a minimal 
 * An IR detector/demodulator must be connected to the input RECV_PIN.
 * An IR LED must be connected to the output PWM pin 3.
 * A button must be connected to the input BUTTON_PIN; this is the
 * send button.
 *
 * The logic is:
 * If the button is pressed, send the IR code.
 * If an IR code is received, record it.
 *
 * Example by Ken Shirriff (http://arcfn.com - 2009) and 
 * modified by Fabricio Rissetto (http://fabriciorissetto.com - 2014)
 */

#include 

int RECV_PIN = A5;
int BUTTON_PIN = 2;

IRrecv irrecv(RECV_PIN);
IRsend irsend;

decode_results results;

void setup()
{
  Serial.begin(9600);
  irrecv.enableIRIn(); // Start the receiver
  pinMode(BUTTON_PIN, INPUT);
}

// Storage for the recorded code
int codeType = -1; // The type of code
unsigned long codeValue; // The code value if not raw
unsigned int rawCodes[RAWBUF]; // The durations if raw
int codeLen; // The length of the code
int toggle = 0; // The RC5/6 toggle state

// Stores the code for later playback
// Most of this code is just logging
void storeCode(decode_results *results) {
  codeType = results->decode_type;
  int count = results->rawlen;
  if (codeType == UNKNOWN) {
    Serial.println("Received unknown code, saving as raw");
    codeLen = results->rawlen - 1;
    // To store raw codes:
    // Drop first value (gap)
    // Convert from ticks to microseconds
    // Tweak marks shorter, and spaces longer to cancel out IR receiver distortion
    for (int i = 1; i <= codeLen; i++) {       if (i % 2) {         // Mark         rawCodes[i - 1] = results->rawbuf[i]*USECPERTICK - MARK_EXCESS;
        Serial.print(" m");
      } 
      else {
        // Space
        rawCodes[i - 1] = results->rawbuf[i]*USECPERTICK + MARK_EXCESS;
        Serial.print(" s");
      }
      Serial.print(rawCodes[i - 1], DEC);
    }
    Serial.println("");
  }
  else {
    if (codeType == NEC) {
      Serial.print("Received NEC: ");
      if (results->value == REPEAT) {
        // Don't record a NEC repeat value as that's useless.
        Serial.println("repeat; ignoring.");
        return;
      }
    } 
    else if (codeType == SONY) {
      Serial.print("Received SONY: ");
    } 
    else if (codeType == RC5) {
      Serial.print("Received RC5: ");
    } 
    else if (codeType == RC6) {
      Serial.print("Received RC6: ");
    } 
    else {
      Serial.print("Unexpected codeType ");
      Serial.print(codeType, DEC);
      Serial.println("");
    }
    Serial.println(results->value, HEX);
    codeValue = results->value;
    codeLen = results->bits;
  }
}

void sendCode(int repeat) {
  if (codeType == NEC) {
    if (repeat) {
      irsend.sendNEC(REPEAT, codeLen);
      Serial.println("Sent NEC repeat");
    } 
    else {
      irsend.sendNEC(codeValue, codeLen);
      Serial.print("Sent NEC ");
      Serial.println(codeValue, HEX);
    }
  } 
  else if (codeType == SONY) {
    irsend.sendSony(codeValue, codeLen);
    Serial.print("Sent Sony ");
    Serial.println(codeValue, HEX);
  } 
  else if (codeType == RC5 || codeType == RC6) {
    if (!repeat) {
      // Flip the toggle bit for a new button press
      toggle = 1 - toggle;
    }
    // Put the toggle bit into the code to send
    codeValue = codeValue & ~(1 << (codeLen - 1));
    codeValue = codeValue | (toggle << (codeLen - 1));
    if (codeType == RC5) {
      Serial.print("Sent RC5 ");
      Serial.println(codeValue, HEX);
      irsend.sendRC5(codeValue, codeLen);
    } 
    else {
      irsend.sendRC6(codeValue, codeLen);
      Serial.print("Sent RC6 ");
      Serial.println(codeValue, HEX);
    }
  } 
  else {
    // Assume 38 KHz
    irsend.sendRaw(rawCodes, codeLen, 38);
    Serial.println("Sent raw");
  }
}

int lastButtonState;

void loop() {
  // If button pressed, send the code.
  int buttonState = digitalRead(BUTTON_PIN);
  if (lastButtonState == HIGH && buttonState == LOW) {
    Serial.println("Released");
    irrecv.enableIRIn(); // Re-enable receiver
  }

  if (buttonState) {
    Serial.println("Pressed, sending");    
    sendCode(lastButtonState == buttonState);
    delay(50); // Wait a bit between retransmissions
  } 
  else if (irrecv.decode(&results)) {
    storeCode(&results);
    irrecv.resume(); // resume receiver
  }
  lastButtonState = buttonState;
}
```

Abaixo um vídeo mostrando o circuito em funcionamento:

<iframe width="560" height="315" src="//www.youtube.com/embed/OCFCDeaP_hU" frameborder="0" allowfullscreen=""></iframe>