# projetovale084091
#include <RH_ASK.h>
#include <SPI.h>
#include <Wire.h>
#include <LiquidCrystal_I2C.h>

#define BUZZER 6

#define LED_PERIGO 5
#define LED_ATENCAO 11
#define LED_LIVRE 10

struct Pacote {
  uint8_t colisao;
  int16_t dist0;
  int16_t dist1;
  int16_t dist2;
};

Pacote dadosRecebidos;

RH_ASK driver(200, 2, 12);

LiquidCrystal_I2C lcd(0x27, 16, 2);

void setup() {
  Serial.begin(9600);

  pinMode(BUZZER, OUTPUT);

  pinMode(LED_PERIGO, OUTPUT);
  pinMode(LED_ATENCAO, OUTPUT);
  pinMode(LED_LIVRE, OUTPUT);

  digitalWrite(LED_PERIGO, LOW);
  digitalWrite(LED_ATENCAO, LOW);
  digitalWrite(LED_LIVRE, LOW);

  lcd.init();
  lcd.backlight();
  lcd.clear();

  if (driver.init()) {
    Serial.println("Receptor RF iniciado.");
    lcd.setCursor(0, 0);
    lcd.print("RF Iniciado");
  } else {
    Serial.println("Erro ao inicializar o modulo RF!");
    lcd.setCursor(0, 0);
    lcd.print("Erro no RF");
  }

  delay(1000);
  lcd.clear();
}

void loop() {

  uint8_t tamanhoMensagem = sizeof(dadosRecebidos);

  if (driver.recv((uint8_t *)&dadosRecebidos, &tamanhoMensagem)) {

    if (tamanhoMensagem == sizeof(Pacote)) {

      bool perigo =
        (dadosRecebidos.dist0 > 0 && dadosRecebidos.dist0 <= 50) ||
        (dadosRecebidos.dist1 > 0 && dadosRecebidos.dist1 <= 50) ||
        (dadosRecebidos.dist2 > 0 && dadosRecebidos.dist2 <= 50);

      bool atencao =
        (!perigo) &&
        (
          (dadosRecebidos.dist0 > 50 && dadosRecebidos.dist0 <= 100) ||
          (dadosRecebidos.dist1 > 50 && dadosRecebidos.dist1 <= 100) ||
          (dadosRecebidos.dist2 > 50 && dadosRecebidos.dist2 <= 100)
        );

      if (perigo) {

        digitalWrite(LED_PERIGO, HIGH);
        digitalWrite(LED_ATENCAO, LOW);
        digitalWrite(LED_LIVRE, LOW);

        tone(BUZZER, 2200);
        delay(120);
        noTone(BUZZER);
        delay(120);

        Serial.println("Colisao iminente!");

      } else if (atencao) {

        digitalWrite(LED_PERIGO, LOW);
        digitalWrite(LED_ATENCAO, HIGH);
        digitalWrite(LED_LIVRE, LOW);

        tone(BUZZER, 1800);
        delay(300);
        noTone(BUZZER);
        delay(300);

        Serial.println("Colisao a caminho.");

      } else {

        digitalWrite(LED_PERIGO, LOW);
        digitalWrite(LED_ATENCAO, LOW);
        digitalWrite(LED_LIVRE, HIGH);

        noTone(BUZZER);

        Serial.println("Sem colisao!");
      }

      Serial.print("Colisao: ");
      Serial.println(dadosRecebidos.colisao);

      Serial.print("Esquerda: ");
      Serial.print(dadosRecebidos.dist0);
      Serial.println(" cm");

      Serial.print("Centro: ");
      Serial.print(dadosRecebidos.dist1);
      Serial.println(" cm");

      Serial.print("Direita: ");
      Serial.print(dadosRecebidos.dist2);
      Serial.println(" cm");

      lcd.clear();

      lcd.setCursor(0, 0);
      lcd.print("E:");
      lcd.print(dadosRecebidos.dist0);
      lcd.print(" C:");
      lcd.print(dadosRecebidos.dist1);

      lcd.setCursor(0, 1);
      lcd.print("D:");
      lcd.print(dadosRecebidos.dist2);

      if (dadosRecebidos.colisao) {
        lcd.print(" !");
      } else {
        lcd.print(" OK");
      }
    }
  }
}
