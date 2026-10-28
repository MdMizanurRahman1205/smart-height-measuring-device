
/*
  ==========================================================
       ELECTROSURGEON - ULTRASONIC HEIGHT METER
  ==========================================================

  Project Description:
  This project measures a person's height using an HC-SR04
  ultrasonic sensor mounted at a fixed height of 7 feet.

  The Arduino UNO calculates the person's height by
  subtracting the measured distance from the sensor's
  mounting height.

  Height is displayed on a 16x2 I2C LCD in:
    1. Centimeters (cm)
    2. Meters (m)
    3. Feet (ft)

  A push button switches between the measurement units.
  The Serial Monitor also displays the calculated height.

  Developed by: ElectroSurgeon
  Creator: Md Mizanur Rahman

  Components Required:
    - Arduino UNO
    - HC-SR04 Ultrasonic Sensor
    - 16x2 I2C LCD (address 0x27)
    - Push Button
    - Jumper Wires
    - USB Cable / 5V Power Supply

  Pin Connections:
    HC-SR04 TRIG -> D9
    HC-SR04 ECHO -> D8
    Push Button  -> D2 and GND
    LCD SDA      -> A4
    LCD SCL      -> A5
    LCD VCC      -> 5V
    LCD GND      -> GND

  Required Library:
    LiquidCrystal_I2C

  Sensor mounting height: 213.36 cm (7 feet)
  ==========================================================
*/

#include <Wire.h>
#include <LiquidCrystal_I2C.h>

// ---------------- LCD CONFIGURATION ----------------

LiquidCrystal_I2C lcd(0x27, 16, 2);

// If your LCD uses address 0x3F, change 0x27 to 0x3F.

// ---------------- HC-SR04 CONFIGURATION ----------------

#define TRIG_PIN 9
#define ECHO_PIN 8

// ---------------- BUTTON CONFIGURATION ----------------

#define BUTTON_PIN 2

// ---------------- SENSOR MOUNTING HEIGHT ----------------

// 7 feet = 213.36 cm
const float SENSOR_HEIGHT_CM = 213.36;

// ---------------- MEASUREMENT UNITS ----------------

// 0 = Centimeters
// 1 = Meters
// 2 = Feet

byte unitMode = 0;

bool lastButtonState = HIGH;
unsigned long lastButtonTime = 0;

const unsigned long DEBOUNCE_MS = 50;

// ---------------- SETUP ----------------

void setup() {
  pinMode(TRIG_PIN, OUTPUT);
  pinMode(ECHO_PIN, INPUT);
  pinMode(BUTTON_PIN, INPUT_PULLUP);

  digitalWrite(TRIG_PIN, LOW);

  Serial.begin(9600);

  lcd.init();
  lcd.backlight();

  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("HEIGHT METER");

  lcd.setCursor(0, 1);
  lcd.print("Starting...");

  delay(2000);

  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Developed By");

  lcd.setCursor(0, 1);
  lcd.print("ELECTRO_SURGEON");

  delay(2000);
  lcd.clear();

  Serial.println("================================");
  Serial.println(" ELECTROSURGEON HEIGHT METER");
  Serial.println(" Sensor Height: 213.36 cm");
  Serial.println("================================");
}

// ---------------- MAIN LOOP ----------------

void loop() {

  // -------- PUSH BUTTON / UNIT SELECTION --------

  bool currentButtonState = digitalRead(BUTTON_PIN);

  if (lastButtonState == HIGH &&
      currentButtonState == LOW &&
      millis() - lastButtonTime >= DEBOUNCE_MS) {

    unitMode++;

    if (unitMode > 2) {
      unitMode = 0;
    }

    lastButtonTime = millis();

    lcd.clear();

    Serial.print("Unit changed to: ");

    if (unitMode == 0) {
      Serial.println("Centimeters");
    } else if (unitMode == 1) {
      Serial.println("Meters");
    } else {
      Serial.println("Feet");
    }
  }

  lastButtonState = currentButtonState;

  // -------- ULTRASONIC DISTANCE MEASUREMENT --------

  digitalWrite(TRIG_PIN, LOW);
  delayMicroseconds(2);

  digitalWrite(TRIG_PIN, HIGH);
  delayMicroseconds(10);

  digitalWrite(TRIG_PIN, LOW);

  // Timeout after 30 milliseconds
  unsigned long duration = pulseIn(ECHO_PIN, HIGH, 30000UL);

  // -------- VALIDATE SENSOR READING --------

  if (duration == 0) {
    showInvalidReading("No Echo Detected");
    delay(300);
    return;
  }

  // Distance in centimeters
  float distanceCM = duration * 0.0343f / 2.0f;

  // -------- CALCULATE PERSON'S HEIGHT --------

  float heightCM = SENSOR_HEIGHT_CM - distanceCM;

  // Accept measurements between 50 cm and sensor height
  if (distanceCM <= 0 ||
      distanceCM > SENSOR_HEIGHT_CM ||
      heightCM < 50.0f) {

    showInvalidReading("Stand Under Sensor");
    delay(300);
    return;
  }

  // -------- DISPLAY HEIGHT IN SELECTED UNIT --------

  lcd.setCursor(0, 0);
  lcd.print("Height:         ");

  lcd.setCursor(0, 1);

  Serial.print("Height = ");

  if (unitMode == 0) {

    lcd.print(heightCM, 1);
    lcd.print(" cm          ");

    Serial.print(heightCM, 1);
    Serial.println(" cm");

  } else if (unitMode == 1) {

    float heightMeter = heightCM / 100.0f;

    lcd.print(heightMeter, 2);
    lcd.print(" m           ");

    Serial.print(heightMeter, 2);
    Serial.println(" meter");

  } else {

    float heightFeet = heightCM / 30.48f;

    lcd.print(heightFeet, 2);
    lcd.print(" ft          ");

    Serial.print(heightFeet, 2);
    Serial.println(" feet");
  }

  delay(300);
}

// ---------------- INVALID READING MESSAGE ----------------

void showInvalidReading(const char* message) {
  lcd.setCursor(0, 0);
  lcd.print("Height Meter    ");

  lcd.setCursor(0, 1);
  lcd.print("                ");
  lcd.setCursor(0, 1);

  // Keep the message within the 16-character LCD width.
  for (byte i = 0; i < 16 && message[i] != '\0'; i++) {
    lcd.print(message[i]);
  }

  Serial.println("No valid person detected");
}
