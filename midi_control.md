// noise-box
// One-in-one-out project for 60-223

//set the MIDI commands
int noteON[] = {144, 145, 146, 147, 148, 149}; //144 = 10010000 in binary, note on command
int noteOFF[] = {128, 129, 130, 131, 132, 133}; //128 = 10000000 in binary, note off command

//choose 6 notes, and corresponding velocities
int note[] = {75, 78, 70, 81, 68, 65};
int velocity[] = {100, 100, 100, 80, 110, 110};

int sensor[6];
int sensor_start[6];
int bend[6];

int button1 = 13;
int button1_state = 0;
int newbutton1_state;

void setup() {
  pinMode(button1, INPUT);
  //  Set MIDI baud rate:
  Serial.begin(115200);
  update_sensor();
  for (int i = 0; i < 6; i++) {
    sensor_start[i] = sensor[i]; // set initial sensor values
  }
}

void loop() {
  // check if the button was pushed
  newbutton1_state = digitalRead(button1);
  if (newbutton1_state != button1_state) {
    button1_state = newbutton1_state;
    if (button1_state == 0) {
      for (int i = 0; i < 6; i++) {
        MIDImessage(noteOFF[i], note[i], velocity[i]);
      }
    }
    else { //button1_state == 1
      for (int i = 0; i < 6; i++) {
        MIDImessage(noteON[i], note[i], velocity[i]);
      }
    }
    update_sensor();
    for (int i = 0; i < 6; i++) {
      sensor_start[i] = sensor[i];
    }
  }
  //send pitch bend information while the button is on
  if (button1_state == 1) {
    update_sensor();
    for (int i = 0; i < 6; i++) {
      bend[i] = (sensor_start[i] - sensor[i]) / 2;
      if (bend[i] < -63) {
        bend[i] = -63;
      }
      if (bend[i] > 63) {
        bend[i] = 63;
      }
      MIDIpitchbend(i, 0, bend[i]);
    }
  }
}

//update sensor values
void update_sensor() {
  sensor[0] = analogRead(A0);
  sensor[1] = analogRead(A1);
  sensor[2] = analogRead(A2);
  sensor[3] = analogRead(A3);
  sensor[4] = analogRead(A4);
  sensor[5] = analogRead(A5);
}

//send MIDI pitch bend message
void MIDIpitchbend(int channel, int LSB, int MSB) {
  Serial.write(224 + channel);
  Serial.write(LSB);//always use 0
  Serial.write(64 + MSB);//a value of 64 is no bending
}

//send MIDI message
void MIDImessage(int command, int MIDInote, int MIDIvelocity) {
  Serial.write(command);//send note on or note off command 
  Serial.write(MIDInote);//send pitch data
  Serial.write(MIDIvelocity);//send velocity data
}
