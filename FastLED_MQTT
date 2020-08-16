#include <FastLED.h>
#include <ESP8266WiFi.h>
#include <PubSubClient.h>

FASTLED_USING_NAMESPACE

#if defined(FASTLED_VERSION) && (FASTLED_VERSION < 3001000)
#warning "Requires FastLED 3.1 or later; check github for latest code."
#endif

#define DATA_PIN    6
//#define CLK_PIN   4
#define LED_TYPE    WS2811
#define COLOR_ORDER GRB
#define NUM_LEDS    26
CRGB leds[NUM_LEDS];

#define BRIGHTNESS          96
#define FRAMES_PER_SECOND  120

const char* ssid = "user";
const char* wifi_password = "password";

const char* mqtt_server = "local_ip";
const char* mqtt_topic = "topic";
const char* mqtt_username = "user";
const char* mqtt_password = "password!";

const char* clientID = "ID";

WiFiClient wifiClient;
PubSubClient client(mqtt_server, 1883, wifiClient);

int LED_MODE = 0;
uint8_t gHue = 0; // rotating "base color" used by many of the patterns

void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("Message arrived [");
  Serial.print(topic);
  Serial.print("] ");
  for (int i = 0; i < length; i++) {
    char receivedChar = (char)payload[i];
    Serial.print(receivedChar);
    Serial.println();
    if (receivedChar == '0'){
      LED_MODE = 0;
    }
    if (receivedChar == '1'){
      LED_MODE = 1;
    }
    if (receivedChar == '2'){
      LED_MODE = 2;
      gHue = 0;
    }
  }
}

void setup(){
  delay(3000);
  
  FastLED.addLeds<LED_TYPE,DATA_PIN,COLOR_ORDER>(leds, NUM_LEDS).setCorrection(TypicalLEDStrip);

  // set master brightness control
  FastLED.setBrightness(BRIGHTNESS);
  
  fill_solid(leds, NUM_LEDS, CRGB(255, 0, 0));
  FastLED.show();

  Serial.begin(115200);
  delay(10);
  
  // tell FastLED about the LED strip configuration
  
  WiFi.begin(ssid, wifi_password);
  while (WiFi.status() != WL_CONNECTED){
    delay(500);
    Serial.print(".");
  }

  Serial.println("");
  Serial.println("WiFi connected");
  Serial.print("IP address: ");
  Serial.println(WiFi.localIP());

  reconnect();
  delay(10);
  resubscribe();

  client.setCallback(callback);

  fill_solid(leds, NUM_LEDS, CRGB(0, 255, 0));
  FastLED.show();
  delay(500);
  turnStripOff();
}

void loop(){
  if (!client.connected()) {
    reconnect();
    delay(10);
    resubscribe();
  }
  client.loop();
  
  // Call the current pattern function once, updating the 'leds' array.
  if(LED_MODE == 0){
    turnStripOff();
  }else if (LED_MODE == 1){
    turnStripOn();
  }else if (LED_MODE == 2){
    rainbow();
  }

  FastLED.show();  
  FastLED.delay(1000/FRAMES_PER_SECOND); 

  EVERY_N_MILLISECONDS( 20 ) { gHue+=2; } // slowly cycle the "base color" through the rainbow
}

void rainbow(){
  fill_rainbow( leds, NUM_LEDS, gHue, 7);
}

void turnStripOff(){
  fill_solid(leds, NUM_LEDS, CRGB(0, 0, 0));
}

void turnStripOn(){
  fill_solid(leds, NUM_LEDS, CRGB(255, 255, 0));
}

void reconnect(){
  while(!client.connected()){
    Serial.print("\nConnecting to [");
    Serial.print(mqtt_username);
    Serial.println("]");
    if(client.connect(clientID, mqtt_username, mqtt_password)){
      Serial.print("\nConnected to [");
      Serial.print(mqtt_username);
      Serial.println("]");
      Serial.println("");
    } else {
      Serial.println("\nTrying to connect again");
      delay(5000);
    }
  }
}

void resubscribe(){
  if (client.subscribe(mqtt_topic)) {
    Serial.print("Subscribed to ");
    Serial.print(mqtt_topic);
    Serial.println("!");
    Serial.println(">--------------------------<");
  }
  else {
    Serial.println("Subscription to ");
    Serial.print(mqtt_topic);
    Serial.println("failed...");
  }
}
