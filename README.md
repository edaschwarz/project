#include <main.h>
#include <stm8s.h>

#define IN_PORT GPIOD               /* inicializace tlačítka */
#define IN_PIN GPIO_PIN_1

#define BATTERY_PORT GPIOD         /*  inicializace přepínače */
#define BATTERY_PIN GPIO_PIN_3

void ring(){
    if(GPIO_ReadInputPin(BATTERY_PORT, BATTERY_PIN) == 0){
        TIM2_SetCompare1(4000);
        delay_ms(300);
        TIM2_SetCompare1(500);
        delay_ms(300);
        TIM2_SetCompare1(3000);             // délka zvuku při nižším napětím
        delay_ms(300);
        TIM2_SetCompare1(200);
        delay_ms(300);
        TIM2_SetCompare1(0);
    }else{
        TIM2_SetCompare1(2500);
        delay_ms(1000);
        TIM2_SetCompare1(0);
        delay_ms(100);
        TIM2_SetCompare1(500);
        delay_ms(1000);
        TIM2_SetCompare1(0);
        delay_ms(1000);
        TIM2_SetCompare1(2500);             // délka zvuku při nižším napětí
        delay_ms(1000);
        TIM2_SetCompare1(0);
        delay_ms(100);
        TIM2_SetCompare1(500);
        delay_ms(1000);
        TIM2_SetCompare1(0);
        delay_ms(1000);
    }
}
int main(void)
{
    CLK_HSIPrescalerConfig(CLK_PRESCALER_HSIDIV1); //Set CLK

    GPIO_Init(IN_PORT, IN_PIN, GPIO_MODE_IN_PU_NO_IT);
    GPIO_Init(BATTERY_PORT, BATTERY_PIN, GPIO_MODE_IN_PU_NO_IT);        

    init_milis();
    init_uart1();

    TIM2_TimeBaseInit(TIM2_PRESCALER_4, 4000);
    TIM2_OC1Init(
        TIM2_OCMODE_PWM1,
        TIM2_OUTPUTSTATE_ENABLE,            
        3000,
        TIM2_OCPOLARITY_HIGH);
    TIM2_OC1PreloadConfig(ENABLE);
    TIM2_Cmd(ENABLE);

    TIM2_SetCompare1(0);            // aby zvuk nebyl poničen

    uint8_t old_down_state = 0;
    printf("HELLO WORLD\n");
    while(1) {
        if(GPIO_ReadInputPin(IN_PORT, IN_PIN) == 0){
            if(old_down_state == 0) {                   // hlavní program
                ring();
            }
            old_down_state = 1;
        }else old_down_state = 0;
    }
}
/*-------------------------------  Assert -----------------------------------*/
#include "__assert__.h"
