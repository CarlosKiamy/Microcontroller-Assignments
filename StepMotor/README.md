# Step Motor Assignment
A stepper motor will be controlled using a potentiometer as an analog sensing device.

For stepper motor manipulation with the potentiometer, the on-board ADC is used with 12-bit resolution using 3.3 volts as the cap. The stepper motor driver inputs are connected to the general purpose outputs of the board for signal handling.

The datasheet for the stepper motor specifies that the motor has 32 steps for each revolution (11.25° each step) on the main rotor. It is worth mentioning that the motor has a gear train with a ratio of 1:64, which means that the output shaft must perform a total of 64*32 = 2048 steps per revolution (0.17578125°).


Below is the pinout declared for this assignment.

![StepMotor](https://github.com/CarlosKiamy/Microcontroller-Assignments/blob/main/img/StepMotor_pinout.png)


Wave drive signal sequence for the stepper motor driver.

![waveDrive](https://github.com/CarlosKiamy/Microcontroller-Assignments/blob/main/img/waveDrive.png)

Variables used for data transmission via UART.
```
char msg[12]; 					//raw data
char msgP[12]; 					//step motor "steps"
char msgAn[20]; 				//motor angle relative to starting point
```

Integer variables store variables for sequence, raw data, and steps within the sequence. within the sequence.
```
int pasos=0; 					//step motor "steps"
int raw; 					//Raw Data
int step=0; 					//Sequences
```

The float variables store the angle at which the motor is located and the degrees of each step.
```
float angulo; 					//Current motor angle/position
float anguloPaso = 0.17578125; 		        //Number of degrees for each step
```

The following code initializes the ADC and stores the raw data inside the "raw" variable, and then prints it out via UART.
```
HAL_ADC_Start(&hadc1);
HAL_ADC_PollForConversion(&hadc1, HAL_MAX_DELAY);
raw = HAL_ADC_GetValue(&hadc1);

sprintf(msg, "%hu\r\n", raw);
HAL_UART_Transmit(&huart2, (uint8_t*)msg, strlen(msg), HAL_MAX_DELAY);
```
Since the ADC has a resolution of 12 bits, the maximum value (in raw decimal) is 4096. This means that, for this activity, the ratio between the digital value of the potentiometer and the motor steps is 2:1.

The following condition makes sure that the raw data and the motor steps stay within the 2:1 ratio, and is only fulfilled if the potentiometer is not within its dead zone (raw != 0). This condition checks if the potentiometer is changing its position in an upward way, so it checks that it is greater than steps.

```
if(raw/2 >= pasos && raw != 0)
```

The for is used to perform the sequence of the motor signals, but in a descending way. This is because it is desired that the stepper motor follows the direction taken by the potentiometer. In this case the potentiometer and the motor will go clockwise.

```
for (step=3; step>=0; step--)
```

The switch case is used for signal switching for the motor inputs. In order for the motor to be clockwise, the signals must be switched downwards from case 3 to case 0.
```
			 switch (step){
				 case 0:
					 HAL_GPIO_WritePin(GPIOB, IN1_Pin, GPIO_PIN_SET);    // IN1
					 HAL_GPIO_WritePin(GPIOB, IN2_Pin, GPIO_PIN_RESET);  // IN2
					 HAL_GPIO_WritePin(GPIOB, IN3_Pin, GPIO_PIN_RESET);  // IN3
					 HAL_GPIO_WritePin(GPIOB, IN4_Pin, GPIO_PIN_RESET);  // IN4
				 break;
				 case 1:
					 HAL_GPIO_WritePin(GPIOB, IN1_Pin, GPIO_PIN_RESET);  // IN1
					 HAL_GPIO_WritePin(GPIOB, IN2_Pin, GPIO_PIN_SET);    // IN2
					 HAL_GPIO_WritePin(GPIOB, IN3_Pin, GPIO_PIN_RESET);  // IN3
					 HAL_GPIO_WritePin(GPIOB, IN4_Pin, GPIO_PIN_RESET);  // IN4
				 break;
				 case 2:
					 HAL_GPIO_WritePin(GPIOB, IN1_Pin, GPIO_PIN_RESET);  // IN1
					 HAL_GPIO_WritePin(GPIOB, IN2_Pin, GPIO_PIN_RESET);  // IN2
					 HAL_GPIO_WritePin(GPIOB, IN3_Pin, GPIO_PIN_SET);    // IN3
					 HAL_GPIO_WritePin(GPIOB, IN4_Pin, GPIO_PIN_RESET);  // IN4
				 break;
				 case 3:
					 HAL_GPIO_WritePin(GPIOB, IN1_Pin, GPIO_PIN_RESET);  // IN1
					 HAL_GPIO_WritePin(GPIOB, IN2_Pin, GPIO_PIN_RESET);  // IN2
					 HAL_GPIO_WritePin(GPIOB, IN3_Pin, GPIO_PIN_RESET);  // IN3
					 HAL_GPIO_WritePin(GPIOB, IN4_Pin, GPIO_PIN_SET);    // IN4
				 break;

```
Sequence to be followed by the switch case (clockwise):

![StepMotor_clockwise](https://github.com/CarlosKiamy/Microcontroller-Assignments/blob/main/img/StepMotor_clockwise.png)

Each case is one step that the motor turns, which means that each cycle that "for" completes is one step that the motor takes. For this reason the variable "pasos" is incremented by 1. To obtain the current angle of the motor, multiply the number of current steps by the number of angle per step. When these variables are obtained, they are put into the char arrays and displayed on the serial port.

```
			 HAL_Delay(1);
			 pasos++;
			 angulo = pasos*anguloPaso;

			 sprintf(msgP, "%hu\r\n", pasos);
			 HAL_UART_Transmit(&huart2, (uint8_t*)msgP, strlen(msgP), HAL_MAX_DELAY);

			 sprintf(msgAn, "%f\r\n", angulo);
			 HAL_UART_Transmit(&huart2, (uint8_t*)msgAn, strlen(msgAn), HAL_MAX_DELAY);
```

The following condition will check if the potentiometer value is falling.
```
if(raw/2 <= pasos)
```

The for cycle will now go in an upward direction so that the stepper motor makes its counterclockwise rotation.
```
for (step=0; step<=3; step++)
```
Sequence to be followed by the switch case (counterclockwise):

![StepMotor_clockwise](https://github.com/CarlosKiamy/Microcontroller-Assignments/blob/main/img/StepMotor_counterclockwise.png)

If the condition that checks that the potentiometer value is decreasing is met, the motor steps will decrease. For this reason the steps variable is decremented by one unit within this for loop.

```
pasos--;
```
