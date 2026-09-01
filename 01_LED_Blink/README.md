#include <Arduino.h>

// ========================================================
// ESP32 Multi-Sensor Control System
// Features:
// - IR Detection
// - PWM Brightness Control
// - Serial Telemetry
// Compatible with ESP32 Arduino Core 2.x
// ========================================================

// --- Hardware Pin Configurations ---
const int IR_SENSOR_PIN = 27;  // IR Sensor Digital Input
const int IR_LED_PIN = 25;     // LED indicator for IR Status
const int POT_PIN = 34;        // Potentiometer Analog Input
const int PWM_LED_PIN = 26;    // PWM Controlled LED

// --- ESP32 LEDC PWM Parameters ---
const int PWM_FREQ = 5000;
const int PWM_CHANNEL = 0;
const int PWM_RES = 8;

// --- Non-Blocking Timing ---
unsigned long lastSerialTime = 0;
const unsigned long SERIAL_INTERVAL = 250;

void setup() {
    Serial.begin(115200);

    pinMode(IR_SENSOR_PIN, INPUT_PULLUP);
    pinMode(IR_LED_PIN, OUTPUT);

    ledcSetup(PWM_CHANNEL, PWM_FREQ, PWM_RES);
    ledcAttachPin(PWM_LED_PIN, PWM_CHANNEL);

    digitalWrite(IR_LED_PIN, LOW);
    ledcWrite(PWM_CHANNEL, 0);

    Serial.println("=========================================");
    Serial.println(" ESP32 Control System Fully Initialized");
    Serial.println("=========================================");
}

void loop() {

    // ----------------------------------------------------
    // Task 1: IR Sensor Reading & Digital Output
    // ----------------------------------------------------
    int irState = digitalRead(IR_SENSOR_PIN);
    bool objectDetected = (irState == LOW);

    digitalWrite(IR_LED_PIN, objectDetected ? HIGH : LOW);

    // ----------------------------------------------------
    // Task 2: Potentiometer, PWM & Calculations
    // ----------------------------------------------------
    int potRaw = analogRead(POT_PIN);

    int pwmDutyCycle = map(potRaw, 0, 4095, 0, 255);

    float brightnessPercent =
        (potRaw / 4095.0f) * 100.0f;

    float voltage =
        (potRaw / 4095.0f) * 3.3f;

    ledcWrite(PWM_CHANNEL, pwmDutyCycle);

    // ----------------------------------------------------
    // Task 3: Non-Blocking Serial Telemetry
    // ----------------------------------------------------
    unsigned long currentMillis = millis();

    if (currentMillis - lastSerialTime >= SERIAL_INTERVAL) {

        lastSerialTime = currentMillis;

        Serial.print("IR Status: ");
        Serial.print(
            objectDetected ? "[ DETECTED ] " : "[ CLEAR ] "
        );

        Serial.print("| Pot Raw: ");
        Serial.print(potRaw);

        Serial.print(" | Brightness: ");
        Serial.print(brightnessPercent, 1);
        Serial.print("%");

        Serial.print(" | Voltage: ");
        Serial.print(voltage, 2);
        Serial.println(" V");
    }
}
