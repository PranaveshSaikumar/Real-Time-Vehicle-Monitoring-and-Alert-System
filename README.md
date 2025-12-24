# Real-Time Vehicle Monitoring and Alert System (VMAS) Simulation

### Name: Pranavesh Saikumar
### Regno: 212223040149

## Description

The Vehicle Monitoring and Alert System (VMAS) is a simulation program designed for ESP8266-based embedded systems with an OLED display and a buzzer. It monitors vehicle parameters like speed and engine temperature and provides real-time alerts:

Overspeed alert: Visual and audio notification when vehicle speed exceeds a defined threshold.

Engine heat alert: Predictive alert using a simple algorithm to calculate temperature risk based on rate of change and acceleration of temperature rise.

The system is implemented in Arduino C++ and demonstrates how edge AI concepts can be applied for predictive vehicle monitoring.

## Features

### Live Data Display:

OLED display shows speed (SPD) and engine temperature (TMP) in real-time.

### Audio-Visual Alerts:

Buzzer beep + multi-line messages for overspeed and engine heat alerts.

Predictive Engine Heat Alert:

Uses rate and acceleration of temperature rise to calculate a risk score.

Alerts before engine temperature reaches the critical limit.

Smooth Display Handling:

Keeps the last valid readings on display after the dataset finishes, preventing 0s from showing.

### Simulation Data:

Includes a mock dataset of 50 speed and temperature readings to simulate realistic vehicle behavior.

## Hardware Requirements

ESP8266 Board (NodeMCU v2 or similar)

OLED Display (128x64, I2C)

Buzzer (active or passive)

Connecting wires and breadboard

Software Requirements

Arduino IDE

## Libraries:

Adafruit GFX

Adafruit SSD1306

## Circuit Connections
![WhatsApp Image 2025-12-24 at 7 20 34 AM](https://github.com/user-attachments/assets/b05be2ba-a82f-4bfa-8c2c-1cb778c6c986)

Component	ESP8266 Pin
OLED SDA	D2
OLED SCL	D1
Buzzer	D5
GND	GND
VCC	3.3V/5V

## Program Code
```
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 64
#define OLED_RESET -1

Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, OLED_RESET);

#define BUZZER_PIN D5
#define SPEED_LIMIT 80
#define TEMP_ALERT 95     // Critical temp
#define RISK_ALERT 55     // Predictive risk threshold

const int DATA_LEN = 50;

int speedData[DATA_LEN] = {
  0,10,20,30,40,50,60,65,70,75,
  82,85,88,90,92,95,
  85,80,75,70,
  60,55,50,
  65,70,75,80,85,90,
  95,98,100,
  90,85,80,
  70,65,60,
  55,50,45,40,35,30
};

float engineTemp[DATA_LEN] = {
  35,38,40,42,45,47,49,51,53,55,
  58,60,62,65,68,70,
  72,74,76,78,
  80,82,84,
  86,88,90,92,94,96,
  98,100,102,
  100,98,96,
  94,92,90,
  88,86,84,82,80,78
};

float prevTemp = engineTemp[0];
float prevRate = 0;
bool overspeedActive = false;
bool heatAlertActive = false;

// ------------------ FUNCTIONS ------------------

void buzzerAlert() {
  tone(BUZZER_PIN, 4000);
  delay(300);
  noTone(BUZZER_PIN);
  delay(200);
}

void blinkMessage(String line1, String line2) {
  for (int i = 0; i < 3; i++) {
    display.clearDisplay();
    display.setTextSize(3);
    display.setTextColor(SSD1306_WHITE);
    display.setCursor(0, 10);
    display.println(line1);
    display.setCursor(0, 35);
    display.println(line2);
    display.display();
    buzzerAlert();
    delay(500);

    display.clearDisplay();
    display.display();
    delay(300);
  }
}

void showNormal(float speed, float temp, float risk) {
  display.clearDisplay();
  display.setTextSize(1);
  display.setCursor(0, 0);
  display.println("VMAS LIVE DATA");

  display.setTextSize(2);
  display.setCursor(0, 18);
  display.print("SPD:");
  display.print(speed);

  display.setCursor(0, 40);
  display.print("TMP:");
  display.print(temp);

  display.display();
}

// ------------------ SETUP ------------------

void setup() {
  pinMode(BUZZER_PIN, OUTPUT);
  Serial.begin(9600);

  Wire.begin(D2, D1);

  if (!display.begin(SSD1306_SWITCHCAPVCC, 0x3C)) {
    Serial.println("OLED FAILED");
    while (true);
  }

  display.clearDisplay();
  display.display();

  Serial.println("VMAS Simulation Started");
  Serial.println("--------------------------------");
}

// ------------------ LOOP ------------------

void loop() {
  for (int i = 0; i < DATA_LEN; i++) {
    float speed = speedData[i];
    float temp = engineTemp[i];

    // -------- Speed Threshold Logic --------
    if (speed > SPEED_LIMIT && !overspeedActive) {
      Serial.print("OVERSPEED ALERT: ");
      Serial.print(speed);
      Serial.println(" km/h");
      blinkMessage("OVER", "SPEED");
      overspeedActive = true;  // Avoid repeated alerts
    } else if (speed <= SPEED_LIMIT) {
      overspeedActive = false; // Reset when speed drops
    }

    // -------- Engine Heat Prediction Logic --------
    float rate = temp - prevTemp;
    float accel = rate - prevRate;
    float risk = (temp * 0.5) + (rate * 5.0) + (accel * 10.0);

    // Predictive alert: triggers before temp reaches critical
    if (risk >= RISK_ALERT && !heatAlertActive) {
      Serial.print("ENGINE HEAT PREDICTION ALERT! | Risk: ");
      Serial.println(risk);
      blinkMessage("ENGINE", "HEAT");
      heatAlertActive = true;
    } else if (risk < RISK_ALERT) {
      heatAlertActive = false;  // Reset if risk goes down
    }

    // Log all values
    Serial.print("Speed: "); Serial.print(speed);
    Serial.print(" km/h | Temp: "); Serial.print(temp);
    Serial.print(" C | Rate: "); Serial.print(rate);
    Serial.print(" | Accel: "); Serial.print(accel);
    Serial.print(" | ML Risk: "); Serial.println(risk);

    // Show normal readings
    showNormal(speed, temp, risk);

    prevTemp = temp;
    prevRate = rate;

    delay(3000);
  }

  // Keep last reading displayed until next loop
  showNormal(speedData[DATA_LEN - 1], engineTemp[DATA_LEN - 1], 0);
  Serial.println("---- Simulation Loop Restart ----");
  delay(3000);
}
```

## How It Works

### Simulation Loop:
The program iterates over a predefined dataset of speed and engine temperature.

### Overspeed Detection:
If the speed exceeds SPEED_LIMIT, a visual + audio alert is triggered.

### Engine Heat Prediction:

Calculates rate of temperature change: rate = temp - prevTemp

Calculates acceleration of temperature: accel = rate - prevRate

Computes risk score: risk = temp*0.5 + rate*5 + accel*10

If risk exceeds RISK_ALERT, predictive engine heat alert is triggered before reaching critical temperature.

## Display:

Normal mode: Shows live speed and temperature.

Alert mode: Shows multi-line messages with buzzer sounds.

## Sample Output

### Serial Monitor Logs:
<img width="773" height="472" alt="image" src="https://github.com/user-attachments/assets/136096a2-6479-41f3-ba62-1bb7ec5ae8a4" />

## OLED Display:

### Normal:
![WhatsApp Image 2025-12-24 at 7 12 37 AM](https://github.com/user-attachments/assets/8170bd7c-d91d-4412-8785-e712d9935889)

### Alert (Engine Heat):
![WhatsApp Image 2025-12-24 at 7 12 37 AM (2)](https://github.com/user-attachments/assets/2dfaa267-2e4f-433c-8dda-7e62a9922dd0)

### Alert (Overspeed):
![WhatsApp Image 2025-12-24 at 7 12 37 AM (1)](https://github.com/user-attachments/assets/3df125e9-f70a-4fbb-b2e2-89e767afb690)

## Future Improvements

Integrate real scanner for live data instead of simulation.

Refine predictive algorithm using machine learning or moving averages.

Logging data in esp flash

enclose evrything in boxs and design it a product 
