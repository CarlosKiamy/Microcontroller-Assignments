# PWM Assignment
The assignment involves the following task:

Perform the control of a DC motor to vary the speed by means of two buttons, where the number of presses increases or decreases the PWM by 10 %, starting from speed 0. 

Button A will determine the increase or decrease of the motor speed.

Button B will increase the speed by 10% each time it is activated.


Below is the pinout declared for this assignment.

![PWM_pinout](https://github.com/CarlosKiamy/Microcontroller-Assignments/blob/main/img/PWM_pinout.png)


Prescaler and Period value declaration.

![Prescaler](https://github.com/CarlosKiamy/Microcontroller-Assignments/blob/main/img/PrescalerPeriod.png)

Variables used for data transmission via UART.
```
char msg[20];
char msgT[20];
```

Initialization for PWM. The dutyCycle integer will take the value that the TIM pin currently captures. The variables "toggle" and "i" are used for button manipulation.
```
HAL_TIM_PWM_Start(&htim2, TIM_CHANNEL_1);
uint16_t dutyCycle = HAL_TIM_ReadCapturedValue(&htim2, 
TIM_CHANNEL_1);
int toggle = 0;
int i = 0;
```

Since button A works with 2 states (the value is incremented or decremented), it was configured as a boolean and the integer "toggle" will take its values.
```
//toggle A button
	  if(HAL_GPIO_ReadPin(GPIOA, botonA_Pin)){
		  toggle = 1;
	  }
	  if(!HAL_GPIO_ReadPin(GPIOA, botonA_Pin)){
		  toggle = 0;
	  }
```
where:

o toggle = 1: Speed Increase

o toggle = 0: Speed Decrease

The following section is the configuration for the B button. Depending on the toggle state, pressing button B will increase or decrease its value by 1 until it reaches 0 or 10. The task calls for 10% increments or decrements of speed each time button B is pressed. This means that we have 10 speed states (1 <= i <= 10) and a total stop (i = 0).
```
//increment/decrement buttons
	  if(toggle==1 && HAL_GPIO_ReadPin(GPIOA, botonB_Pin)==1 && i<10){
		  ++i;
	  }
	  if(toggle==0 && HAL_GPIO_ReadPin(GPIOA, botonB_Pin)==1 && i>0){
		  --i;
	  }
 ```
 
 dutyCycle gets the period set for PWM, which is 999.
 ```
 dutyCycle = __HAL_TIM_GET_AUTORELOAD(&htim2);
 ```
 
A switch case was used to manipulate the motor speed. Each value of the variable "i" serves as a speed indicator (for example, if i = 5, this means that the speed will be 50%). For this reason in each case the value of dutyCycle has a decimal multiplier to decrease the value of the maximum speed to the desired percentage. 

As the variable "i" depends on the switching of button B, it will remain constant until button B is pressed, and only then the speed will change until it reaches its maximum of 100% (i=10) or minimum (i=0), as indicated by the "if" conditions previously established.
```
	  switch(i){
	  	  case 0:
	  		  __HAL_TIM_SET_COMPARE(&htim2, TIM_CHANNEL_1, dutyCycle*0);
	  	  break;

		  case 1:
			  __HAL_TIM_SET_COMPARE(&htim2, TIM_CHANNEL_1, dutyCycle*0.1);
		  break;

		  case 2:
			  __HAL_TIM_SET_COMPARE(&htim2, TIM_CHANNEL_1, dutyCycle*0.2);
		  break;

		  case 3:
			  __HAL_TIM_SET_COMPARE(&htim2, TIM_CHANNEL_1, dutyCycle*0.3);
		  break;

		  case 4:
			  __HAL_TIM_SET_COMPARE(&htim2, TIM_CHANNEL_1, dutyCycle*0.4);
		  break;

		  case 5:
			  __HAL_TIM_SET_COMPARE(&htim2, TIM_CHANNEL_1, dutyCycle*0.5);
		  break;

		  case 6:
			  __HAL_TIM_SET_COMPARE(&htim2, TIM_CHANNEL_1, dutyCycle*0.6);
		  break;

		  case 7:
			  __HAL_TIM_SET_COMPARE(&htim2, TIM_CHANNEL_1, dutyCycle*0.7);
		  break;

		  case 8:
			  __HAL_TIM_SET_COMPARE(&htim2, TIM_CHANNEL_1, dutyCycle*0.8);
		  break;

		  case 9:
			  __HAL_TIM_SET_COMPARE(&htim2, TIM_CHANNEL_1, dutyCycle*0.9);
		  break;

		  case 10:
			  __HAL_TIM_SET_COMPARE(&htim2, TIM_CHANNEL_1, dutyCycle);
		  break;
	  }
```

The UART transmitter was only used to corroborate the values of "toggle" and "i". It was digitally corroborated that pressing the A and B buttons had the desired change of values before connecting the motor. A small delay of 100 milliseconds for each duty cycle was added.
```
	  sprintf(msg, "%hu\r\n", i);
	  sprintf(msgT,"%hu\r\n", toggle);
	  HAL_UART_Transmit(&huart2, (uint8_t*)msg,  strlen(msg),
			  	  	    HAL_MAX_DELAY);
	  HAL_UART_Transmit(&huart2, (uint8_t*)msgT, strlen(msgT),
			  	  	    HAL_MAX_DELAY);
	  HAL_Delay(100);
```
