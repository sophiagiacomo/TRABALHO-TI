# TRABALHO-TI
#include <Arduino.h>

#define LATCH 4 #define CLK 7 #define DATA 8

#define DIG1 5 #define DIG2 6 #define DIG3 9 #define DIG4 10

#define LED1 13 #define LED2 12 #define LED3 11 #define LED4 2

#define S1 A1 #define S2 A2

#define BUZZER 3

const byte NUM[] = { 0x3F,0x06,0x5B,0x4F,0x66, 0x6D,0x7D,0x07,0x7F,0x6F };

#define sO 0x3F #define sF 0x71 #define sX 0x00

byte senha[4]; byte posSenha = 0; byte numeroAtual = 0;

int tempo = 90;

byte disp[4] = {0,0,0,0};

bool acabou = false;

unsigned long tTick = 0; unsigned long tMux = 0; unsigned long tBtn1 = 0; unsigned long tBtn2 = 0;

byte digAtual = 0;

void ligarBuzzer() { digitalWrite(BUZZER, LOW); }

void desligarBuzzer() { digitalWrite(BUZZER, HIGH); }

void enviarDisplay(byte seg, byte dig) { digitalWrite(DIG1, LOW); digitalWrite(DIG2, LOW); digitalWrite(DIG3, LOW); digitalWrite(DIG4, LOW);

digitalWrite(LATCH, LOW); shiftOut(DATA, CLK, MSBFIRST, seg); digitalWrite(LATCH, HIGH);

byte pinos[] = {DIG1, DIG2, DIG3, DIG4}; digitalWrite(pinos[dig], HIGH); }

void mostrarNumero(int n) { disp[3] = (n >= 1000) ? NUM[n/1000] : sX; disp[2] = (n >= 100) ? NUM[(n/100)%10] : sX; disp[1] = (n >= 10) ? NUM[(n/10)%10] : sX; disp[0] = NUM[n%10]; }

void mostrarOFF() { disp[3] = sX; disp[2] = sO; disp[1] = sF; disp[0] = sF; }

void atualizarLEDs() { digitalWrite(LED1, posSenha & 1); digitalWrite(LED2, posSenha & 2); digitalWrite(LED3, posSenha & 4); digitalWrite(LED4, posSenha & 8); }

void setup() { pinMode(LATCH, OUTPUT); pinMode(CLK, OUTPUT); pinMode(DATA, OUTPUT);

pinMode(DIG1, OUTPUT); pinMode(DIG2, OUTPUT); pinMode(DIG3, OUTPUT); pinMode(DIG4, OUTPUT);

pinMode(LED1, OUTPUT); pinMode(LED2, OUTPUT); pinMode(LED3, OUTPUT); pinMode(LED4, OUTPUT);

pinMode(S1, INPUT_PULLUP); pinMode(S2, INPUT_PULLUP);

pinMode(BUZZER, OUTPUT);

desligarBuzzer();

randomSeed(analogRead(A5));

for (int i = 0; i < 4; i++) { senha[i] = random(0, 10); }

mostrarNumero(tempo); tTick = millis(); }

void loop() { unsigned long agora = millis();

if (agora - tMux >= 5) { tMux = agora; enviarDisplay(disp[digAtual], digAtual);

digAtual++;
if (digAtual > 3) digAtual = 0;
}

if (acabou) return;

if (agora - tTick >= 1000) { tTick = agora; tempo--;

mostrarNumero(tempo);

if (tempo <= 0) {
  acabou = true;
  mostrarNumero(0);
  ligarBuzzer();
}
}

if (digitalRead(S1) == LOW && agora - tBtn1 > 250) { tBtn1 = agora;

numeroAtual++;
if (numeroAtual > 9) numeroAtual = 0;

mostrarNumero(numeroAtual);
}

if (digitalRead(S2) == LOW && agora - tBtn2 > 250) { tBtn2 = agora;

if (numeroAtual == senha[posSenha]) {
  posSenha++;
  atualizarLEDs();

  if (posSenha >= 4) {
    acabou = true;
    mostrarOFF();

    for (int i = 0; i < 6; i++) {
      ligarBuzzer();
      delay(100);
      desligarBuzzer();
      delay(100);
    }
  }
}
else {
  posSenha = 0;
  atualizarLEDs();

  ligarBuzzer();
  delay(300);
  desligarBuzzer();
}
} }
