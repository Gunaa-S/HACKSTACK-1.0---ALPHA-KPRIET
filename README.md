# HACKSTACK-1.0---ALPHA-KPRIET
/*************************************************************
   TEAM ALPHA KPRIET "PORTABLE PLANTATION"
 *************************************************************/

// Template ID, Device Name and Auth Token are provided by the Blynk.Cloud
// See the Device Info tab, or Template settings
#define BLYNK_TEMPLATE_ID           "TMXFbx1i"
#define BLYNK_DEVICE_NAME           "Quickstart Device"
#define BLYNK_AUTH_TOKEN            "yPERyNHSF3ulWHXXhWk"


// Comment this out to disable prints and save space
#define BLYNK_PRINT Serial


#include <ESP8266WiFi.h>
#include <BlynkSimpleEsp8266.h>

char auth[] = BLYNK_AUTH_TOKEN;
int moi;
int value;
long previousMillis = 0;
long interval = 100;
int sensor_pin = D0;
int output_value ;
const char* host = "maker.ifttt.com";
// Your WiFi credentials.
// Set password to "" for open networks.
char ssid[] = "Bbb";
char pass[] = "12345671";

BlynkTimer timer;

// This function is called every time the Virtual Pin 0 state changes
BLYNK_WRITE(V0)
{
  // Set incoming value from pin V0 to a variable
   value = param.asInt();

  //Update state
  Blynk.virtualWrite(Vx, value);
}

// This function is called every time the device is connected to the Blynk.Cloud
BLYNK_CONNECTED()
{
  // Change Web Link Button message to "Congratulations!"
  Blynk.setProperty(V3x, "offImageUrl", "https://static-image.nyc3.cdn.digitaloceanspaces.com/general/fte/congratulations.png");
  Blynk.setProperty(V3x, "onImageUrl",  "https://static-image.nyc3.cdn.digitaloceanspaces.com/general/fte/congratulations_pressed.png");
  Blynk.setProperty(V3x "url", "https://docs.blynk.io/en/getting-started/what-do-i-need-to-blynk/how-quickstart-device-was-made");
}

// This function sends Arduino's uptime every second to Virtual Pin 2.
void myTimerEvent()
{ 
  Serial.begin(9600);
  // You can send any value at any time.
  // Please don't send more that 10 values per second.             // code snippet for rounding of analog value between 0% to 100%
  //Blynk.virtualWrite(V4, millis() / 100);
  moi = analogRead(A0);                                                       
  moi = map(moi, 0, 1023, 0, 100);
  Blynk.virtualWrite(V4x, moi);
  Serial.println(moi);
}

void setup()
{
  // Debug console
  Serial.begin(115200);

  Blynk.begin(auth, ssid, pass);
  // You can also specify server:
  //Blynk.begin(auth, ssid, pass, "blynk.cloud", 80);
  //Blynk.begin(auth, ssid, pass, IPAddress(192,168,1,100), 8080);
  pinMode(D4, OUTPUT);
  pinMode(A0, INPUT);
  pinMode(D8, OUTPUT);
    
  
  // Setup a function to be called every second
  timer.setInterval(100L, myTimerEvent);
}

void waterlevel()
//this to convert analog value into digital value as 0's and 1's
 {  
    output_value = digitalRead(sensor_pin); 
    Serial.begin(115);
    Serial.print("Moisture : "); 
    Serial.print(output_value);
    if (output_value) {
    	Serial.println("Status: Soil is too dry - time to water!");
    	digitalWrite(D8,HIGH);
  } else {
    Serial.println("Status: Soil moisture is perfect");
    digitalWrite(D8,LOW);
  }
  
}
////automation feature for emagency action
void autowater(){
  if(moi>95){
  	digitalWrite(D4,HIGH);
  	delay(2000);
  	digitalWrite(D4,LOW);
  }
  else{
       Serial.println("water not needed");
  }  
}
/// end of automation feature
void telecall() 
{           

// this for notification alert and telecall//
      WiFiClient client; 
      const int httpPort = 80;  
      if (!client.connect(host, httpPort)) 
      {  
         Serial.println("connection failed");  
         return;
      } 
      if(Serial.available())
      {                      
        
         
         if (output_value)
        {    
            String url1 = "/trigger/alertCall/json/with/key/epJ8vWtD_1eWk69cmlSoZjLFDLvbrbIc9G6C_ZUHGF3"; 
            Serial.print("Requesting URL: ");
            Serial.println(url1);
            Serial.println(output_value);            
            client.print(String("GET ") + url1 + " HTTP/1.1\r\n" + "Host: " + host + "\r\n" + "Connection: close\r\n\r\n");    
         }                     
          else
          {
            Serial.println("Correct character not pressed.Try again");            
          }
       }          
       
             
}


void loop(){
  
  digitalWrite(D4, value);
  Blynk.run();
  timer.run();
  waterlevel();
  telecall();
  autowater();
  unsigned long currentMillis = millis();
  if(currentMillis - previousMillis > interval) {
    previousMillis = currentMillis;  
//to push notification to the mobiles//
  	if (moi>=90)
       		 {
			 Serial.print("Connecting to "); 
			 Serial.println(host);
 			 WiFiClient client;
			 const int httpPort = 80;
			 if (!client.connect(host, httpPort)) {
			 Serial.println("Connection failed");
    			 return;
  }
  String url = "/trigger/RMA/with/key/epJ8vWtD_1elSoZjLFDLvbrbIc9G6C_ZUHGF3";
  Serial.print("Requesting URL: ");
  Serial.println(url);
  client.print(String("GET ") + url + " HTTP/1.1\r\n" +
               "Host: " + host + "\r\n" + 
               "Connection: close\r\n\r\n");
  while(client.connected())
  {
    if(client.available())
    {
      String line = client.readStringUntil('\r');
      Serial.print(line);
    } else {
      delay(10);
    };
  }
  Serial.println();
  Serial.println("closing connection");
  client.stop();
  }  
}

  
}

// team apha kpriet.
