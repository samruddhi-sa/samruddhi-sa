exp led buzzer
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



