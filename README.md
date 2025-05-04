exp 5 led relay buzzer
#include<P18F4550.h>

void delay()
{
	unsigned int i;
	for(i=0;i<30000;i++);
}

void main()
{
    unsigned char i, key = 0;

 TRISB = 0x00;                        //LED pins as output
    PORTB = 0x00;

TRISDbits.TRISD0 = 1;                   //set RD0 as input
    TRISDbits.TRISD1 = 1;                   //set RD1 as input

TRISDbits.TRISD2 = 0;       //set buzzer pin RD2 as output
    TRISAbits.TRISA4 = 0;        //set relay pin RA4 as output

  while(1)
    {

 if(PORTDbits.RD0 == 0) key =0;   //If button1 pressed
if(PORTDbits.RD1 == 0) key =1;    //If button2 pressed

 if(key == 0)
        {
            PORTAbits.RA4 = 1;             //Relay OFF
            PORTDbits.RD2 = 0;             //Buzzer OFF
            for(i=0;i<8;i++)         //Chase LED right to left
            {
                PORTB = 1<<i;
                delay();
                PORTB = 0x00;
                delay();
            }
        }
        if(key == 1)
        {
            PORTAbits.RA4 = 0;             //Relay ON
            PORTDbits.RD2 = 1;             //Buzzer ON
            for(i=7;i> 0;i--)               //Chase LED left to right
            {
                PORTB = 1<<i;
                delay();
                PORTB = 0x00;
                delay();
            }
        }

    }
}




exp 1
ORG 0000H
MOV R1,#05		;Counter
MOV R0,#30H		;Source
MOV DPTR,#2000H ;Destination
back:MOV A,@R0
	 MOVX @DPTR,A	;A->ext destn
	 INC R0			;Source increment
	 INC DPTR		;destination increment
	 DJNZ R1,back

END






//external code memory to internal RAM memory data transfer

ORG 0000H

MOV R1,#05		;Counter

MOV R0,#40H		;Source

MOV DPTR,#2000H ; external Destination

back:MOVC A,@ A+DPTR    ;external destination -> A
	MOV @R0,A	;A->int destn
	 INC R0			;Source increment
	 INC DPTR		;destination increment
         CLR A
	 DJNZ R1,back
END


EXP 1 INTERNAL TO INTERNAL

ORG 0000H
MOV R0, #40H
MOV R1, #60H
MOV R2, #10
BASIC:MOVX A, @R0
MOV @R1, A
INC R0
INC R1
DJNZ R2, BASIC
End





EXP 6 LCD

#include <p18f4550.h>

#define LCD_EN LATCbits.LC1
#define LCD_RS LATCbits.LC0
#define LCDPORT LATB


void lcd_delay(unsigned int time)
{
 unsigned int i , j ;

    for(i = 0; i < time; i++)
    {
            for(j=0;j<100;j++);
    }
}


void SendInstruction(unsigned char command)
{
     LCD_RS = 0;		// RS low : Instruction
     LCDPORT = command;
     LCD_EN = 1;		// EN High
     lcd_delay(10);
     LCD_EN = 0;		// EN Low; command sampled at EN falling edge
     lcd_delay(10);
}

void SendData(unsigned char lcddata)
{
     LCD_RS = 1;		// RS HIGH : DATA
     LCDPORT = lcddata;
     LCD_EN = 1;		// EN High
     lcd_delay(10);
     LCD_EN = 0;		// EN Low; data sampled at EN falling edge
     lcd_delay(10);
}

void InitLCD(void)
{
    TRISB = 0x00; //set data port as output
    TRISCbits.RC0 = 0; //EN pin
    TRISCbits.RC1 = 0; // RS pin

    SendInstruction(0x38);      //8 bit mode, 2 line,5x7 dots
    SendInstruction(0x06);	// entry mode
    SendInstruction(0x0C);	//Display ON cursor OFF
    SendInstruction(0x01);      //Clear display
    SendInstruction(0x80);      //set address to 1st line
    
}
/********************************************************************************************************************/

unsigned char *String1 = "Micro-PIC Board";
unsigned char *String2 = " MicroEmbedded";

void main(void)
{
    TRISB = 0x00; //set data port as output
    TRISCbits.RC0 = 0; //EN pin
    TRISCbits.RC1 = 0; // RS pin

  SendInstruction(0x38);      //8 bit mode, 2 line,5x7 dots
    SendInstruction(0x06);	// entry mode
    SendInstruction(0x0C);	//Display ON cursor OFF
    SendInstruction(0x01);      //Clear display
    SendInstruction(0x80);      //set address to 1st line

 while(*String1)
 {
  SendData(*String1);
  String1++;
 }

 SendInstruction(0xC0);      //set address to 2nd line
 while(*String2)
 {
  SendData(*String2);
  String2++;
 }

 while(1);
 
}




EXP 7 SQUAREWAVE USING TIMER INTERRUPT


#include<p18f4550.h>

volatile bit timer_set = 0;

void timerInit(void)
{
     // Timer0 configuration
    T0CON = 0b00000111;                     //Timer0 16-bit; prescaler 1:256
    TMR0H = 0xED;
    TMR0L = 0xB0;
}

void Interrupt_Init(void)
{
    RCONbits.IPEN = 1;
    INTCON = 0b11100000;                    //Enable global and Timer0 interrupts; Clear Timer0 interrupt flag
    INTCON2bits.TMR0IP = 0;
}

void interrupt low_priority timerinterrupt(void)
{
    if(TMR0IF == 1)                         //If timer0 interrupt flag is set.....
    {
      TMR0ON = 0;                           // Stop the timer
      TMR0IF = 0;                           //Clear the interrupt flag
      TMR0H = 0xED;                         //Reload Timer0
      TMR0L = 0xB0;
      LATB =~LATB;                          //Toggle PORTB
      TMR0ON = 1;                           // Start the timer
    }
}


void main(void)
{
   TRISB = 0x00;
   LATB = 0xFF;

   Interrupt_Init();
   timerInit();
   TMR0ON = 1;                              // Start the timer

   while(1);                                //Loop forever; do nothing
}








