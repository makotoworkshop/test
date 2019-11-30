#define relaisMOSFET 0            //PB0, pin5
#define Bouton 2                  //PB2, pin7
#define TXD0 3                    //PB3, pin2
#define BoutonGPIO21 1            //PB1, pin6
#define STATE_NORMAL 0            // no button activity
#define STATE_SHORT 1             // short button press
#define STATE_LONG 2              // long button press
volatile int  resultButton = 0;   // global value set by checkButton()
int EtatTXD0;
int EtatBouton;
int var = 0;

void setup() {
  attachInterrupt(0, checkButton, CHANGE);
  pinMode(Bouton, INPUT_PULLUP);
//  pinMode(Bouton, INPUT);
  pinMode(TXD0, INPUT);
  pinMode(BoutonGPIO21, OUTPUT);
  pinMode(relaisMOSFET, OUTPUT);
  digitalWrite(relaisMOSFET, HIGH); // power off
}

/*****************************************************************/
void checkButton() {                                    // appelée par interruption à l'appuie sur le bouton
  /*
  * This function implements a short press and a long press and identifies between
  * the two states. Your sketch can continue processing while the button
  * function is driven by pin changes.
  * https://www.electro-tech-online.com/threads/dual-state-pushbutton-debounced-using-interrupts-for-arduino.147069/
  * Modifié par MakotoWorkshop
  */
  const unsigned long LONG_DELTA = 3000ul;              // hold seconds for a long press
  const unsigned long SHORT_DELTA = 0ul;                // hold seconds for a long press

  static int lastButtonStatus = HIGH;                   // HIGH indicates the button is NOT pressed
  int buttonStatus;                                     // button atate Pressed/LOW; Open/HIGH
  static unsigned long longTime = 0ul, shortTime = 0ul; // future times to determine is button has been poressed a short or long time
  boolean Released = true, Transition = false;          // various button states
  boolean timeoutShort = false, timeoutLong = false;    // flags for the state of the presses

  buttonStatus = digitalRead(Bouton);                   // read the button state on the pin "Bouton"
  timeoutShort = (millis() > shortTime);                // calculate the current time states for the button presses
  timeoutLong = (millis() > longTime);

  if (buttonStatus != lastButtonStatus) {               // reset the timeouts if the button state changed
      shortTime = millis() + SHORT_DELTA;
      longTime = millis() + LONG_DELTA;
  }

  Transition = (buttonStatus != lastButtonStatus);      // has the button changed state
  Released = (Transition && (buttonStatus == HIGH));    // for input pullup circuit

  lastButtonStatus = buttonStatus;                      // save the button status

  if ( ! Transition) {                                  //without a transition, there's no change in input
                                                        // if there has not been a transition, don't change the previous result
       resultButton =  STATE_NORMAL | resultButton;
       return;
  }

  if (timeoutLong && Released) {                        // long timeout has occurred and the button was just released
       resultButton = STATE_LONG | resultButton;        // ensure the button result reflects a long press
  } else if (timeoutShort && Released) {                // short timeout has occurred (and not long timeout) and button was just released
      resultButton = STATE_SHORT | resultButton;        // ensure the button result reflects a short press
  } else {                                              // else there is no change in status, return the normal state
      resultButton = STATE_NORMAL | resultButton;       // with no change in status, ensure no change in button status
  }
}
/*****************************************************************/

void loop() {
  SoftPowerONOFF();
  HardPowerOFF();   // Coupe l'alim brutalement
  delay(170);       // wait for a second
}

/*****************************************************************/
void SoftPowerONOFF() {
  EtatTXD0 = digitalRead(TXD0);

  if ((EtatTXD0 == LOW) && (resultButton == STATE_SHORT)) //POWER ON via bouton
  {
    digitalWrite(relaisMOSFET, LOW);                      // power on    //PB0, pin5
    resultButton=STATE_NORMAL;
  }
}

/*****************************************************************/
void HardPowerOFF() {
  if ((resultButton == STATE_LONG))     // appuie long sur le bouton (3 sec)
  {
    digitalWrite(relaisMOSFET, HIGH);  //power off
    resultButton=STATE_NORMAL;
  }
}
