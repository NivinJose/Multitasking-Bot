# Multitasking-Bot

<h3>Project Description:</h3> Multitasking Bot with Line Following, Obstacle Avoidance, Remote control and Voice control

<h1>Features:</h1>

<h3>Line Following:</h3>
                  The bot will be equipped with sensors capable of detecting lines on the ground and autonomously following them. This feature enables the robot to navigate predefined paths accurately, making it useful for tasks like warehouse automation or guiding visually impaired individuals.

<h3>Obstacle Avoidance:</h3>
                  Utilizing sensors such as ultrasonic or infrared, the robot will detect obstacles in its path and navigate around them. This capability ensures safe operation in dynamic environments, preventing collisions and enabling the robot to explore new territories autonomously.

<h3>Remote Control:</h3>
                  The robot can be controlled remotely using an Android application. Users can send commands to the robot through the application interface, such as initiating specific movements and adjusting speed. This feature adds flexibility and convenience, allowing users to interact with the robot from a distance.

<h3>Voice Control:</h3>
                  The robot can be controlled using an Andriod application. Users can use certain commands to make the robot go straight, right, left and reverse.


# Hardware Requirement: #
           * 5mm Acrylic Sheet 
           * DC Gear Motor(4)
           * Arduino UNO
           * IR Sensor (2)
           * L298 Motor Driver 
           * HC-05 Bluetooth Module 
           * IR Receiver Module
           * IR Remote 
           * sg90 servo motor
           * Ultrasonic Sensor Holder 
           * Ultrasonic Sensor hc-sr04 
           * 4Pcs Smart Robot Car Wheels 
           * Male to Female jumper Wires
           * Male to Male jumper wires
           * On/Off Switch
           * 18650 Battery Holder – 2 Cell(2)
           * 18650 Battery Cell 3.7V(4)

# Software Requirement: #
           * Arduino IDE Software

<h1>Circuit Diagram:</h1>


![capture](https://github.com/user-attachments/assets/c8c2cd8b-de20-4d96-b383-4cbdb782ac4c)

# Code: #

#include <SoftwareSerial.h>
SoftwareSerial BT_Serial(2, 3); 
#include <IRremote.h>
const int RECV_PIN = A5;
IRrecv irrecv(RECV_PIN);
decode_results results;
#define enA 10 
#define in1 9  
#define in2 8  
#define in3 7  
#define in4 6  
#define enB 5 
#define servo A4
#define R_S A0 
#define L_S A1 
#define echo A2    
#define trigger A3 
int distance_L, distance_F = 30, distance_R;
long distance;
int set = 20;
int bt_ir_data; 
int Speed = 130;  
int mode=0;
int IR_data;
void setup()
{ 
     pinMode(R_S, INPUT); 
     pinMode(L_S, INPUT); 
     pinMode(echo, INPUT );
     pinMode(trigger, OUTPUT); 
     pinMode(enA, OUTPUT); 
     pinMode(in1, OUTPUT); 
     pinMode(in2, OUTPUT); 
     pinMode(in3, OUTPUT); 
     pinMode(in4, OUTPUT); 
     pinMode(enB, OUTPUT); 
     irrecv.enableIRIn(); 
     irrecv.blink13(true);
     Serial.begin(9600); 
     BT_Serial.begin(9600); 
     pinMode(servo, OUTPUT);
     for (int angle = 70; angle <= 140; angle += 5)  
     {
          servoPulse(servo, angle); 
     }
     for (int angle = 140; angle >= 0; angle -= 5)
     {
          servoPulse(servo, angle); 
     }
     for (int angle = 0; angle <= 70; angle += 5)  
     {
          servoPulse(servo, angle);  
     }
     delay(500);
}
void loop()
{  
     if(BT_Serial.available() > 0)
     {  
          bt_ir_data = BT_Serial.read(); 
          Serial.println(bt_ir_data);     
          if(bt_ir_data > 20)
              {
                     Speed = bt_ir_data;
              }      
     }
     if (irrecv.decode(&results))
     {  
          Serial.println(results.value,HEX);
          bt_ir_data = IRremote_data();
          Serial.println(bt_ir_data); 
          irrecv.resume(); 
          delay(100);
     }
     if(bt_ir_data == 8)
     {
          mode=0; Stop();
     }    
     else if(bt_ir_data == 9)
     {
          mode=1; Speed=130;
     } 
     else if(bt_ir_data ==10)
     {
          mode=2; Speed=255;
     } 
     analogWrite(enA, Speed); 
     analogWrite(enB, Speed); 
     if(mode==0)
     {     
         if(bt_ir_data == 1){
         forword(); 
     }  
     else if(bt_ir_data == 2)
    {
        backword();
    }  
    else if(bt_ir_data == 3)
   {
        turnLeft();
   }  
    else if(bt_ir_data == 4)
    {
        turnRight();
    } 
    else if(bt_ir_data == 5)
   {
       Stop(); 
   }     
   else if(bt_ir_data == 6)
  {
       turnLeft();  
       delay(400);  
       bt_ir_data = 5;
   }
   else if(bt_ir_data == 7)
   {
       turnRight(); 
       delay(400);  
       bt_ir_data = 5;
    }
 }
if(mode==1){    
if((digitalRead(R_S) == 0)&&(digitalRead(L_S) == 0))
{
forword();
}  
if((digitalRead(R_S) == 1)&&(digitalRead(L_S) == 0))
{
turnRight();
}
if((digitalRead(R_S) == 0)&&(digitalRead(L_S) == 1))
{
turnLeft();
} 
if((digitalRead(R_S) == 1)&&(digitalRead(L_S) == 1))
{
Stop();
}     
} 
if(mode==2){    
distance_F = Ultrasonic_read();
 Serial.print("S=");Serial.println(distance_F);
  if (distance_F > set)
{
forword();
}
  else
{
Check_side();
}
}
delay(10);
}
long IRremote_data()
{
if(results.value==0xFF02FD)
{
IR_data=1;
}  
else if(results.value==0xFF9867)
{
IR_data=2;
} 
else if(results.value==0xFFE01F)
{
IR_data=3;
} 
else if(results.value==0xFF906F)
{
IR_data=4;
} 
else if(results.value==0xFF629D || results.value==0xFFA857)
{
IR_data=5;
} 
else if(results.value==0xFF30CF)
{
IR_data=8;
} 
else if(results.value==0xFF18E7)
{
IR_data=9;
} 
else if(results.value==0xFF7A85)
{
IR_data=10;
} 
return IR_data;
}
void servoPulse (int pin, int angle)
{
int pwm = (angle*11) + 500;      
 digitalWrite(pin, HIGH);
 delayMicroseconds(pwm);
 digitalWrite(pin, LOW);
 delay(50);                  
}
long Ultrasonic_read(){
  digitalWrite(trigger, LOW);
  delayMicroseconds(2);
  digitalWrite(trigger, HIGH);
  delayMicroseconds(10);
  distance = pulseIn (echo, HIGH);
  return distance / 29 / 2;
}
void compareDistance(){
       if (distance_L > distance_R){
  turnLeft();
  delay(350);
  }
  else if (distance_R > distance_L)
{
  turnRight();
  delay(350);
  }
  else{
  backword();
  delay(300);
  turnRight();
  delay(600);
  }
}
void Check_side()
{
    Stop();
    delay(100);
 for (int angle = 70; angle <= 140; angle += 5)  
{
   servoPulse(servo, angle); 
 }
    delay(300);
    distance_L = Ultrasonic_read();
    delay(100);
  for (int angle = 140; angle >= 0; angle -= 5)  
{
   servoPulse(servo, angle);  
}
    delay(500);
    distance_R = Ultrasonic_read();
    delay(100);
 for (int angle = 0; angle <= 70; angle += 5) 
{
   servoPulse(servo, angle);  }
    delay(300);
    compareDistance();
}
void forword()
{  
digitalWrite(in1, HIGH); 
digitalWrite(in2, LOW);  
digitalWrite(in3, LOW);  
digitalWrite(in4, HIGH); 
}
void backword()
{ 
digitalWrite(in1, LOW);  
digitalWrite(in2, HIGH); 
digitalWrite(in3, HIGH); 
digitalWrite(in4, LOW);  
}
void turnRight()
{ 
digitalWrite(in1, LOW);  
digitalWrite(in2, HIGH); 
digitalWrite(in3, LOW);  
digitalWrite(in4, HIGH); 
}
void turnLeft()
{ 
digitalWrite(in1, HIGH); 
digitalWrite(in2, LOW);  
digitalWrite(in3, HIGH); 
digitalWrite(in4, LOW);  
}
void Stop()
{ 
digitalWrite(in1, LOW); 
digitalWrite(in2, LOW); 
digitalWrite(in3, LOW); 
digitalWrite(in4, LOW); 
}


