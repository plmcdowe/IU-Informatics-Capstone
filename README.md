# IU Informatics, BS.
## [ \* Capstone Project: "ParkIU" ](https://github.com/plmcdowe/School/tree/1228ab2c2261ae7d5b3b14264a321303cdc0361b/IU-Informatics-Capstone)
My contributions to a project which created an iOS application to track availability of on campus parking.
> 
> Two Particle Photon's fitted with sonar sensors programmed to increment/decrement parking facility occupancy:
>> <img src="https://github.com/user-attachments/assets/638bd649-03e6-41b0-be4a-783cfbbe8448" alt="Alt Text" width="400" height="263">
> Changes to occupancy are published by json web-hooks to a PHP script which updates a SQL datatbase.   
>> ## [ ParticleA.ino ](https://github.com/plmcdowe/IU-Informatics-Capstone/blob/da713ac993d08bc1d79b0551831f399e152470bc/ParticleA.ino) decrements on vehicle exit and publishes the event to ParticleB.
>> ```C#
>> #define echoPin D6 // Echo Pin
>> #define trigPin D2 // Trigger Pin
>> 
>> int maximumRange = 5; // Maximum range for reading
>> int minimumRange = 0; // Minimum range for reading
>> unsigned int cars = 100; //Number of spaces available in garage
>> long duration, distance; // Duration used to calculate distanc
>> 
>> char val[40];
>> 
>> void setup() {
>>   Particle.subscribe("sub", myHandler);
>>   Serial.begin (9600);
>>   pinMode(trigPin, OUTPUT);
>>   pinMode(echoPin, INPUT);
>> }
>> 
>> //void handler that listens for "trig" from ParticleB; when "trig" recieved, count++ in cars
>> void myHandler(const char *event, const char *data) {
>>     
>>   if (strcmp(data, "trig")==0) {
>>     cars++;
>>     sprintf(val, "%u", cars);
>>     Particle.publish("count", val, PUBLIC);
>>   }
>> }
>> 
>> //void loop that watches for signal from sensor, when the sensor is tripped, count is reduced by one.
>> void loop() {
>>   digitalWrite(trigPin, LOW);
>>   delayMicroseconds(2);
>> 
>>   digitalWrite(trigPin, HIGH);
>>   delayMicroseconds(10); 
>> 
>>   digitalWrite(trigPin, LOW);
>>   duration = pulseIn(echoPin, HIGH);
>>   distance = duration/58.2;
>> 
>>   if (distance <= maximumRange){
>>     cars --;
>>     sprintf(val, "%u", cars);
>>     Particle.publish("count", val, PUBLIC);
>>   }
>>  
>>   //Delay in ms before next reading --> 10 second.
>>   delay(10000);
>> }
>> ```
>>
---   
>> ## [ ParticleB.ino ](https://github.com/plmcdowe/IU-Informatics-Capstone/blob/da713ac993d08bc1d79b0551831f399e152470bc/ParticleB.ino) increments on vehicle entrance and publishes all events to the Particle cloud.
>> ```C#
>> #define echoPin D6 // Echo Pin
>> #define trigPin D2 // Trigger Pin
>> 
>> int maximumRange = 5; // Maximum range for reading
>> int minimumRange = 0; // Minimum range for reading
>> long duration, distance; // Duration used to calculate distance
>> 
>> void setup() {
>>   Serial.begin (9600);
>>   pinMode(trigPin, OUTPUT);
>>   pinMode(echoPin, INPUT);
>> }
>> 
>> //void loop that watches for signal from sensor, when the sensor is tripped, Particle.publish sends "trig"; making PartcleA add to count.
>> void loop() {
>>   digitalWrite(trigPin, LOW);
>>   delayMicroseconds(2);
>> 
>>   digitalWrite(trigPin, HIGH);
>>   delayMicroseconds(10); 
>> 
>>   digitalWrite(trigPin, LOW);
>>   duration = pulseIn(echoPin, HIGH);
>>   distance = duration/58.2;
>> 
>>   if (distance <= maximumRange){
>>  
>>   Particle.publish("sub", "trig");
>>   }
>> 
>> //Delay in ms before next reading --> 10 second.
>> delay(10000);
>> } 
>> ```
>>
---   
>> ## [ count.php ](https://github.com/plmcdowe/IU-Informatics-Capstone/blob/da713ac993d08bc1d79b0551831f399e152470bc/count.php) recieves the json formatted web-hook from the Particle cloud and updates the SQL database.   
>> ```php
>> <?php
>> $con=mysqli_connect("127.0.0.1","user_name", "pass_word", "parking", 3306);
>> 
>> $count = $_GET['count'];
>> $num = (int)$count;
>> 
>> $sql="UPDATE garages SET count = '$num' WHERE id = '1'";
>> 
>> if (!mysqli_query($con,$sql))
>> { die('Error: ' . mysqli_error($con)); }
>> 
>> mysqli_close($conn);
>> ?>
>> ```
>>
---    
