/******************************************************************************
* Copyright (C) 2002 - 2020 Xilinx, Inc.  All rights reserved.
* Copyright (C) 2022 - 2023 Advanced Micro Devices, Inc. All Rights Reserved.
* SPDX-License-Identifier: MIT
******************************************************************************/

/*****************************************************************************/
/**
* @file  xtmrctr_low_level_example.c
*
* This file contains a design example using the Timer Counter (XTmrCtr)
* low level driver and hardware device in a polled mode.
*
* @note
*
* None
*
* <pre>
* MODIFICATION HISTORY:
*
* Ver   Who  Date	 Changes
* ----- ---- -------- -----------------------------------------------
* 1.00b jhl  02/13/02 First release
* 1.00b sv   04/26/05 Minor changes to comply to Doxygen and coding guidelines
* 2.00a ktn  10/30/09 Updated the example as the macros in the driver are
*                     renamed by removing _m in the definition.
*                     Minor changes as per coding guidelines are done.
* 4.2   ms   01/23/17 Added xil_printf statement in main function to
*                     ensure that "Successfully ran" and "Failed" strings
*                     are available in all examples. This is a fix for
*                     CR-965028.
*</pre>
******************************************************************************/

#include "xstatus.h"
#include "xtmrctr_l.h"
#include "xil_printf.h"
#include "xparameters.h"



#ifndef SDT
#define TMRCTR_BASEADDR		XPAR_TMRCTR_0_BASEADDR
#else
#define TMRCTR_BASEADDR		XPAR_XTMRCTR_0_BASEADDR
#endif


#define TIMER_COUNTER_0	 0

//Indirizzi GPIO
volatile int * gpio_0_data = (volatile int*) 0x40000000;
volatile int * gpio_0_tri  = (volatile int*) 0x40000004;

//Indirizzi di controllo Interrupt
volatile int * IER = (volatile int*)	0x41200008;
volatile int * MER = (volatile int*)	0x4120001C;
volatile int * IISR = (volatile int*)	0x41200000;
volatile int * IIAR = (volatile int*)	0x4120000C;


void myISR(void) __attribute__((interrupt_handler));
int TmrCtrLowLevelExample(UINTPTR TmrCtrBaseAddress, u8 TimerCounter);

volatile int led_state = 0; //Variabile per il controllo dei led

int main(void)
{
	int Status;
	*gpio_0_tri = 0x00000000;
	*gpio_0_data = 0x00000000;

	//Configurazione interrupt controller
    *IER = XPAR_AXI_TIMER_0_INTERRUPT_MASK;//|XPAR_BUTTON_IP2INTC_IRPT_MASK|XPAR_GPIO_IP2INTC_IRPT_MASK;      // enable IRQ0 and IRQ1 (use 0x2 if only GPIO2 is wired)
    *MER = 0x3; //0b11     // ME | HIE

    microblaze_enable_interrupts();

    //Avvio del timer
	Status = TmrCtrLowLevelExample(TMRCTR_BASEADDR, TIMER_COUNTER_0);
	if (Status != XST_SUCCESS) {
		xil_printf("Tmrctr lowlevel Example Failed\r\n");
		return XST_FAILURE;
	}
	while(1);
	return XST_SUCCESS;

}

int TmrCtrLowLevelExample(UINTPTR TmrCtrBaseAddress, u8 TmrCtrNumber)
{
	u32 ControlStatus;
	float time_blink;
	/*
	 * Clear the Control Status Register
	 */
	XTmrCtr_SetControlStatusReg(TmrCtrBaseAddress, TmrCtrNumber, XTC_CSR_AUTO_RELOAD_MASK|XTC_CSR_ENABLE_INT_MASK|XTC_CSR_DOWN_COUNT_MASK);

	/*
	 * Set the value that is loaded into the timer counter and cause it to
	 * be loaded into the timer counter
	 */

	//IMPOSTA QUI IL VALORE DEI SECONDI PER CUI I LED DEVONO LAMPEGGIARE (Es 2 sec, 2.5 sec)
	time_blink=1; //Ora ho impostato 1 secondo
	XTmrCtr_SetLoadReg(TmrCtrBaseAddress, TmrCtrNumber, time_blink*100000000);
	XTmrCtr_LoadTimerCounterReg(TmrCtrBaseAddress, TmrCtrNumber);

	/*
	 * Clear the Load Timer bit in the Control Status Register
	 */
	ControlStatus = XTmrCtr_GetControlStatusReg(TmrCtrBaseAddress,
			TmrCtrNumber);
	XTmrCtr_SetControlStatusReg(TmrCtrBaseAddress, TmrCtrNumber,
				    ControlStatus & (~XTC_CSR_LOAD_MASK));
	/*
	 * Start the timer counter such that it's incrementing by default
	 */
	XTmrCtr_Enable(TmrCtrBaseAddress, TmrCtrNumber);

	return XST_SUCCESS;
}

void myISR(void)
{
    unsigned p = *IISR;  // snapshot
    // Controlla se è il timer 0
    if (p & XPAR_AXI_TIMER_0_INTERRUPT_MASK) {               // BTN
    	int ControlStatus = XTmrCtr_GetControlStatusReg(TMRCTR_BASEADDR,0);
    	XTmrCtr_SetControlStatusReg(TMRCTR_BASEADDR, 0,
    				    ControlStatus | XTC_CSR_INT_OCCURED_MASK);   // clear device first
        *IIAR  = XPAR_AXI_TIMER_0_INTERRUPT_MASK; //Ack          // then ack INTC
        //Progetto precedente: accendevo e spegnevo entrambi i led contemporaneamente
        //*((int*)gpio_0_data)= ~(*((int*)gpio_0_data));

        //Con questa implementazione accendo i led in alternanza
        if(led_state==0x0) led_state = 0x1;
        else if(led_state==0x1) led_state = 0x2;
        else led_state = 0x1;
        *((int *)gpio_0_data) = led_state;
    }
}
