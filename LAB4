
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
#include "xil_printf.h"
#include "xtmrctr_l.h"
#include "xstatus.h"

#ifndef SDT
#define TMRCTR_BASEADDR		XPAR_TMRCTR_0_BASEADDR
#else
#define TMRCTR_BASEADDR		XPAR_XTMRCTR_0_BASEADDR
#endif
#define TIMER_COUNTER_0	 0

// Maschera per capire se il timer è scaduto
#define CSR_INT_OCCURRED_MASK 0x00000100

//Indirizzi dei due led
volatile int * gpio_leds_data = (volatile int*) 0x40000000;
volatile int * gpio_leds_tri  = (volatile int*) 0x40000004;

//Indirizzi bottone Onboard
volatile int * btn_onboard_data = (volatile int *) 0x40060000;
volatile int * btn_onboard_tri  = (volatile int *) (0x40060004);


//Indirizzi bottone esterno
volatile int * btn_esterno_data = (volatile int *) 0x40050000;
volatile int * btn_ensterno_tri  = (volatile int*) 0x40050004;

//Variabili globali
int flash_state = 0; // 1 = Luce, 0 = Buio
int ticks = 0;       // Contatore per rallentare il lampeggio


//Dichiarazione degli stati per la FSM
typedef enum {pressed, idle} debounce_state_t;
typedef enum {left, centre, right} direzione_led;

//Dichiarazione funzioni
void FSM_leds(int button_right,int button_left);
int FSM_debounce_L(int buttons); // Funzione dedicata Sinistra
int FSM_debounce_R(int buttons); // Funzione dedicata Destra

int main(void)
{

	init_platform();

	*gpio_leds_tri = 0x00000000; // LED = OUTPUT (0)
	*btn_onboard_tri = 0xFFFFFFFF; // Btn1 = INPUT (1)
	*btn_ensterno_tri = 0xFFFFFFFF; // Btn2 = INPUT (1)

    //Configura timer
    XTmrCtr_SetControlStatusReg(TMRCTR_BASEADDR, TIMER_COUNTER_0, 0);
    XTmrCtr_SetLoadReg(TMRCTR_BASEADDR, TIMER_COUNTER_0, 3000000); //Da qui decido il blind del led
    XTmrCtr_LoadTimerCounterReg(TMRCTR_BASEADDR, TIMER_COUNTER_0);

    u32 ControlStatus = XTC_CSR_AUTO_RELOAD_MASK | XTC_CSR_ENABLE_INT_MASK | XTC_CSR_DOWN_COUNT_MASK;
    XTmrCtr_SetControlStatusReg(TMRCTR_BASEADDR, TIMER_COUNTER_0, ControlStatus);
    XTmrCtr_Enable(TMRCTR_BASEADDR, TIMER_COUNTER_0);

	//Bottone
	int raw_L, raw_R;
	int click_L, click_R;

    while (1) {

    	u32 timer_status = XTmrCtr_GetControlStatusReg(TMRCTR_BASEADDR, TIMER_COUNTER_0);

    	if (timer_status & CSR_INT_OCCURRED_MASK) {
    		XTmrCtr_SetControlStatusReg(TMRCTR_BASEADDR, TIMER_COUNTER_0, timer_status | CSR_INT_OCCURRED_MASK);

    		// Fai avanzare il tempo
    		ticks++;
    		if (ticks > 20) { // Regola velocità lampeggio qui
    			ticks = 0;
    			flash_state = !flash_state; // Toggle lampeggio
    		}
        	//Lettura bottoni
        	raw_L = *btn_onboard_data & 0x1;
        	raw_R = *btn_esterno_data & 0x1;

        	//debounce dei bottoni
        	click_L = FSM_debounce_L(raw_L);
        	click_R = FSM_debounce_R(raw_R);
        	//Richiamo alla funzione per spostarmi di stato a seconda del bottone premuto
        	FSM_leds(click_R,click_L);
    	}

    }
    cleanup_platform();
    return 0;
}

//Ho dovuto dividere il debounce poiché non potevo usare lo stesso per due diversi bottoni

//Debounce del pulsante di Sinistra (Interno)
int FSM_debounce_L(int buttons){
	static int debounced_buttons = 0;
	static debounce_state_t currentState = idle;
	switch (currentState) {
		case idle:
			debounced_buttons = buttons;
			if (buttons != 0) currentState = pressed;
			break;
		case pressed:
			debounced_buttons = 0;
			if (buttons == 0) currentState = idle;
			break;
	}
    return debounced_buttons;
}

// Debounce del pulsante di Destra (Esterno)
int FSM_debounce_R(int buttons){
	static int debounced_buttons = 0;
	static debounce_state_t currentState = idle;
	switch (currentState) {
		case idle:
			debounced_buttons = buttons;
			if (buttons != 0) currentState = pressed;
			break;
		case pressed:
			debounced_buttons = 0;
			if (buttons == 0) currentState = idle;
			break;
	}
    return debounced_buttons;
}
void FSM_leds(int button_right,int button_left){
	int led_val = 0;
	static direzione_led current_state = centre;
	//Logica della FMS
	switch (current_state){
	case centre:
		if(button_right){
			current_state=right;
		}else if(button_left){
			current_state=left;
		}
		break;
	case right:
		if(button_left){
			current_state=left;
		}else if(button_right){
			current_state=centre;
		}
		break;
	case left:
		if(button_left){
			current_state=centre;
		}else if(button_right){
			current_state=right;
		}
		break;
		}
	led_val = 0;
	//Stabilisco quale led accendere a seconda del bottone premuto
	if (current_state == left && flash_state)  led_val = 0x2;
	if (current_state == right && flash_state) led_val = 0x1;
	*gpio_leds_data = led_val;
}
