# ADC assignment
The assignment involves the following tasks:

- Task 1:

o Perform the measurement of an analog variable with a potentiometer and print its value on 10 LEDs.

- Task 2:

o Use the potentiometer characterization experiment to visualize by means of LEDs the phenomena of linearity, dead zone, saturation and bias. Explain in detail your observations even if the sensor used does not have any of these characteristics.

- Task 3

o Represent in a table the values of position, voltage, binary word. As an example, your table should look as follows

![table](https://github.com/CarlosKiamy/Microcontroller-Assignments/blob/main/img/table.jpeg)

- Task 4

o Using the interpolation formulas seen in class, demonstrate analytically the values found in the table of task 3.

You can find the main code used in the source folder under main.c, or at MAINCODEHERE.c under the ADC folder. The code is explained below:

Below you can see the different variables that were used to assist the operation of the program and structure it for its correct use. The raw variable was used to read the data that we have at the moment in the ADC. The variable msg helped with the transmission of data from our serial display. The variables binario and filo were of great help for the management and arrangement of the bits for the sampling of the data bits for the sampling of results. For the array fill function and the bit allocation, it was necessary to make use of for cycles and array strings such as int led_pins[]. 
```
int raw;
char msg[10]; 
int binario[10], filo[10];
int i, j;
int led_pins[] = {LED1_Pin, LED2_Pin, LED3_Pin, LED4_Pin, 
LED5_Pin, LED6_Pin, LED7_Pin, LED8_Pin, LED9_Pin, LED10_Pin};
```


The following code does the reading for the ADC measurements.
o HAL_ADC_Start(&hadc1); initializes the ADC, calling the microcontroller pin that was previously assigned.
o HAL_ADC_PollForConversion(&hadc1, HAL_MAX_DELAY); "instructs" the ADC to read the current value via its pin &hadc1.
o raw = HAL_ADC_GetValue(&hadc1); stores the raw data of the ADC in the raw variable
```
while (1)
 {
HAL_ADC_Start(&hadc1);
HAL_ADC_PollForConversion(&hadc1, HAL_MAX_DELAY);
raw = HAL_ADC_GetValue(&hadc1);
```


In this section, sprintf stores the raw data as a char inside msg, and afterwards transmits the result via UART.
```
sprintf(msg, "%hu\r\n", raw);
HAL_UART_Transmit(&huart2,(uint8_t*)msg,strlen(msg,HAL_MAX_DELAY);
```


To start with the conversion, the parameters of the for cycle are established, which will be in charge of controlling the flow of the conversion organization until it is completed. In the first cycle the counters i and j have values 0 and 9 respectively. The binary variable of type array takes the first residue of the value obtained from the ADC and places it in the position 0.
```
for(i=0, j=9; i<10; i++, j--)
{
 binario[i]=raw%2;
 raw=raw/2;
 filo[j]=binario[i];
 sprintf(msg, "%hu\r\n", filo[i]);
 HAL_UART_Transmit(&huart2,(uint8_t*)msg,strlen(msg),HAL_MAX_DELAY);

 //turn LEDs On
 if (filo[i]){
  HAL_GPIO_TogglePin(GPIOB, led_pins[i]);
 }
```

To continue with the first cycle of the loop, the first residue (1 or 0) must be set so that it is in the least significant bit and not in the most significant bit. When using the method of residues, these will take inverse positions and will be shown erroneously in the serial display. So it was decided to take an array variable "filo" to rearrange the bits. This is where J=9 does its job and rearranges the first residue stored in the binary array to the least significant bit in the edge array.

Finally, all the LEDs will turn off from left to right to give way to a new binary calculation and display of the measurement obtained.
```
for(i=0;i<10;i++)
{
  HAL_GPIO_WritePin(GPIOB, led_pins[i], 0);
}
HAL_Delay(1);
```
![10leds](https://github.com/CarlosKiamy/Microcontroller-Assignments/blob/main/img/10leds.jpeg)
