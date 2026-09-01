#include <Arduino.h>

// ========================================================
// ESP32 Motor Protection and Control System
// Features:
// - ACS712 Current Monitoring
// - Relay Motor Control
// - Emergency Stop (E-Stop)
// - Overcurrent Protection
// - Status LEDs
// ========================================================

// ---------------- Pin Configuration ----------------
const int CURRENT_SENSOR_PIN = 34;
const int RELAY_PIN = 25;
const int ESTOP_PIN = 27;

const int RUNNING_LED_PIN = 26;
const int STOPPED_LED_PIN = 14;
const int HIGH_CURRENT_LED_PIN = 12;
const int NORMAL_CURRENT_LED_PIN = 13;

// ---------------- ACS712 Parameters ----------------
// Change this according to the ACS712 version used.
const float ADC_REFERENCE = 3.3;
const float ADC_MAX = 4095.0;

// ACS712 sensitivity:
// 5A  = 0.185 V/A
// 20A = 0.100 V/A
// 30A = 0.066 V/A
const float ACS712_SENSITIVITY = 0.100;

// Approximate zero-current voltage
const float ACS712_ZERO_VOLTAGE = 2.5;

// ---------------- Protection Settings ----------------
const float OVERCURRENT_LIMIT = 2.0;

// ---------------- Timing ----------------
unsigned long lastReadingTime = 0;
const unsigned long READING_INTERVAL = 500;

// ---------------- System State ----------------
bool motorRunning = false;
bool overCurrent = false;
bool emergencyStop = false;


// ========================================================
// Function: Read Motor Current
// ========================================================
float readCurrent()
{
    int adcValue = analogRead(CURRENT_SENSOR_PIN);

    float sensorVoltage =
        (adcValue / ADC_MAX) * ADC_REFERENCE;

    float current =
        (sensorVoltage - ACS712_ZERO_VOLTAGE)
        / ACS712_SENSITIVITY;

    // Convert negative readings to positive magnitude
    current = abs(current);

    return current;
}


// ========================================================
// Function: Stop Motor
// ========================================================
void stopMotor()
{
    digitalWrite(RELAY_PIN, LOW);

    motorRunning = false;

    digitalWrite(RUNNING_LED_PIN, LOW);
    digitalWrite(STOPPED_LED_PIN, HIGH);
}


// ========================================================
// Function: Start Motor
// ========================================================
void startMotor()
{
    if (!emergencyStop && !overCurrent)
    {
        digitalWrite(RELAY_PIN, HIGH);

        motorRunning = true;

        digitalWrite(RUNNING_LED_PIN, HIGH);
        digitalWrite(STOPPED_LED_PIN, LOW);
    }
}


// ========================================================
// Setup
// ========================================================
void setup()
{
    Serial.begin(115200);

    // Inputs
    pinMode(CURRENT_SENSOR_PIN, INPUT);
    pinMode(ESTOP_PIN, INPUT_PULLUP);

    // Outputs
    pinMode(RELAY_PIN, OUTPUT);

    pinMode(RUNNING_LED_PIN, OUTPUT);
    pinMode(STOPPED_LED_PIN, OUTPUT);
    pinMode(HIGH_CURRENT_LED_PIN, OUTPUT);
    pinMode(NORMAL_CURRENT_LED_PIN, OUTPUT);

    // Initial state
    digitalWrite(RELAY_PIN, LOW);

    digitalWrite(RUNNING_LED_PIN, LOW);
    digitalWrite(STOPPED_LED_PIN, HIGH);

    digitalWrite(HIGH_CURRENT_LED_PIN, LOW);
    digitalWrite(NORMAL_CURRENT_LED_PIN, HIGH);

    Serial.println("=========================================");
    Serial.println(" ESP32 Motor Protection System");
    Serial.println("=========================================");
    Serial.println("System Initialized");
}


// ========================================================
// Main Loop
// ========================================================
void loop()
{
    // ----------------------------------------------------
    // Read E-Stop
    // ----------------------------------------------------
    emergencyStop = (digitalRead(ESTOP_PIN) == LOW);

    // ----------------------------------------------------
    // Read Motor Current
    // ----------------------------------------------------
    float current = readCurrent();

    // ----------------------------------------------------
    // Check Overcurrent
    // ----------------------------------------------------
    overCurrent = (current >= OVERCURRENT_LIMIT);

    // ----------------------------------------------------
    // Protection Logic
    // ----------------------------------------------------
    if (emergencyStop)
    {
        stopMotor();

        digitalWrite(HIGH_CURRENT_LED_PIN, LOW);
        digitalWrite(NORMAL_CURRENT_LED_PIN, LOW);
    }
    else if (overCurrent)
    {
        stopMotor();

        digitalWrite(HIGH_CURRENT_LED_PIN, HIGH);
        digitalWrite(NORMAL_CURRENT_LED_PIN, LOW);
    }
    else
    {
        digitalWrite(HIGH_CURRENT_LED_PIN, LOW);
        digitalWrite(NORMAL_CURRENT_LED_PIN, HIGH);

        // Start motor when system is safe
        startMotor();
    }

    // ----------------------------------------------------
    // Serial Monitoring
    // ----------------------------------------------------
    unsigned long currentMillis = millis();

    if (currentMillis - lastReadingTime >= READING_INTERVAL)
    {
        lastReadingTime = currentMillis;

        Serial.print("Motor Current: ");
        Serial.print(current, 2);
        Serial.println(" A");

        Serial.print("E-Stop: ");

        if (emergencyStop)
        {
            Serial.println("PRESSED");
        }
        else
        {
            Serial.println("RELEASED");
        }

        Serial.print("Motor Status: ");

        if (motorRunning)
        {
            Serial.println("RUNNING");
        }
        else
        {
            Serial.println("STOPPED");
        }

        Serial.print("Protection Status: ");

        if (overCurrent)
        {
            Serial.println("OVERCURRENT");
        }
        else if (emergencyStop)
        {
            Serial.println("EMERGENCY STOP");
        }
        else
        {
            Serial.println("NORMAL");
        }

        Serial.println("-----------------------------------------");
    }
}
