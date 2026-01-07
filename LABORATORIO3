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

/***************************** Include Files *********************************/

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

/***************************** Include Files *********************************/

#include "xstatus.h"
#include "xtmrctr_l.h"
#include "xil_printf.h"
#include "xparameters.h"

/************************** Constant Definitions *****************************/

/*
 * The following constants map to the XPAR parameters created in the
 * xparameters.h file. They are only defined here such that a user can easily
 * change all the needed parameters in one place.
 */
#ifndef SDT
#define TMRCTR_BASEADDR		XPAR_TMRCTR_0_BASEADDR
#else
#define TMRCTR_BASEADDR		XPAR_XTMRCTR_0_BASEADDR
#endif

#define TIMER_COUNTER_0	 0

/***************** Macros (Inline Functions) Definitions *********************/
volatile int * gpio_rgb_data = (volatile int *) (0x40000008);
volatile int * gpio_rgb_tri  = (volatile int *) (0x4000000C);

//Controller dei registri interrupt
volatile int * IER = (volatile int*)	0x41200008;
volatile int * MER = (volatile int*)	0x4120001C;
volatile int * IISR = (volatile int*)	0x41200000;
volatile int * IIAR = (volatile int*)	0x4120000C;
//Variabili globali pwm RGB
volatile u8 R = 0;
volatile u8 G = 0;
volatile u8 B = 0;
volatile int pwm_count = 0;

/************************** Function Prototypes ******************************/
void myISR(void) __attribute__((interrupt_handler));
int TmrCtrLowLevelExample(UINTPTR TmrCtrBaseAddress, u8 TimerCounter);
void pwm(int data);

int main(void)
{
	int Status;
	int data;
	//Imposto il tri e inizializzo rgb_data
	*gpio_rgb_tri = 0x00000000; // Output (Tutti i pin del canale 2)
	*gpio_rgb_data = 0x0;
	/*
	 * Run the Timer Counter - Low Level example .
	 */
    *IER = XPAR_AXI_TIMER_0_INTERRUPT_MASK;//|XPAR_BUTTON_IP2INTC_IRPT_MASK|XPAR_GPIO_IP2INTC_IRPT_MASK;      // enable IRQ0 and IRQ1 (use 0x2 if only GPIO2 is wired)
    *MER = 0x3; //0b11     // ME | HIE
    //INSERISCI QUI in "data" un numero da 0 a 9 per cambiare colore
    data=3;
    //Richiamo la funzione che fa la PWM;
    pwm(data);

    microblaze_enable_interrupts();

    //Avvio il timer
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

	XTmrCtr_SetControlStatusReg(TmrCtrBaseAddress, TmrCtrNumber, XTC_CSR_AUTO_RELOAD_MASK|XTC_CSR_ENABLE_INT_MASK|XTC_CSR_DOWN_COUNT_MASK);
	XTmrCtr_SetLoadReg(TmrCtrBaseAddress, TmrCtrNumber, 2000);
	XTmrCtr_LoadTimerCounterReg(TmrCtrBaseAddress, TmrCtrNumber);
	ControlStatus = XTmrCtr_GetControlStatusReg(TmrCtrBaseAddress,
			TmrCtrNumber);
	XTmrCtr_SetControlStatusReg(TmrCtrBaseAddress, TmrCtrNumber,
				    ControlStatus & (~XTC_CSR_LOAD_MASK));
	XTmrCtr_Enable(TmrCtrBaseAddress, TmrCtrNumber);
	return XST_SUCCESS;
}

void pwm(int data){
	//In base al dato ottenuto, ottengo un colore diverso nel led RGB
    switch(data){
    	case 0: R=255; G=255; B=255; break; //Spento
    	case 1: R=255; G=0; B=0; break;
    	case 2: R=0; G=255; B=0; break;
    	case 3: R=0; G=0; B=255; break;
    	case 4: R=255; G=255; B=0; break;
    	case 5: R=0; G=255; B=255; break;
    	case 6: R=255; G=0; B=255; break;
    	case 7: R=255; G=128; B=0; break;
    	case 8: R=128; G=0; B=255; break;
    	case 9: R=0; G=0; B=0; break; //Tutte accese
    }

}
void myISR(void)
{

    unsigned p = *IISR;  // snapshot
    if (p & XPAR_AXI_TIMER_0_INTERRUPT_MASK) {
        pwm_count++;
        u8 count_byte = (u8)pwm_count;
        int rgb_out = 0;
        // Gestione PWM
        if (count_byte < B) rgb_out |= 0x1;
        if (count_byte < G) rgb_out |= 0x2;
        if (count_byte < R) rgb_out |= 0x4;

        *gpio_rgb_data = rgb_out;

    	int ControlStatus = XTmrCtr_GetControlStatusReg(TMRCTR_BASEADDR,0);
    	XTmrCtr_SetControlStatusReg(TMRCTR_BASEADDR, 0,
    				    ControlStatus | XTC_CSR_INT_OCCURED_MASK);   // clear device first
        *IIAR  = XPAR_AXI_TIMER_0_INTERRUPT_MASK;           // then ack INTC

    }
}
