<p>
To visualize how your settings make this happen, here is how those pieces work together in a chain reaction:
1. The Base Speed: The internal clock gets slowed down by your Prescaler (PSC) to tick at exactly 1 MHz (1 million times a second).
2. The Total Loop (ARR): The counter counts those 1 MHz ticks up to your Counter Period (ARR) of 100, then resets. This defines the frequency (how fast the loop repeats).
3. The Duty Cycle (CH1 / CCR1): Channel 1 watches that count. It decides how much of that 100-tick loop the pin stays HIGH versus LOW based on your Pulse (CCR1) value.
So if you change Pulse (CCR1) to 50 in your code later, Channel 1 keeps the pin HIGH for 50 ticks and LOW for 50 ticks, giving you a perfect 50% duty cycle.
</p>


Step 1: Start the PWM (Put in main.c before your loop)

Add this line right after your peripherals initialize to turn the signal on:

```
c

HAL_TIM_PWM_Start(&htim1, TIM_CHANNEL_1);
```



Step 2: Change the Duty Cycle (Inside your while(1) loop)

Use this specific macro to change the Pulse (CCR1) value dynamically. Since your ARR is 100, the value you type here directly equals your duty cycle percentage (0 to 100):
```
c
// Set duty cycle to 25% (ON for 25 ticks, OFF for 75 ticks)
__HAL_TIM_SET_COMPARE(&htim1, TIM_CHANNEL_1, 25);
```



Complete Code:
```
/* Initialize all configured peripherals */
MX_GPIO_Init();
MX_TIM1_Init();

/* 1. Start the PWM signal on Channel 1 */
HAL_TIM_PWM_Start(&htim1, TIM_CHANNEL_1);

while (1)
{
    /* 2. Change duty cycle to 10% (Dim) */
    __HAL_TIM_SET_COMPARE(&htim1, TIM_CHANNEL_1, 10);
    HAL_Delay(1000); // Wait 1 second
    
    /* 3. Change duty cycle to 90% (Bright) */
    __HAL_TIM_SET_COMPARE(&htim1, TIM_CHANNEL_1, 90);
    HAL_Delay(1000); // Wait 1 second
}
```



