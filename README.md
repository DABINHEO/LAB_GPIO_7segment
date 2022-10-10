

**Date:** 2022.10.10

**Author/Partner:** Heo Dabin/Ga Seungho

**Github:** [Github code](https://github.com/DABINHEO/LAB_GPIO_7segment.git)

**Demo Video:** [Youtube Link](https://youtube.com/shorts/-fcIKQPVV98?feature=share)

##            



### Introduction

In this lab, we will use nucleo-F401RE hardware and 7-segment display to output numbers from 0 to 9 each time a button is pressed. To this end, we used the eight pins of the three GPIOs of nucleo-F401RE to pass through the Array Register and properly input them into the 7-segment display. The button will use the one built into nucleo-F401RE and set PC13 to digital input and pull-up, and the pins connected to the 7-segment display set PA5,6,7,8,9 / PB6,10 / PC7 to digital output, push-pull, no pull-up-down, speed.

##           



### Requirement

#### Hardware

* MCU
  
  * NUCLEO-F401RE
  
* Actuator/Display/Others:
  * 7-segment display(5101ASR)
  * Array resistor (330 ohm)
* breadboard
  
  

#### Software

* Keil uVision, CMSIS, EC_HAL library

##          



### Problem1: Connecting 7-Segment



#### Procedure

In order to properly control the 7-segment display through the nucleo-F401RE, we connected it as shown in the following figure.

![image](https://user-images.githubusercontent.com/113574284/194786399-a00f83f5-bf4f-4730-bc97-52c293a49789.png)

<center>Figure1. Circuit of Hardware with 7-segment display</center>

And through several tests, it was confirmed that PA5 turned on the lights of e, PA6 turned on the lights of d, PA7 turned on c, PB6 turned on dp, PC7 turned on g, PA9 turned on f, PA8 turned on a, and PB10 turned on b that is part of the 7-segment display.

![image](https://user-images.githubusercontent.com/113574284/194786652-4caaf2c3-2d3d-49d3-8b8e-573434512215.png)

<center>Figure2. 7-segment display internal circuit</center>



#### Discussion

1. Draw the truth table for the BCD 7-segment decoder with the 4-bit input.

   ![image](https://user-images.githubusercontent.com/113574284/194787105-4df309fa-a95e-4bc5-8515-ebd9da82ca70.png)

2. What are the common cathode and common anode of 7-segment display?

   Common Anode 7 Segment LED Display
   In common anode type, the common voltage of +5V is applied to all the diodes.
   Depending upon the logic, the corresponding 0V is given which power-ups the diode and light is emitted from it.
   It is important to use a resistor in series with supply voltage; otherwise, the circuit can be damaged due to over-current.
   Common Cathode 7 Segment LED Display
   In the common cathode type, the common voltage of 0V is applied to all the diodes.
   Depending upon the logic, the corresponding +5V is given which power-ups the diode, and light is emitted from it. It is important to use a resistor in series with supply voltage; otherwise, the circuit can be damaged due to over-current.

3. Does the LED of a 7-segment display (common anode) pin turn ON when 'HIGH' is given to the LED pin from the MCU?

   In my case, it was lit when it was 'LOW'.

##          



### Problem2: Display 0~9 with button press



#### Procedure

In this part, you will press the button to create a code that lights up from 0 to 9 on the 7-segment display and returns to 0 to count. In order to properly control the system, it is first necessary to control the GPIO. Therefore, setting of the resister is necessary. The button will use the one built into nucleo-F401RE and set PC13 to digital input and pull-up, and the pins connected to the 7-segment display set PA5,6,7,8,9 / PB6,10 / PC7 to digital output, push-pull, no pull-up-down, speed.

#### Configuration

![image](https://user-images.githubusercontent.com/113574284/194787707-08afc5e9-abf1-422d-ade2-0e5ab22f4473.png)

#### Exercise

![image](https://user-images.githubusercontent.com/113574284/194789030-b7d534ea-e89b-434b-8d73-36eaa550fb9a.png)



#### Description with Code

* Description1

```c++
#include "stm32f4xx.h"
#include "ecGPIO.h"
#include "ecRCC.h"
#include "ecSysTick.h"

#define LED_PIN 	5
#define BUTTON_PIN 13

void setup(void);
	
int main(void) { 
	// Initialiization --------------------------------------------------------
	setup();
	unsigned int cnt = 0;
	
	// Inifinite Loop ----------------------------------------------------------
	while(1){
		sevensegment_decode(cnt % 10); 		  // = (0~9)
		if(GPIO_read(GPIOC, BUTTON_PIN) == 0){
			cnt++;
			for(int i = 0; i < 500000;i++){}  // delay
		}
		if (cnt > 9) cnt = 0;
	}
}


// Initialiization 
void setup(void)
{
	RCC_HSI_init();						  // setting for use GPIO
	GPIO_init(GPIOC, BUTTON_PIN, INPUT);  // calls RCC_GPIOC_enable()
	sevensegment_init();				  // it described below
}
```

```c++
void sevensegment_init(){		  		  // setting for use 7-segment display
	GPIO_init(GPIOA, 5, OUTPUT);
	GPIO_init(GPIOA, 6, OUTPUT);
	GPIO_init(GPIOA, 7, OUTPUT);
	GPIO_init(GPIOB, 6, OUTPUT);
	GPIO_init(GPIOC, 7, OUTPUT);
	GPIO_init(GPIOA, 9, OUTPUT);
	GPIO_init(GPIOA, 8, OUTPUT);
	GPIO_init(GPIOB, 10, OUTPUT);
	
	GPIO_otype(GPIOA, 5, Push_Pull);
	GPIO_otype(GPIOA, 6, Push_Pull);
	GPIO_otype(GPIOA, 7, Push_Pull);
	GPIO_otype(GPIOB, 6, Push_Pull);
	GPIO_otype(GPIOC, 7, Push_Pull);
	GPIO_otype(GPIOA, 9, Push_Pull);
	GPIO_otype(GPIOA, 8, Push_Pull);
	GPIO_otype(GPIOB, 10, Push_Pull);
	
	GPIO_pupd(GPIOA, 5, EC_NUD);
	GPIO_pupd(GPIOA, 6, EC_NUD);
	GPIO_pupd(GPIOA, 7, EC_NUD);
	GPIO_pupd(GPIOB, 6, EC_NUD);
	GPIO_pupd(GPIOC, 7, EC_NUD);
	GPIO_pupd(GPIOA, 9, EC_NUD);
	GPIO_pupd(GPIOA, 8, EC_NUD);
	GPIO_pupd(GPIOB, 10, EC_NUD);
	
	GPIO_ospeed(GPIOA, 5, Medium);
	GPIO_ospeed(GPIOA, 6, Medium);
	GPIO_ospeed(GPIOA, 7, Medium);
	GPIO_ospeed(GPIOB, 6, Medium);	
	GPIO_ospeed(GPIOC, 7, Medium);
	GPIO_ospeed(GPIOA, 9, Medium);
	GPIO_ospeed(GPIOA, 8, Medium);
	GPIO_ospeed(GPIOB, 10, Medium);
}

void sevensegment_decode(int num){			// decoder function
	int number[11][8]={						// state table
		{0,0,0,1,1,0,0,0},          //zero
		{0,1,1,1,1,0,1,1},          //one
		{0,0,1,1,0,1,0,0},          //two
		{1,0,0,1,0,1,0,0},          //three
		{1,1,0,1,0,0,1,0},          //four
		{1,0,0,1,0,0,0,1},          //five
		{0,0,0,1,0,0,1,1},          //six
		{1,1,0,1,1,1,0,0},          //seven
		{0,0,0,1,0,0,0,0},          //eight
		{1,1,0,1,0,0,0,0},          //nine
		{1,1,1,0,1,1,1,1}           //dot
	};
	GPIO_write(GPIOA, 5, number[num][0]); //e
	GPIO_write(GPIOA, 6, number[num][1]); //d
	GPIO_write(GPIOA, 7, number[num][2]); //c
	GPIO_write(GPIOB, 6, number[num][3]); //DP
	GPIO_write(GPIOC, 7, number[num][4]); //g
	GPIO_write(GPIOA, 9, number[num][5]); //f
	GPIO_write(GPIOA, 8, number[num][6]); //a
	GPIO_write(GPIOB, 10, number[num][7]); //b
}
```



### Results

In this experiment, we were able to practice implementing a decoder through code, and we were able to confirm that it worked as intended.

![image](https://user-images.githubusercontent.com/113574284/194790079-10d3958d-3a15-4a3a-9f2d-8a05989c0023.jpg)

#### Demo Video

[Youtube Link](https://youtube.com/shorts/-fcIKQPVV98?feature=share)

##          



### Reference

[LAB: GPIO Digital InOut 7-segment](https://ykkim.gitbook.io/ec/course/lab/lab-gpio-digital-inout-7segment)

[Difference Between Common Cathode and Common Anode 7 Segment LED Display](https://instrumentationtools.com/difference-between-common-cathode-and-common-anode-7-segment-led-display/#h-difference-between-them)

Class materials in Embedded Controller by Prof. Kim

RM0383 Reference manual

##          



### Troubleshooting

When I first made the matrix, I set 1 as the light and 0 as the light turned off. Soon I realized that it was the opposite, and I put ~ on the GPIO input to make it a station, but only f didn't light up. When the matrix, not ~, was written in reverse, it was confirmed that the light was turned on normally, and the inverse of 1 was not 0.
