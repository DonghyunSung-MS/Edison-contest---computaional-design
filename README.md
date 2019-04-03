# Edison-contest---computaional-design
19.01.05 to 19.01.24
#include <Servo.h>
#include <Stepper.h>
int n=0;
int a=0;
int m=0;
int k=0;

//초음파센서
int trigPin= 26;
int echoPin= 28;
int distance=30;
int duration=30;

//LED 센서
int S0=53;
int S1=2;
int S2=49;
int S3=10;
int SensorOut=51;
int redFrequency = 0;
int greenFrequency = 0;
int blueFrequency = 0;
int redColor = 0;
int greenColor = 0;
int blueColor = 0;

//적외선 센서
int LW= A0;
int LL= A1;
int RL= A2;
int RW= A3;
int BL=A4;
int BR=A5;

//DC모터
int DCL=172;
int DCR=192;
int RDCL=230;
int RDCR=165;
int N1= 7;//왼쪽
int N2= 6;
int ENA= 9;
int N3= 5;//오른쪽
int N4= 4;
int ENB= 8;

//Rotating Motor

Servo Rmotor;
int angle=0;

//ArmGrab Motor 
const int stepsPerRevolution = 1024;
Stepper myStepper(stepsPerRevolution,36,32,34,30);        

//Liftmotor
Servo amotor;
int Lmotorspeed = 100;
int LiftArduino = 3;

//-------------------------------------------------------------------------------------------------

void setup() {
  
 Serial.begin(9600);
 
 //초음파 센서
 pinMode(trigPin,OUTPUT);
 pinMode(echoPin,INPUT);

 //LED 센서
 pinMode(S0, OUTPUT);
 pinMode(S1, OUTPUT);
 pinMode(S2, OUTPUT);
 pinMode(S3, OUTPUT);
 pinMode(SensorOut, INPUT);
 digitalWrite(S0,HIGH);
 digitalWrite(S1,LOW);
 
 //적외선 센서
 pinMode(LW, INPUT);
 pinMode(LL, INPUT);
 pinMode(RL, INPUT);
 pinMode(RW, INPUT);
 pinMode(BL, INPUT);
 pinMode(BR, INPUT);
 
 //DC모터 모터드라이브
 pinMode(ENA,INPUT);
 pinMode(ENB,INPUT);
 pinMode(N1,OUTPUT);
 pinMode(N2,OUTPUT);
 pinMode(N3,OUTPUT);
 pinMode(N4,OUTPUT);
 digitalWrite(N1,LOW);
 digitalWrite(N2,LOW);  
 digitalWrite(N3,LOW);
 digitalWrite(N4,LOW);

 //roatingmotor(servo)
 Rmotor.attach(11);
 Rmotor.write(90); 

//ArmGrab Motor
 myStepper.setSpeed(30); 

//Lift Motor
  amotor.attach(LiftArduino);
 
}

//-------------------------------------------------------------------------------------------------

void loop() {
  long duration, distance;
  //초음파 거리측정
  digitalWrite(trigPin,LOW);
  delayMicroseconds(2);
  digitalWrite(trigPin,HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin,LOW);
  duration=pulseIn(echoPin, HIGH);
  distance=duration/29/2;  //distance(cm)
  // 적외선 1 검은색 0흰색
  int LW1 = analogRead(LW)/800;
  int LL1 = analogRead(LL)/800;
  int RL1 = analogRead(RL)/800;
  int RW1 = analogRead(RW)/800;
  int BL1 = analogRead(BL)/800;
  int BR1 = analogRead(BR)/800;
  Serial.print(LW1);
  Serial.print(LL1);
  Serial.print(RL1);
  Serial.print(RW1);
  Serial.print(distance);
  //알고리즘 시작
  if(distance>5){
    Serial.println("There is no obstacle near hear");
    if(LW1==0 && RW1==0){
      Serial.println("Linetracing");
      linetracing();
      delay(100);
    }
    else if(LW1==1 && RW1==0&&LL1==1&&RL1==1){
      Serial.println("COUNT");
      turnleft();
    }
}
  
  else if(distance<=5 && distance>1){
    Serial.println("Watch Out!");
    /*digitalWrite(N1,HIGH);
    digitalWrite(N2,LOW);
    digitalWrite(N3,HIGH);
    digitalWrite(N4,LOW);
    analogWrite(ENA,DCspeed);
    analogWrite(ENB,DCspeed);
    delay(200);*/
    n=n+1;
    stopping();
    if(n==1){
      Serial.println("I'm in 1st mission.");   
      Grabbing();
      Grabbing();
      halfGrabbing();
      colordetect();
      Serial.print("R= ");
      Serial.print(redFrequency);
      Serial.print("");
      Serial.print("G= ");
      Serial.print(greenFrequency);
      Serial.print("");
      Serial.print("B= ");
      Serial.print(blueFrequency);
      Serial.print("");
      if(redFrequency<300 && greenFrequency>400 && blueFrequency>300){
        Serial.print("Red");
        Serial.print("");
        lifting();
        halflifting();
        bodyTR();
        delay(500);
        releasing();
        releasing();
        bodyTRD();
        goback();
        halfgoback();
        
      }
      else if(redFrequency>300 && greenFrequency<600 &&  blueFrequency<400){
        Serial.print("Blue");
        Serial.print("");
        lifting();
        halflifting();
        bodyTL();
        releasing();
        releasing();
        bodyTLD();
        goback();
        halfgoback();
      }
      else{
        if(redFrequency<300 && greenFrequency>400 && blueFrequency>300){
          Serial.print("Red");
          Serial.print("");
          lifting();
          halflifting();
          bodyTR();
          delay(500);
          releasing();
          releasing();
          bodyTRD();
          goback();
          halfgoback();
        }
        else if(redFrequency>300 && greenFrequency<600 &&  blueFrequency<400){
          Serial.print("Blue");
          Serial.print("");
          lifting();
          halflifting();
          bodyTL();
          releasing();
          halfreleasing();
          bodyTLD();
          goback();
          halfgoback();
        }
        //Random Choice
        else{
          Serial.print("I don't know");
          lifting();
          halflifting();
          bodyTL();
          releasing();
          halfreleasing();
          bodyTLD();
          goback();
          halfgoback();
        }
      }
    }
      
    else if(n==2){
      Serial.println("I'm in 2nd mission.");   
      Grabbing();
      delay(10);
      Grabbing();
      Grabbing();
      lifting();
      digitalWrite(trigPin,LOW);
      delayMicroseconds(2);
      digitalWrite(trigPin,HIGH);
      delayMicroseconds(10);
      digitalWrite(trigPin,LOW);
      duration=pulseIn(echoPin, HIGH);
      distance=duration/29/2;  //distance(cm)
      Serial.println(distance);
      while(distance<10){
        int LW1 = analogRead(LW)/800;
        int LL1 = analogRead(LL)/800;
        int RL1 = analogRead(RL)/800;
        int RW1 = analogRead(RW)/800;
        int BL1 = analogRead(BL)/800;
        int BR1 = analogRead(BR)/800;
        long duration, distance;
  //초음파 거리측정
        digitalWrite(trigPin,LOW);
        delayMicroseconds(2);
        digitalWrite(trigPin,HIGH);
        delayMicroseconds(10);
        digitalWrite(trigPin,LOW);
        duration=pulseIn(echoPin, HIGH);
        distance=duration/29/2;  //distance(cm)
        Serial.println("line");
        Serial.print(LW1);
        Serial.print(LL1);
        Serial.print(RL1);
        Serial.print(RW1);
        Serial.print(distance);
        linetracing();
        delay(100);
        if((LW1==1 && LL1==1&&RL1==1) or (RW1==1 && LL1==1&&RL1==1)){
          Serial.println("stop and release");
          stopping();
          goback();
          releasing();;
          releasing();
          break;      
          /*digitalWrite(N1,LOW);
          digitalWrite(N2,HIGH);
          digitalWrite(N3,LOW);
          digitalWrite(N4,HIGH);
          analogWrite(ENA,RDCL);
          analogWrite(ENB,RDCR);
          delay(800);*/
        }
        /*if(distance>11){
          break;
      }*/
    }
      int LW1 = analogRead(LW)/800;
      int LL1 = analogRead(LL)/800;
      int RL1 = analogRead(RL)/800;
      int RW1 = analogRead(RW)/800;
      int BL1 = analogRead(BL)/800;
      int BR1 = analogRead(BR)/800;
  while(BL1==0 or BR1==0){
    int BL1 = analogRead(BL)/800;
    int BR1 = analogRead(BR)/800;
    Serial.println("reverse");
    reverse();
    if(BL1==1&&BR1==1){
      digitalWrite(N1,LOW);
      digitalWrite(N2,LOW);  
      digitalWrite(N3,LOW);
      digitalWrite(N4,HIGH);
      analogWrite(ENA,120);
      analogWrite(ENB,220);
      delay(750);
      digitalWrite(N1,HIGH);
      digitalWrite(N2,LOW);  
      digitalWrite(N3,HIGH);
      digitalWrite(N4,LOW);
      analogWrite(ENA,220);
      analogWrite(ENB,150);
      delay(900);
      Serial.println("break");
      break;
      }
      }
    }
      
    else if(n==3){
      Serial.println("I'm in 3rd mission.");   
      Grabbing();
      Grabbing();
      Grabbing();
      halfGrabbing();
      lifting();
      Serial.println("Grab and Lift");
      int LW1 = analogRead(LW)/800;
      int LL1 = analogRead(LL)/800;
      int RL1 = analogRead(RL)/800;
      int RW1 = analogRead(RW)/800;
      while(LW1==1 or LL1==1 or RL1==1 or RW1==1){
         int LW1 = analogRead(LW)/800;
         int LL1 = analogRead(LL)/800;
         int RL1 = analogRead(RL)/800;
         int RW1 = analogRead(RW)/800;
         Serial.println("Mission3 LineTracing");
         linetracing2();
         if(LW1==0&&LL1==0&&RL1==0&&RW1==0){
          Serial.println("Done!");
    }
}
}
  }
}

//---------------------------------------------------------------------------------------------
//DC모터 함수
void linetracing(){
  int LW1 = analogRead(LW)/800;
  int LL1 = analogRead(LL)/800;
  int RL1 = analogRead(RL)/800;
  int RW1 = analogRead(RW)/800;
    //전진
  if(LL1==1 && RL1==1){
    digitalWrite(N1,HIGH);
    digitalWrite(N2,LOW);
    digitalWrite(N3,HIGH);
    digitalWrite(N4,LOW);
    analogWrite(ENA,(0.80*DCL));
    analogWrite(ENB,(0.90*DCR));
  }
  //좌보정
  else if(LL1==1 && RL1==0){
    digitalWrite(N1,LOW);
    digitalWrite(N2,LOW);
    digitalWrite(N3,HIGH);
    digitalWrite(N4,LOW);
    analogWrite(ENB,(0.69*DCR));
    
  }
  //우보정
  else if(LL1==0 && RL1==1){
    digitalWrite(N1,HIGH);
    digitalWrite(N2,LOW);
    digitalWrite(N3,LOW);
    digitalWrite(N4,LOW);
    analogWrite(ENA,(0.65*DCL));
  }
  else{
    digitalWrite(N1,LOW);
    digitalWrite(N2,HIGH);
    digitalWrite(N3,LOW);
    digitalWrite(N4,HIGH);
    analogWrite(ENA,RDCL);
    analogWrite(ENB,RDCR);       
  }
}

void linetracing2(){
  int LW1 = analogRead(LW)/800;
  int LL1 = analogRead(LL)/800;
  int RL1 = analogRead(RL)/800;
  int RW1 = analogRead(RW)/800;
    //전진
  if(LL1==1 && RL1==1){
    digitalWrite(N1,HIGH);
    digitalWrite(N2,LOW);
    digitalWrite(N3,HIGH);
    digitalWrite(N4,LOW);
    analogWrite(ENA,(0.80*DCL)+10);
    analogWrite(ENB,(0.90*DCR)+10);
  }
  //좌보정
  else if(LL1==1 && RL1==0){
    digitalWrite(N1,LOW);
    digitalWrite(N2,LOW);
    digitalWrite(N3,HIGH);
    digitalWrite(N4,LOW);
    analogWrite(ENB,(0.69*DCR)+10);
    
  }
  //우보정
  else if(LL1==0 && RL1==1){
    digitalWrite(N1,HIGH);
    digitalWrite(N2,LOW);
    digitalWrite(N3,LOW);
    digitalWrite(N4,LOW);
    analogWrite(ENA,(0.65*DCL)+10);
  }
  else{
    digitalWrite(N1,LOW);
    digitalWrite(N2,HIGH);
    digitalWrite(N3,LOW);
    digitalWrite(N4,HIGH);
    analogWrite(ENA,RDCL);
    analogWrite(ENB,RDCR+10);       
  }
}

void stopping(){
 digitalWrite(N1,LOW);
 digitalWrite(N2,LOW);  
 digitalWrite(N3,LOW);
 digitalWrite(N4,LOW);
}
void reverse(){
  int BL1 = analogRead(BL)/800;
  int BR1 = analogRead(BR)/800;
    //후진
  if(BL1==0 && BR1==0){
    digitalWrite(N1,LOW);
    digitalWrite(N2,HIGH);
    digitalWrite(N3,LOW);
    digitalWrite(N4,HIGH);
    analogWrite(ENA,RDCL);
    analogWrite(ENB,RDCR);
  }
  //좌보정
  else if(BL1==0 && BR1==1){
    digitalWrite(N1,LOW);
    digitalWrite(N2,HIGH);
    digitalWrite(N3,HIGH);
    digitalWrite(N4,LOW);
    analogWrite(ENA,RDCL);
    analogWrite(ENB,235);
  }
  //우보정
  else if(BL1==1 && BR1==0){
    digitalWrite(N1,LOW);
    digitalWrite(N2,LOW);
    digitalWrite(N3,LOW);
    digitalWrite(N4,HIGH);
    analogWrite(ENB,RDCR);
  }  
}
void turnleft(){
  int LW1 = analogRead(LW)/800;
  int LL1 = analogRead(LL)/800;
  int RL1 = analogRead(RL)/800;
  int RW1 = analogRead(RW)/800;
  digitalWrite(N1,LOW);
  digitalWrite(N2,LOW);  
  digitalWrite(N3,HIGH);
  digitalWrite(N4,LOW);
  analogWrite(ENA,DCL);
  analogWrite(ENB,DCL);
  delay(800); 
}


//서보모터구동
void lifting(){
  // Run Motor
  Lmotorspeed = 1000; //set position variable
  amotor.writeMicroseconds(Lmotorspeed); // tell servo to move as indicated by variable 'mspeed'
  delay(2000); //time for the servo to move

  //Stop Motor
  Lmotorspeed= 1500; //set position variable
  amotor.writeMicroseconds(Lmotorspeed); // tell servo to move as indicated by variable 'mspeed'
  delay(15); //time for the servo to move
}
void halflifting(){
  // Run Motor
  Lmotorspeed = 1000; //set position variable
  amotor.writeMicroseconds(Lmotorspeed); // tell servo to move as indicated by variable 'mspeed'
  delay(1000); //time for the servo to move

  //Stop Motor
  Lmotorspeed= 1500; //set position variable
  amotor.writeMicroseconds(Lmotorspeed); // tell servo to move as indicated by variable 'mspeed'
  delay(15); //time for the servo to move
}
void halfgoback(){
  // Run Motor
  Lmotorspeed=2000; //set position variable
  amotor.writeMicroseconds(Lmotorspeed); // tell servo to move as indicated by variable 'mspeed'
  delay(900); //time for the servo to move

  //Stop Motor
  Lmotorspeed= 1500; //set position variable
  amotor.writeMicroseconds(Lmotorspeed); // tell servo to move as indicated by variable 'mspeed'
  delay(15); //time for the servo to move
}
void goback(){
  // Run Motor
  Lmotorspeed=2000; //set position variable
  amotor.writeMicroseconds(Lmotorspeed); // tell servo to move as indicated by variable 'mspeed'
  delay(1980); //time for the servo to move

  //Stop Motor
  Lmotorspeed= 1500; //set position variable
  amotor.writeMicroseconds(Lmotorspeed); // tell servo to move as indicated by variable 'mspeed'
  delay(15); //time for the servo to move
}

//스텝모터 구동
void Grabbing(){
  myStepper.step(-stepsPerRevolution);

  }

void halfGrabbing(){
  myStepper.step(-stepsPerRevolution/2);
}

void releasing(){
  myStepper.step(stepsPerRevolution);

}

void halfreleasing(){
  myStepper.step(stepsPerRevolution/2);
}

void bodyTR(){
    for(angle =90; angle > 0; angle -=3)    // command to move from 0 degrees to 180 degrees 
  {                                  
    Rmotor.write(angle);                 //command to rotate the servo to the specified angle
                       
  } 
}
void bodyTRD(){
    for(angle =0; angle < 90.; angle +=2)    // command to move from 0 degrees to 180 degrees 
  {                                  
    Rmotor.write(angle);                 //command to rotate the servo to the specified angle
                         
  } 
}
void bodyTL(){
    for(angle =90; angle < 180; angle +=3)    // command to move from 0 degrees to 180 degrees 
  {                                  
    Rmotor.write(angle);                 //command to rotate the servo to the specified angle
                          
  }
} 
void bodyTLD(){
 
  for(angle = 180 ; angle>100; angle-=1)     // command to move from 180 degrees to 0 degrees 
  {                                
    Rmotor.write(angle);              //command to rotate the servo to the specified angle
    delay(20);                      
  } 

}

void colordetect(){
    // Setting RED (R) filtered photodiodes to be read
  digitalWrite(S2,LOW);
  digitalWrite(S3,LOW);
  
  // Reading the output frequency
  redFrequency = pulseIn(SensorOut, LOW);
  // Remaping the value of the RED (R) frequency from 0 to 255
  // You must replace with your own values. Here's an example: 
  // redColor = map(redFrequency, 70, 120, 255,0);
  //redColor = map(redFrequency, XX, XX, 255,0);
  
  // Printing the RED (R) value
  delay(100);
  
  // Setting GREEN (G) filtered photodiodes to be read
  digitalWrite(S2,HIGH);
  digitalWrite(S3,HIGH);
  
  // Reading the output frequency
  greenFrequency = pulseIn(SensorOut, LOW);
  // Remaping the value of the GREEN (G) frequency from 0 to 255
  // You must replace with your own values. Here's an example: 
  // greenColor = map(greenFrequency, 100, 199, 255, 0);
  //greenColor = map(greenFrequency, XX, XX, 255, 0);
  
  // Printing the GREEN (G) value  
  delay(100);
 
  // Setting BLUE (B) filtered photodiodes to be read
  digitalWrite(S2,LOW);
  digitalWrite(S3,HIGH);
  
  // Reading the output frequency
  blueFrequency = pulseIn(SensorOut, LOW);
  // Remaping the value of the BLUE (B) frequency from 0 to 255
  // You must replace with your own values. Here's an example: 
  // blueColor = map(blueFrequency, 38, 84, 255, 0);
  //blueColor = map(blueFrequency, XX, XX, 255, 0);
  
  // Printing the BLUE (B) value 
  delay(100);
}
