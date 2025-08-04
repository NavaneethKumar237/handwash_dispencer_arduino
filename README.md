# handwash_dispencer_arduino
In the wake of growing hygiene awareness, this project presents a smart, contactless hand wash dispenser designed to minimize germ transmission through automation. The system uses an ultrasonic sensor to detect hand presence, automatically activates a sanitizer dispenser, and follows up with a short UV sterilization cycle to clean the dispensing area — all without requiring physical contact.
This project prioritizes both user convenience and safety, using a time-controlled relay system, visual feedback via an I2C LCD, and a buzzer alert to ensure intuitive operation. The system eliminates the need for manual buttons and maintains hygienic conditions by reducing touch points.


# V1
```
#include <Wire.h>
#include <LiquidCrystal_I2C.h>

#define TRIG_PIN 9
#define ECHO_PIN 10

#define RELAY_SANITIZER 6
#define RELAY_UV        7
#define BUZZER_PIN      8

const int DISPENSE_TIME = 5;  // Sanitizer time (seconds)
const int UV_DELAY_TIME = 3;  // Delay before UV (seconds)
const int UV_TIME = 5;        // UV on time (seconds)

LiquidCrystal_I2C lcd(0x27, 16, 2); // I2C LCD (address, cols, rows)

void setup() {
  Serial.begin(9600);
  lcd.init();
  lcd.backlight();

  pinMode(TRIG_PIN, OUTPUT);
  pinMode(ECHO_PIN, INPUT);

  pinMode(RELAY_SANITIZER, OUTPUT);
  pinMode(RELAY_UV, OUTPUT);
  pinMode(BUZZER_PIN, OUTPUT);

  digitalWrite(RELAY_SANITIZER, LOW);
  digitalWrite(RELAY_UV, LOW);

  lcd.setCursor(0, 0);
  lcd.print("Auto Dispenser");
  lcd.setCursor(0, 1);
  lcd.print("Place your hand");
}

void loop() {
  if (detectHand()) {
    tone(BUZZER_PIN, 1000, 200); // Beep on detection
    runSanitizer();
    delay(UV_DELAY_TIME * 1000);
    runUV();
    
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Place your hand");
  }
}

bool detectHand() {
  long duration;
  int distance;

  digitalWrite(TRIG_PIN, LOW);
  delayMicroseconds(2);
  digitalWrite(TRIG_PIN, HIGH);
  delayMicroseconds(10);
  digitalWrite(TRIG_PIN, LOW);

  duration = pulseIn(ECHO_PIN, HIGH);
  distance = duration * 0.034 / 2;

  Serial.print("Distance: ");
  Serial.println(distance);

  if (distance > 0 && distance <= 10) {
    delay(200); // Debounce
    return true;
  }

  return false;
}

void runSanitizer() {
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Dispensing...");
  digitalWrite(RELAY_SANITIZER, HIGH);

  for (int i = DISPENSE_TIME; i > 0; i--) {
    lcd.setCursor(0, 1);
    lcd.print("Sanitize: ");
    lcd.print(i);
    lcd.print("s   ");
    delay(1000);
  }

  digitalWrite(RELAY_SANITIZER, LOW);
}

void runUV() {
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("UV Sterilizing...");
  digitalWrite(RELAY_UV, HIGH);

  for (int i = UV_TIME; i > 0; i--) {
    lcd.setCursor(0, 1);
    lcd.print("UV Time: ");
    lcd.print(i);
    lcd.print("s   ");
    delay(1000);
  }

  digitalWrite(RELAY_UV, LOW);
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Thank you!");
  delay(2000);
}
```
