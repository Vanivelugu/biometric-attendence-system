# biometric-attendence-system
#include <Adafruit_Fingerprint.h>
#include <LiquidCrystal.h>
#include <SoftwareSerial.h>

// Fingerprint sensor pins (Digital pin 2 -> TX, pin 3 -> RX)
SoftwareSerial fingerSerial(2, 3);
Adafruit_Fingerprint finger(&fingerSerial);

// LCD pin setup: RS, E, D4, D5, D6, D7
LiquidCrystal lcd(7, 8, 9, 10, 11, 12);

// Buzzer and LED pins
int greenLED = 5;
int redLED = 6;
int buzzer = 4;

void setup() {
  Serial.begin(9600);            // Serial Monitor
  fingerSerial.begin(57600);     // Fingerprint sensor baud rate
  lcd.begin(16, 2);              // Initialize 16x2 LCD
  lcd.print("Fingerprint Sys");

  pinMode(greenLED, OUTPUT);
  pinMode(redLED, OUTPUT);
  pinMode(buzzer, OUTPUT);

  delay(2000);
  lcd.clear();

  if (finger.begin()) {
    lcd.print("Sensor Ready");
  } else {
    lcd.print("Sensor Error");
    while (1); // Halt if no sensor is detected
  }

  finger.getTemplateCount(); // Optional: shows number of fingerprints stored
  delay(1000);
  lcd.clear();
}

void loop() {
  lcd.setCursor(0, 0);
  lcd.print("Place Finger...");

  int result = getFingerprintID();

  if (result == FINGERPRINT_OK) {
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Access Granted!");
    digitalWrite(greenLED, HIGH);
    tone(buzzer, 1000); // Beep
    delay(1000);
    digitalWrite(greenLED, LOW);
    noTone(buzzer);
  } else {
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Access Denied!");
    digitalWrite(redLED, HIGH);
    tone(buzzer, 2000); // High-pitched beep
    delay(1000);
    digitalWrite(redLED, LOW);
    noTone(buzzer);
  }

  delay(2000);
  lcd.clear();
}

// Function to get fingerprint ID
int getFingerprintID() {
  uint8_t p = finger.getImage();
  if (p != FINGERPRINT_OK) return -1;

  p = finger.image2Tz();
  if (p != FINGERPRINT_OK) return -1;



  p = finger.f ingerSearch();
  if (p == FINGERPRINT_OK) {
    Serial.print("ID found: ");
    Serial.println(finger.fingerID);
    return FINGERPRINT_OK;
  } else {
    return -1;
  }
}
