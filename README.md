# TimetoTask
This repositories is just for collect my old microcontroller project. This project can't be clone and use or developed further since it will need exact electronic component and mcu that I developed.
The MCU I used in this project is LPC1769.
The LPC1769 is a microcontroller chip from NXP Semiconductors, belonging to the ARM Cortex-M3 family. It is widely used in embedded systems and offers features such as high performance, low power consumption, and a variety of peripherals. The LPC1769 is often employed in applications like industrial control, robotics, and other embedded systems that require a compact and efficient microcontroller.


**about project**
In this microcontroller project, we will develop an embedded system that serves as a digital daily to-do list manager. This system aims to enhance productivity and task management by providing users with an efficient way to plan and track their daily tasks.

I wanted to do this project because this project will help me on time management. Normally,I only make a list of each day in my head. Sometimes I forgot some tasks or sometimes I forgot about times.
This project will help remind me about what I should do in particular time. It will enhance productivity of users. I wanted it to display time as a clock too.

**What is the functionality of this project?**
- Storing the entire day's timetable with timings.
- Sounding a buzzer indicating an incoming task.
- LED for indicating an incoming task.
- Be able to turn on and off with a switch.
- Button to go to the next/previous task.
- Display time.
- Display the current task.
- Ability to reprogram the board.

**Topic to apply in this project**
-GPIO
-Interrupt technique
-Timer (Real Time Clock)
-Pulse Width Modulation (PWM)
-LCD screen

**Hardware Component**
-LPC1769
-BreadBoard x 1
-LCD screen (ILI9341 TFT) x 1
-LED x 1
-Push Button x 4
-Buzzer x 1
-Cable wires


**Source code**
#include <stdio.h>
#include "mbed.h"
#include "SPI_TFT_ILI9341.h"
#include <string>
#include "Arial12x12.h"
#include "Arial24x23.h"
#include "Arial28x28.h"
#include "font_big.h"
#include <time.h>
#include "rtc_api.h"


//declaration
DigitalOut LCD_LED(P0_2); // the Watterott display has a backlight switch
InterruptIn btn(P0_3);
InterruptIn btn2(P0_21);
InterruptIn btnNext(P0_22);
InterruptIn btnBack(P0_25);
PwmOut PWM1(P2_0);
PwmOut PWM2(P2_1);
SPI_TFT_ILI9341 TFT(P0_9, P0_8, P0_7, P0_6, P0_0, P0_1,"TFT"); // mosi, miso, sclk, cs, reset, dc
int task_index = 0;
int beep_duration_ms = 100;
int beep_interval_ms = 200;
const char* task[] = {"Shower ","DressUp ","Breakfast ","Study&HW ","FreeTime ","Lunch ","HouseWork ","Free "};
const char* time_task[] = {"9:00-9:30 ","9:30-9:40 ","9:40-10:10 ","10:10-11:40 ","11:40-13:40 ","13:40-14:10 ","14:10-14:40 ",""};
//functions and interrupts
void powerOff(){
    LCD_LED = 0;
}


void powerOn(){
    LCD_LED = 1;
}


void next(){
    wait(0.5);
    if(task_index == 7){
        task_index = task_index + 0;
    } else{
        task_index = task_index + 1;
        task_index = task_index % 8;
    }
}


void back(){
    wait(0.5);
    if(task_index == 0){
        task_index = task_index + 0;
    } else{
        task_index = task_index -1;
        task_index = task_index % 8;
    }


}


void buzzer(){
    // Generate the first "beep"
    PWM1.period(1.0 / 500);  // Set the frequency to 500 Hz
    PWM1.write(0.5); // Set the duty cycle to 50%
    PWM2.period(1.0 / 500);
    PWM2.write(0.5);
    wait_ms(beep_duration_ms);


    // Turn off the buzzer
    PWM1.write(0.0);
    PWM2.write(0.0);
    wait_ms(beep_interval_ms);


    PWM1.period(1.0 / 500);
    PWM1.write(0.5);
    PWM2.period(1.0 / 500);
    PWM2.write(0.5);
    wait_ms(beep_duration_ms);




    PWM1.write(0.0);
    PWM2.write(0.0);
    wait_ms(beep_interval_ms);


    PWM1.period(1.0 / 500);
    PWM1.write(0.5);
    PWM2.period(1.0 / 500);
    PWM2.write(0.5);
    wait_ms(beep_duration_ms);




    PWM1.write(0.0);
    PWM2.write(0.0);
    wait_ms(beep_interval_ms);


}


void displayMain(){
    //first row display
    TFT.set_orientation(1);
    TFT.set_font((unsigned char*) Arial28x28);
    TFT.locate(10,10);
    TFT.printf("Time to (Task)");


    //horizontal for first row
    int x;
    int x0 = 0;
    int x1 = 250;
    TFT.set_orientation(1);
    for (x = x0; x<= x1; x++){
        TFT.pixel(x,45,Green);
    }
    //vertical for first row
    int startx = 250;
    int y;
    int y0 = 0;
    int y1 = 45;
    TFT.set_orientation(1);
    for (y = y0; y <= y1; y++){
        TFT.pixel(startx,y,Green);
    }
    //horizontal for last row
    int xl;
    int xl0 = 165;
    int xl1 = 320;
    TFT.set_orientation(1);
    for (xl = xl0; xl <= xl1 ; xl++){
        TFT.pixel(xl,200,Red);
    }
    //vertical for last row
    int startxl = 165;
    int yl;
    int yl0 = 200;
    int yl1 = 240;
    TFT.set_orientation(1);
    for (yl = yl0; yl <= yl1; yl++){
        TFT.pixel(startxl,yl,Red);
    }


}


void displayWhile(){
    //second row display
    TFT.set_orientation(1);
    TFT.set_font((unsigned char*) Arial24x23);
    TFT.locate(10,60);
    TFT.printf("Task: %s",task[task_index %8]);


    //third row
    TFT.set_orientation(1);
    TFT.set_font((unsigned char*) Arial24x23);
    TFT.locate(10,105);
    TFT.printf("Time: %s",time_task[task_index % 8]);


    //fourth row
    TFT.set_orientation(1);
    TFT.set_font((unsigned char*) Arial24x23);
    TFT.locate(10,150);
    TFT.printf("next: %s",task[task_index+1 %8]);
}


void displayTime(){
    time_t rtc_time = rtc_read();
    struct tm *rtc_time_info = localtime(&rtc_time);
    int hour = rtc_time_info->tm_hour;
    int min = rtc_time_info->tm_min;
    int sec = rtc_time_info->tm_sec;
    //clock at the last row
    TFT.set_orientation(1);
    TFT.set_font((unsigned char*) Arial24x23);
    TFT.locate(170,210);
    TFT.printf("%02d:%02d:%02d", hour, min, sec);


    if(hour == 9 && min == 0 && sec== 0){
        task_index = 0;
        buzzer();
    }
    else if (hour == 9 && min == 30 && sec < 4){
        task_index = 1;
        buzzer();
    }
    else if (hour == 9 && min == 40 && sec < 4){
        task_index = 2;
        buzzer();
    }
    else if (hour == 10 && min == 10 && sec < 4){
        task_index = 3;
        buzzer();
    }
    else if (hour == 11 && min == 40 && sec < 4){
        task_index = 4;
        buzzer();
    }
    else if (hour == 13 && min == 40 && sec < 4){
        task_index = 5;
        buzzer();
    }
    else if (hour == 14 && min == 10 && sec < 4){
        task_index = 6;
        buzzer();
    }
    else if (hour == 14 && min == 40 && sec < 4){
        task_index = 7;
        buzzer();
    }


}




int main()
 {
    rtc_init();


    time_t t = time(NULL);
    struct tm desired_time;
    desired_time.tm_sec = 50;    // Set seconds
    desired_time.tm_min = 39;    // Set minutes
    desired_time.tm_hour = 9;   // Set hours (e.g., 9:00:00)
    desired_time.tm_mday = 1;   // Set day of the month
    desired_time.tm_mon = 0;    // Set month (0 for January)
    desired_time.tm_year = 121; // Set years (e.g., 2021)


    time_t desired_time_unix = mktime(&desired_time);
    rtc_write(desired_time_unix);






    LCD_LED = 1;  // backlight on


    TFT.claim(stdout);      // send stdout to the TFT display
//  TFT.claim(stderr);      // send stderr to the TFT display
    TFT.set_orientation(1);
    TFT.background(Black);    // set background to black
    TFT.foreground(White);    // set chars to white
    TFT.cls();                // clear the screen


    TFT.set_orientation(0);
    TFT.background(Black);
    TFT.cls();


    //interupt binds
    btn.rise(&powerOn);
    btn2.rise(&powerOff);
    btnNext.rise(&next);
    btnBack.rise(&back);


    displayMain();


    while(1){


        displayWhile();
        displayTime();


    }




}


