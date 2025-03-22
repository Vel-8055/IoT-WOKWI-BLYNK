#include <WiFi.h>
#include <LiquidCrystal_I2C.h>
#include <WiFiClient.h>

#define BLYNK_TEMPLATE_ID ""
#define BLYNK_TEMPLATE_NAME ""
#define BLYNK_AUTH_TOKEN ""
#include <BlynkSimpleEsp32.h>

#define sensor 34
#define buzzer 2

//Initialize the LCD display
LiquidCrystal_I2C lcd(0x27, 16, 2);

BlynkTimer timer;

// Enter your Auth token
char auth[] = "";

//Enter your WIFI SSID and password
char ssid[] = "Wokwi-GUEST";
char pass[] = "";

void setup() {
  // Debug console
  Serial.begin(115200);
  Blynk.begin(auth, ssid, pass, "blynk.cloud", 80);
  lcd.init();
  lcd.backlight();
  pinMode(buzzer, OUTPUT);

  lcd.setCursor(1, 0);
  lcd.print("Gas Detector");
  for (int a = 0; a <= 15; a++) {
    lcd.setCursor(a, 1);
    lcd.print(".");
    delay(200);
  }
  lcd.clear();
}

//Get the ultrasonic sensor values
void GASLevel() {
  int value = analogRead(sensor);
  value = map(value, 0, 4095, 0, 100);

  if (value >= 40) {
    digitalWrite(buzzer, HIGH);
    tone(buzzer, 200);

    Blynk.logEvent("gas_detector");

    lcd.setCursor(0, 1);
    lcd.print("Warning!  ");
    WidgetLED LED(V1);
    LED.on();
  } else {
    digitalWrite(buzzer, LOW);
    noTone(buzzer);
    lcd.setCursor(0, 1);
    lcd.print("Normal   ");
    WidgetLED LED(V1);
    LED.off();
  }

  Blynk.virtualWrite(V0, value);
  Serial.println(value);
  lcd.setCursor(0, 0);
  lcd.print("GAS Level :");
  lcd.print(value);
  lcd.print(" ");
}

void loop() {
  GASLevel();
  Blynk.run();//Run the Blynk library
  delay(200);
}
