# I2C Assignment
The I2C serial protocol was implemented to have a Master-Slave communication between the temperature sensor TMP102 and data interpolation between the sensor outputs and motor inputs, thus creating a tempereture-driven motor controller.

Below is the pinout used.

![i2c_pinout](https://github.com/CarlosKiamy/Microcontroller-Assignments/blob/main/img/i2c_pinout.png)

These variables below will be in charge of transporting the information between Master and Slave, as well as the variables that will be useful to manipulate the information collected by the TMP 102 sensor.
```
static const uint8_t TMP102_DIR =0x48<<1; // Address Slave
static const uint8_t REGISTRO_TEMP=0x00;  // Address Measurements

int16_t raw;			    // raw data from temperature sensor TMP102
uint8_t buffert[12]; 	// data buffer

float room_temp=27; 		// ambient temperature
float temp_c; 				  // temperature readings
float tmp_past=0 ; 			// previous temperature reading
float rest; 				    // temperature difference
int i = 0; 				    	// Counter Switch Case
int div; 				       	// if conditions
char msg[20]; 				  // used for serial output measurement readings
```

Within the infinite cycle "while(1)" is where the transmission and reading of data between Master and Slave is initialized with the help of the variables declared at the beginning. The data strings and the Master Transmit and Master Receive instructions have the necessary arguments to be able to indicate to the Master to whom it will send information, what information, type of information and size.
```
	  // Master-Slave instruction read
	  buffert[0]=REGISTRO_TEMP;
	  HAL_I2C_Master_Transmit(&hi2c1, TMP102_DIR, buffert, 1, HAL_MAX_DELAY);
	  //Reading of 2 bytes of the Temperature Register
	  HAL_I2C_Master_Receive(&hi2c1, TMP102_DIR, buffert, 2, HAL_MAX_DELAY);

	  //byte combination or merging
	  raw = ((int16_t)buffert[0] << 4) | (buffert[1] >> 4);

	  //Conversion with factor of 2 taking into account negative values
	  if (raw > 0x7FF){
		  raw |= 0xF000;
	  }
	  //Celsius conversion
	  temp_c = raw * 0.0625;

	  //Converting TEMP to decimal format
	  rest = temp_c - room_temp;
	  tmp_past=temp_c;
	  temp_c *= 100;

	  sprintf((char*)buffert, "%u.%02u C\r\n", ((unsigned int)temp_c /100), ((unsigned int)temp_c % 100));

	  // UART Temperature data transmission
	  HAL_UART_Transmit(&huart2,buffert,strlen((char*)buffert),HAL_MAX_DELAY);
 ```
 
Depending on the room temperature difference and the new measurement received, the following section is responsible for redirecting the flow of the same to each case. Each case contains the corresponding motor speeds depending on the mentioned temperature range difference. In this case 27 degrees Celsius is the room temperature. The dutyCycle will increase if the temperature difference is 2 degrees positive and will decrease if the difference is 2 degrees negative.

 ```
// Increment condition
	  div=rest/2;
	  if(rest >= 2){
		  i=div;
	  }
	  //Decrement condition
	  if(rest<=-2){
		  --i;
	  }

	  dutyCycle = __HAL_TIM_GET_AUTORELOAD(&htim2);

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
	sprintf(msg, "%hu\r\n", i);
	HAL_UART_Transmit(&huart2, (uint8_t*)msg, strlen(msg), HAL_MAX_DELAY);

	HAL_Delay(500);
 ```
