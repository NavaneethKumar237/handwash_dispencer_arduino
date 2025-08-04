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
# component pin out

| Component       | Arduino UNO Pin    |
| --------------- | ------------------ |
| Button 5 sec    | 2                  |
| Button 10 sec   | 3                  |
| Button 15 sec   | 4                  |
| Button 20 sec   | 5                  |
| Relay Sanitizer | 6                  |
| Relay UV        | 7                  |
| Buzzer          | 8                  |
| HC-SR04 Trigger | 9                  |
| HC-SR04 Echo    | 10                 |
| I2C LCD         | A4 (SDA), A5 (SCL) |
---------------------------------------------------------

# v2 
```
#include <Wire.h>
#include <LiquidCrystal_I2C.h>

#define TRIG_PIN 9
#define ECHO_PIN 10

#define RELAY_SANITIZER 6
#define RELAY_UV        7
#define BUZZER_PIN      8

#define BTN_10S 2
#define BTN_15S 3
#define BTN_20S 4
#define BTN_25S 5

const int UV_DELAY_TIME = 3;
const int UV_TIME = 5;

int selectedTime = 5;  // Default time

LiquidCrystal_I2C lcd(0x27, 16, 2);

void setup() {
  Serial.begin(9600);
  lcd.init();
  lcd.backlight();

  pinMode(TRIG_PIN, OUTPUT);
  pinMode(ECHO_PIN, INPUT);

  pinMode(RELAY_SANITIZER, OUTPUT);
  pinMode(RELAY_UV, OUTPUT);
  pinMode(BUZZER_PIN, OUTPUT);

  pinMode(BTN_10S, INPUT_PULLUP);
  pinMode(BTN_15S, INPUT_PULLUP);
  pinMode(BTN_20S, INPUT_PULLUP);
  pinMode(BTN_25S, INPUT_PULLUP);

  digitalWrite(RELAY_SANITIZER, LOW);
  digitalWrite(RELAY_UV, LOW);

  lcd.setCursor(0, 0);
  lcd.print("Auto Dispenser");
  lcd.setCursor(0, 1);
  lcd.print("Place your hand");
}

void loop() {
  checkButtons();  // Check if a longer time is requested

  if (detectHand()) {
    tone(BUZZER_PIN, 1000, 200);
    runSanitizer(selectedTime);
    delay(UV_DELAY_TIME * 1000);
    runUV();

    selectedTime = 5; // Reset to default after one use

    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Auto Dispenser");
    lcd.setCursor(0, 1);
    lcd.print("Place your hand");
  }
}

void checkButtons() {
  if (digitalRead(BTN_10S) == LOW) selectedTime = 10;
  if (digitalRead(BTN_15S) == LOW) selectedTime = 15;
  if (digitalRead(BTN_20S) == LOW) selectedTime = 20;
  if (digitalRead(BTN_25S) == LOW) selectedTime = 25;

  if (selectedTime > 5) {
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Timer Set: ");
    lcd.print(selectedTime);
    lcd.print("s");
    lcd.setCursor(0, 1);
    lcd.print("Place your hand");
    delay(300); // Small debounce
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

  if (distance > 0 && distance <= 10) {
    delay(200); // debounce
    return true;
  }

  return false;
}

void runSanitizer(int seconds) {
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Dispensing...");

  digitalWrite(RELAY_SANITIZER, HIGH);

  for (int i = seconds; i > 0; i--) {
    lcd.setCursor(0, 1);
    lcd.print("Time Left: ");
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
  lcd.print("Done. Thank You!");
  delay(2000);
}

```
pin out 
| Function        | Pin   |
| --------------- | ----- |
| Button 10 sec   | 2     |
| Button 15 sec   | 3     |
| Button 20 sec   | 4     |
| Button 25 sec   | 5     |
| Relay Sanitizer | 6     |
| Relay UV Light  | 7     |
| Buzzer          | 8     |
| HC-SR04 Trigger | 9     |
| HC-SR04 Echo    | 10    |
| I2C LCD         | A4/A5 |
