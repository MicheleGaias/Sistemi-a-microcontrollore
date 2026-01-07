/******************************************************************************
*
* Copyright (C) 2009 - 2014 Xilinx, Inc.  All rights reserved.
*
* Permission is hereby granted, free of charge, to any person obtaining a copy
* of this software and associated documentation files (the "Software"), to deal
* in the Software without restriction, including without limitation the rights
* to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
* copies of the Software, and to permit persons to whom the Software is
* furnished to do so, subject to the following conditions:
*
* The above copyright notice and this permission notice shall be included in
* all copies or substantial portions of the Software.
*
* Use of the Software is limited solely to applications:
* (a) running on a Xilinx device, or
* (b) that interact with a Xilinx device through a bus or interconnect.
*
* THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
* IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
* FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL
* XILINX  BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY,
* WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF
* OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
* SOFTWARE.
*
* Except as contained in this notice, the name of the Xilinx shall not be used
* in advertising or otherwise to promote the sale, use or other dealings in
* this Software without prior written authorization from Xilinx.
*
******************************************************************************/

/*
 * helloworld.c: simple test application
 *
 * This application configures UART 16550 to baud rate 9600.
 * PS7 UART (Zynq) is not initialized by this application, since
 * bootrom/bsp configures it to baud rate 115200
 *
 * ------------------------------------------------
 * | UART TYPE   BAUD RATE                        |
 * ------------------------------------------------
 *   uartns550   9600
 *   uartlite    Configurable only in HW design
 *   ps7_uart    115200 (configured by bootrom/bsp)
 */

#include <stdio.h>
#include "platform.h"
#include "xparameters.h"

volatile int * gpio_0_data = (volatile int*) 0x40000000;
volatile int * gpio_0_tri  = (volatile int*) 0x40000004;

//Bottone onboard
volatile int * btn_onboard_data = (volatile int *) 0x40060000;
volatile int * btn_onboard_tri  = (volatile int *) (0x40060004);
volatile int * GGIER_1        = (volatile int*) 0x4006011C;
volatile int * GIER_1         = (volatile int*) 0x40060128;
volatile int * GISR_1         = (volatile int*) 0x40060120;

//Bottone esterno
volatile int * btn_esterno_data = (volatile int *) 0x40050000;
volatile int * btn_ensterno_tri  = (volatile int*) 0x40050004;
volatile int * GGIER_2        = (volatile int*) (0x40050000 + 0x11C);
volatile int * GIER_2         = (volatile int*) (0x40050000 + 0x128);
volatile int * GISR_2         = (volatile int*) (0x40050000 + 0x120);

//Gestione interrupt
volatile int * IER = (volatile int*)	0x41200008;
volatile int * MER = (volatile int*)	0x4120001C;
volatile int * IISR = (volatile int*)	0x41200000;
volatile int * IIAR = (volatile int*)	0x4120000C;

//Variabile per gestione dell'accensione e spegnimento dei led
volatile int led_state = 0;

void myISR(void) __attribute__((interrupt_handler));

int main(void)
{
	init_platform();
    // Imposta la direzione dei dei registri (opzionale)
	* btn_onboard_tri = 0xFFFFFFFF; //Input
	* btn_ensterno_tri  = 0xFFFFFFFF; //Input
	* gpio_0_tri = 0x00000000; //Output
	//Si parte con i led spenti
	*gpio_0_data = 0x0;

	//Pulizia Interrupt, pulisce per eventuali interrupt pendenti
	*GISR_1 = 0xFFFFFFFF; // Pulisci stato bottone 1
	*GISR_2 = 0xFFFFFFFF; // Pulisci stato bottone 2
	*IIAR   = 0xFFFFFFFF; // Pulisci interrupt controller centrale

    //Abilitazione periferiche Interrupt (due canali)
    *GIER_1  = 0x1;
    *GGIER_1 = 0x80000000;
    *GIER_2 =0x1;
    *GGIER_2 = 0x80000000;

    //Abilità Interrupt controller (se da bottone onboard o esterno)
    *IER = XPAR_BUTTON_IP2INTC_IRPT_MASK | XPAR_GPIO_IP2INTC_IRPT_MASK;      // enable IRQ0 and IRQ1 (use 0x2 if only GPIO2 is wired)
    *MER = 0x3; //0b11     // ME | HIE

    //Abilitazione Interrupt
    microblaze_enable_interrupts();

    while (1) {
    	//Versione con POLLING (Commentata perché attiva interrupt)
/*    	    unsigned p = *IISR;  // Leggi chi ha "chiamato"
    //Gestione bottone onboard
    if (p & XPAR_BUTTON_IP2INTC_IRPT_MASK) {               // BTN
    	*GISR_1 = 0x1;    //Acknowledge, clear device first
        *IIAR   = XPAR_BUTTON_IP2INTC_IRPT_MASK;
        //Accende o spegne un led
        if(led_state==0x2) led_state = 0x0;// then ack INTC
        else led_state = 0x2;


    }
    //Gestione bottone esterno
    if(p & XPAR_GPIO_IP2INTC_IRPT_MASK){
        *GISR_2 = 0x1;    // clear device first
        *IIAR   = XPAR_GPIO_IP2INTC_IRPT_MASK ;
        //Accende o spegne l'altro led
        if(led_state==0x1) led_state = 0x0;// then ack INTC
        else led_state = 0x1;
    }
    //Aggiorna hardware
    *((int *)gpio_0_data) ^= led_state;*/
    }
}

//Funzione dell'INTERRUPT (Interrupt Service Routine)
void myISR(void)
{
    unsigned p = *IISR;  // Leggi chi ha "chiamato"
    //Gestione bottone onboard
    if (p & XPAR_BUTTON_IP2INTC_IRPT_MASK) {               // BTN
    	*GISR_1 = 0x1;    //Acknowledge, clear device first
        *IIAR   = XPAR_BUTTON_IP2INTC_IRPT_MASK;
        //Accende o spegne un led
        if(led_state==0x2) led_state = 0x0;// then ack INTC
        else led_state = 0x2;


    }
    //Gestione bottone esterno
    if(p & XPAR_GPIO_IP2INTC_IRPT_MASK){
        *GISR_2 = 0x1;    // clear device first
        *IIAR   = XPAR_GPIO_IP2INTC_IRPT_MASK ;
        //Accende o spegne l'altro led
        if(led_state==0x1) led_state = 0x0;// then ack INTC
        else led_state = 0x1;
    }
    //Aggiorna hardware
    *((int *)gpio_0_data) ^= led_state;
}
