# Object Detector Using Ultrasonic Sensor and vsdquardon mini.

This project demonstrates how to interface an ultrasonic sensor with an LED using a CH32V microcontroller. The system measures the distance and turns on the LED when an object is closer than 15 cm.

## Code

```c
#include "debug.h"  // Include the appropriate header for CH32V

#define TRIG_PIN GPIO_Pin_4   // Ultrasonic Trigger Pin on PC4
#define ECHO_PIN GPIO_Pin_5   // Ultrasonic Echo Pin on PC5
#define BLUE_LED GPIO_Pin_3   // Blue LED on PC3

void Ultrasonic_Init() {
    GPIO_InitTypeDef GPIO_InitStructure = {0};

    // Enable clock for Port C
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOC, ENABLE);

    // Setup Trigger as Output Push-Pull
    GPIO_InitStructure.GPIO_Pin = TRIG_PIN;
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_Init(GPIOC, &GPIO_InitStructure);
    GPIO_ResetBits(GPIOC, TRIG_PIN);  // Ensure TRIG is low initially

    // Setup Echo as Input Floating
    GPIO_InitStructure.GPIO_Pin = ECHO_PIN;
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IN_FLOATING;
    GPIO_Init(GPIOC, &GPIO_InitStructure);

    // Setup Blue LED as Output Push-Pull
    GPIO_InitStructure.GPIO_Pin = BLUE_LED;
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_Init(GPIOC, &GPIO_InitStructure);
    GPIO_ResetBits(GPIOC, BLUE_LED);  // Ensure LED is off initially
}

uint16_t Measure_Distance() {
    uint32_t duration;
    uint16_t distance;

    // Send 10us Trigger Pulse
    GPIO_SetBits(GPIOC, TRIG_PIN);
    Delay_Us(10);
    GPIO_ResetBits(GPIOC, TRIG_PIN);

    // Wait for Echo to go HIGH
    while (GPIO_ReadInputDataBit(GPIOC, ECHO_PIN) == 0);

    // Measure High Time
    duration = 0;
    while (GPIO_ReadInputDataBit(GPIOC, ECHO_PIN) == 1) {
        duration++;
        Delay_Us(1);
    }

    // Calculate Distance (duration * speed of sound / 2)
    distance = (duration * 0.034) / 2;

    return distance;
}

int main(void) {
    SystemCoreClockUpdate();
    Delay_Init();
    USART_Printf_Init(115200);
    Ultrasonic_Init();

    while (1) {
        uint16_t distance = Measure_Distance();
        printf("Distance: %d cm\n", distance);

        if (distance < 15) {
            GPIO_SetBits(GPIOC, BLUE_LED);   // Turn on LED if distance < 15 cm
        } else {
            GPIO_ResetBits(GPIOC, BLUE_LED); // Turn off LED otherwise
        }

        Delay_Ms(500);  // Check every 500ms
    }
}
