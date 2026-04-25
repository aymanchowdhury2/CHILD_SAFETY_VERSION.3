#include <WiFi.h>
#include <WebServer.h>
#include <Wire.h>
#include <Adafruit_MLX90614.h>
#include "MAX30105.h"

// --- WiFi Credentials ---
const char* ssid = "CHILD SAFETY";
const char* password = "234555444";

IPAddress local_IP(192, 168, 0, 9);
IPAddress gateway(192, 168, 0, 1);
IPAddress subnet(255, 255, 255, 0);

WebServer server(80);
Adafruit_MLX90614 mlx = Adafruit_MLX90614();
MAX30105 heartbeatSensor;

// --- Pin Definitions ---
#define BTN_DANGER 4   
#define PIN_WATER  5   
#define PIN_FIRE   23  
#define MIC_PIN    34 // MAX4466 Analog Out

void handleRoot() {
  float bodyTemp = mlx.readObjectTempC();
  int soundLevel = analogRead(MIC_PIN);
  
  String status = "SYSTEM STABLE";
  String color = "#00FF00"; 

  if (digitalRead(BTN_DANGER) == HIGH) { status = "BABY IN DANGER!"; color = "#FF0000"; }
  else if (digitalRead(PIN_WATER) == HIGH) { status = "BABY UNDER WATER!"; color = "#0000FF"; }
  else if (digitalRead(PIN_FIRE) == LOW) { status = "BABY ON FIRE!"; color = "#FFA500"; }
  else if (bodyTemp > 37.8) { status = "HIGH FEVER!"; color = "#FF4500"; }

  // HTML with Auto-Speech and Auto-Translate Logic
  String h = "<html><head><meta name='viewport' content='width=device-width, initial-scale=1'>";
  h += "<style>body{background:#000;color:#fff;font-family:sans-serif;text-align:center;}";
  h += ".card{border:2px solid #333;padding:20px;margin:10px;border-radius:20px;background:#111;}";
  h += ".status{font-size:30px;font-weight:bold;color:" + color + ";}</style></head>";
  h += "<body onload='autoListen()'>"; // Auto-starts when page opens
  
  h += "<div class='card'><h1>Safety Monitor</h1><div class='status'>" + status + "</div></div>";
  h += "<div class='card'>MLX90614 Temp: " + String(bodyTemp) + " C | Sound: " + String(soundLevel) + "</div>";
  h += "<div class='card'><h3>Voice Monitor (Auto-Translate)</h3>";
  h += "<p id='bn' style='color:#888;'>Listening for Bangla...</p>";
  h += "<p id='en' style='font-size:22px;color:#0f0;font-weight:bold;'></p></div>";
  
  h += "<script>function autoListen(){";
  h += "const r = new (window.SpeechRecognition || window.webkitSpeechRecognition)();";
  h += "r.lang = 'bn-BD'; r.continuous = true; r.start();";
  h += "r.onresult = (e) => { const m = e.results[e.results.length - 1][0].transcript;";
  h += "document.getElementById('bn').innerText = 'Bangla: ' + m;";
  h += "fetch('https://translate.googleapis.com/translate_a/single?client=gtx&sl=bn&tl=en&dt=t&q=' + encodeURI(m))";
  h += ".then(res => res.json()).then(d => { const trans = d[0][0][0];";
  h += "document.getElementById('en').innerText = 'English: ' + trans;";
  h += "const msg = new SpeechSynthesisUtterance(trans); window.speechSynthesis.speak(msg); }); };";
  h += "r.onend = () => { r.start(); }; } </script></body></html>"; // Restart listening if it stops
  
  server.send(200, "text/html", h);
}

void setup() {
  Wire.begin(21, 22);
  mlx.begin();
  heartbeatSensor.begin(Wire, I2C_SPEED_STANDARD);
  pinMode(BTN_DANGER, INPUT_PULLDOWN);
  pinMode(PIN_WATER, INPUT_PULLDOWN);
  pinMode(PIN_FIRE, INPUT_PULLUP);
  WiFi.softAP(ssid, password);
  WiFi.softAPConfig(local_IP, gateway, subnet);
  server.on("/", handleRoot);
  server.begin();
}

void loop() { server.handleClient(); }
