/*
 * fanproj1.c
 *
 * Created: 12/09/2017 2:28:46 p.m.
 * Author : xyu700
 */ 

 //#define F_CPU 8000000UL
 //#define BAUD 9600
 #define TOP 0xFF


#include <avr/io.h>
#include <avr/interrupt.h>

void run_the_fan();
void PWM_generate();

double DutyCycle = 0.8;

void PWM_generate(){
           DDRA |= (1<<DDA2)|(1<<DDA3 ); //Set PA2 and PA3 as output
		   //set PB1 as input (for hall-effect sensor)
		   DDRA &= ~(1<<DDB1);
           //setup timer
		   TCCR0A |= (1<<WGM01)|(1<<WGM00);// set fast PWM
           TCCR0B |= (1 << CS00); //8MHz no prescaling
           //TCCR0A |= (1 << COM0A1); //clear OC0A on compare match with OCR0A
		   //TCCR0B |= (1 << COM0B1); //clear OC0B on compare match with OCR0B

		   // setting pwm to be non-inverted (clear on compare match, set high on overflow/BOTTOM)
		   TCCR0A |= (1<<COM0A1) | (1<<COM0B1);
		   //TCCR0A &= ~(1<<COM0A0);
		   //TCCR0A |= (1<<COM0B1); 
		   //TCCR0A &= ~(1<<COM0B0);

            //reset counter register
			TCNT0 = 0;

		   OCR0A = (TOP+1)*DutyCycle; //Value to compare with for PA2
		   OCR0B = (TOP+1)*DutyCycle; //Value to compare with for PA3
           TOCPMSA0 &= ~((1<<TOCC1S1)| ~(1<<TOCC1S0)); //route OC0A to TOCC1(PA2)
		   TOCPMSA1 &= ~((1<<TOCC2S1)| ~(1<<TOCC2S0)); //route OC0B to TOCC2(PA3)
          // TIMSK0 |= (1<<TOIE0); //enable timer overflow interrupt
           //sei(); //enable interrupt
		   while (1){
		   if (PINB &(1<<PINB1) ) { //read hall sensor signal, if high
			   TOCPMCOE |= (1<<TOCC3OE); //enable TOCC1 at PA3
			   TOCPMCOE &= ~(1<<TOCC2OE); //disable TOCC1 at PA2
			   } else { //if hall sensor signal is low
			   TOCPMCOE |= (1<<TOCC2OE); //enable TOCC1 at PA2
			   TOCPMCOE &= ~(1<<TOCC3OE); //disable TOCC1 at PA3
			   }
			   }
}

//ISR(TIMER0_OVF_vect) {
	//PORTA |= (1<<PA2); //set HIGH on overflow
//}








int main(void)
{
	   //run_the_fan();
	   PWM_generate();
    
}




void run_the_fan(){

 //set PA2 and PA3 as output (to drive the switches)
 DDRA |= (1<<DDA2)|(1<<DDA3 );
 //set PB1 as input (for hall-effect sensor)
 DDRA &= ~(1<<DDB1);
 while(1){
	 if (PINB &(1<<PINB1) ) { //read hall sensor signal, if high
		 PORTA |= (1<<PORTA2 ); //turn on one of the switch
		 PORTA &= ~(1<<PORTA3 ); //turn off the other switch
		 } else { //if hall sensor signal is low
		 PORTA &= ~(1<<PORTA2 ); //turn off one of the switch
		 PORTA |= (1<<PORTA3); //turn on the other
	 }
 }

}
