// 1) Этот код ужасен
// 2) Джойстик неудобен и ненадежен, переделать на кнопки

#include <iarduino_RTC.h>
#include <TM1637.h>
#include <EEPROM.h>

iarduino_RTC time(RTC_DS1307);

#define CLK_CLOCK 8  
#define DIO_CLOCK 7   

#define CLK_ALARM 3  
#define DIO_ALARM 2   


TM1637 tm1637_CLOCK(CLK_CLOCK,DIO_CLOCK);
TM1637 tm1637_ALARM(CLK_ALARM,DIO_ALARM);

#define ON 1
#define OFF 0

int buzzer_pin = 9; // цифровой пин динамика
const unsigned int melody_time = 60000; // время проигрывания мелодии будильника 1 минута = 60000
int photocell_pin = 2; // аналоговый пин фоторезистора
int photocell_data;  // данные с фоторезистора

int current_hours = 0; // текущие часы
int current_minute = 0; // текущие минуты

int alarm_hours = 0; // часы будильника
int alarm_minute = 0; // минуты будильника

int alarm_hours_repeat_1 = 0; // часы будильника 
int alarm_minute_repeat_1 = 0; // минуты будильника

int alarm_hours_repeat_2 = 0; // часы будильника
int alarm_minute_repeat_2 = 0; // минуты будильника

int repeat_alarm_1 = 5;
int repeat_alarm_2 = 10; 

boolean alarm_blink_flag; // мигать индикатором будильника
boolean clock_blink_flag; // мигать индикатором часов
boolean alarm_off = false;
int hours_when_the_alarm_was_stopped;
int minute_when_the_alarm_was_stopped;

int time_sec = 0;
int time_min = 0;
int time_hour = 0;

int brightness_CLOCK = 2; // яркость индикатора часов. после первого включения = 2
int brightness_ALARM = 1; // якрость индикатора будильника. после первого включения  = 1, так как оранжевый индикатор ярче
long brigtness_loop_delay = 10000;
long brigtness_loop_old_mills;

int joy_x_pin = A1; // аналоговый пин джойстика оси X
int joy_y_pin = A0; // аналоговый пин джойстика оси Y
int joy_button_pin = 10; // цифровой пин кнопки джойстика
int joy_x_position = 0; 
int joy_y_position = 0;
int joy_button_state = 0;

int mode_change = 0;
int old_mode_change = 0;

int8_t data_lcd[] = {0x00,0x00,0x00,0x00};
int8_t alarm_lcd[] = {0x00,0x00,0x00,0x00};
unsigned char ClockPoint = 1;

long main_loop_delay = 60000;
long main_loop_old_mills;
long joy_loop_delay = 100;
long joy_loop_old_mills;

long alarm_setup_loop_delay = 300;
long alarm_setup_loop_old_mills;

long clock_setup_loop_delay = 300;
long clock_setup_loop_old_mills;

long for_melody_delay = 300;
long for_melody_delay_old_mills;
boolean melody_on = false;

boolean clock_clear_display = false;
boolean alarm_clear_display = false;

int i;
int ji;
int jk;

void setup() 
{
  
  Serial.begin(9600);
  
  pinMode(joy_x_pin, INPUT);
  pinMode(joy_y_pin, INPUT);
  pinMode(joy_button_pin, INPUT_PULLUP);
  
    tm1637_ALARM.point(POINT_ON);
      
    tm1637_CLOCK.set(brightness_CLOCK); 
    tm1637_CLOCK.init(); 

    tm1637_ALARM.set(brightness_ALARM); 
    tm1637_ALARM.init(); 
    
    time.begin();
    
    joy_loop_old_mills = main_loop_old_mills = millis(); 

  alarm_hours = EEPROM[0]; // сохраненное значение часов из энергонезависимой памяти
  alarm_minute = EEPROM[1]; // сохраненное значение минут из энергонезависимой памяти
  add_time (); // расчет времени повторного срабатывания будильника через время указанное в переменных repeat_alarm_1 и repeat_alarm_2

      request_time(); // запрос времени для вывода на индикатор сразу после включения часов
      set_brigtness(); // установка яркости индикаторов сразу после включения часов
      disp(); // отобразить данные на индикаторов сразу после включения часов

}


// функция установки яркости индикаторов в зависимости от освещенности фоторезистора
// используется две ступени яркости - дневная и вечерняя. 
// желтый индикатор светится ярче, поэтому ему задается степень яроксти на одну единицу меньше

void set_brigtness() {
 if ((millis()- brigtness_loop_old_mills) >= brigtness_loop_delay) {

  photocell_data = analogRead(photocell_pin);
  if (photocell_data > 700) {
              brightness_CLOCK = 1;
              brightness_ALARM = 0;
  } else {
              brightness_CLOCK = 4;
              brightness_ALARM = 5;
      }
  
  tm1637_CLOCK.set(brightness_CLOCK); 
  tm1637_ALARM.set(brightness_ALARM); 

 brigtness_loop_old_mills=millis();
  }
}
// функция опроса часов реального времени
void request_time() {
      
     time.gettime("Hi");
     
    
     current_hours = time.Hours;   //time.Hours определена в iarduino_RTC.h
     current_minute = time.minutes; //time.minutes определена в iarduino_RTC.h
 
          
}

// функция отображения изменений на индикаторах
void disp() 
 {

        // мигание двоеточия часов по полсекнды на исчезновение и появление
      tm1637_CLOCK.point (time.seconds % 2 ? POINT_ON : POINT_OFF); //time.seconds определена в iarduino_RTC.h
      

     
    if (mode_change == 1)  {
          alarm_blink_flag = !alarm_blink_flag;
          if (alarm_blink_flag == false) {
            tm1637_ALARM.set(2); 
          } else {
            tm1637_ALARM.set(3); 
          }
                   
    }

    if (mode_change == 2)  {
          clock_blink_flag = !clock_blink_flag;
          if (clock_blink_flag == false) {
            tm1637_CLOCK.set(2); 
          } else {
            tm1637_CLOCK.set(3); 
          }
                   
    }

    
if (clock_clear_display == false) {
    data_lcd[0] = current_hours / 10; 
    data_lcd[1] = current_hours % 10; 
    data_lcd[2] = current_minute / 10;
    data_lcd[3] = current_minute % 10;
    
    tm1637_CLOCK.display(data_lcd);
}

if (alarm_clear_display == false && alarm_off == false) {
      alarm_lcd[0] = alarm_hours / 10; 
      alarm_lcd[1] = alarm_hours % 10; 
      alarm_lcd[2] = alarm_minute / 10;
      alarm_lcd[3] = alarm_minute % 10;
    
    tm1637_ALARM.display(alarm_lcd);
}

 }

void alarm_setup() {
  alarm_off = false;
 if ((millis()- alarm_setup_loop_old_mills) >= alarm_setup_loop_delay) {
 
     if (joy_x_position > 1000)  {
     alarm_hours++;
     if (alarm_hours > 23) alarm_hours = 0;
  add_time();   // автоматический расчет времени повторного срабатывания будильника через время указанное в переменных repeat_alarm_1 и repeat_alarm_2
  EEPROM[0] = alarm_hours; // обновить данные в энергонезависиомй памяти
      
                    }

     if (joy_x_position < 20)  {
     alarm_hours--;
     if (alarm_hours < 0) alarm_hours = 23;
  add_time(); // расчет времени повторного срабатывания будильника через время указанное в переменных repeat_alarm_1 и repeat_alarm_2     
  EEPROM[0] = alarm_hours; // обновить данные в энергонезависиомй памяти

              }  

     if (joy_y_position > 1000)  {
     alarm_minute++;
     if (alarm_minute > 59) alarm_minute = 0;
  add_time(); // расчет времени повторного срабатывания будильника через время указанное в переменных repeat_alarm_1 и repeat_alarm_2     
  EEPROM[1] = alarm_minute; // обновить данные в энергонезависиомй памяти
     
               }

     if (joy_y_position < 20)  {
     alarm_minute--;
     if (alarm_minute < 0) alarm_minute = 59;
  add_time(); // расчет времени повторного срабатывания будильника через время указанное в переменных repeat_alarm_1 и repeat_alarm_2     
  EEPROM[1] = alarm_minute; // обновить данные в энергонезависиомй памяти

              }  
  disp(); // обновить данные на индикаторах

alarm_setup_loop_old_mills=millis();
 }       

}


void clock_setup() {
       
 if ((millis()- clock_setup_loop_old_mills) >= clock_setup_loop_delay) {
  

     if (joy_x_position > 1000)  {
     current_hours++;
     if (current_hours > 23) current_hours = 0;
     time.settime(time_sec, current_minute, current_hours);  
               }

     if (joy_x_position < 20)  {
     current_hours--;
     if (current_hours < 0) current_hours = 23;
     time.settime(time_sec, current_minute, current_hours);
              }  

     if (joy_y_position > 1000)  {
     current_minute++;
     if (current_minute > 59) current_minute = 0;
     time.settime(time_sec, current_minute, current_hours);
               }

     if (joy_y_position < 20)  {
     current_minute--;
     if (current_minute < 0) current_minute = 59;
     time.settime(time_sec, current_minute, current_hours);
              }  
  
  disp(); 
  clock_setup_loop_old_mills=millis();
 }
 }


void joy_quick_alarm_off () {

if (alarm_off == true) ji = 0; // чтобы ненакапливалось ji при случайных наклонах джойстика
if (analogRead(joy_x_pin) < 600) ji = 0; // если джойстик отпускали, ji должно быть обнулено
 
 if(analogRead(joy_x_pin) > 1000 ){
      while(analogRead(joy_x_pin) > 1000) {
        delay(10);      // ждём пока мы её не отпустим
        
        if(ji<300)
          {
            ji++;
            } else {
ji = 0;
alarm_off = true;
beep();
delay (1000);

if (alarm_off == true) {             // дублирующийся код, вынести в функцию
    tm1637_ALARM.point(POINT_OFF);
    tm1637_ALARM.clearDisplay();
    alarm_lcd[0] = 0x20; 
    alarm_lcd[1] = 0x00;
    alarm_lcd[2] = 0xF;
    alarm_lcd[3] = 0xF;
    tm1637_ALARM.display(alarm_lcd);
}                          // фиксируем, как долго удерживается кнопка SET, если дольше 2 секунд, то стираем экран
      }
      
      }
      
 }


  
}

void joy_quick_alarm_on () {

if (alarm_off == false) jk = 0; // чтобы не накапливалось ji при случайных наклонах джойстика
if (analogRead(joy_x_pin) > 400) jk = 0; // если джойстик отпускали, ji должно быть обнулено
 
 if(analogRead(joy_x_pin) < 20 ){
      while(analogRead(joy_x_pin) < 20) {
        delay(10);     
        
        if(jk<300)
          {
            jk++;
            } else {
jk = 0;
alarm_off = false;
beep();
delay (300);

if (alarm_off == false) {           
disp();                              // enable clock display after alarm setup
}                                    // фиксируем, как долго удерживается кнопка SET, если дольше 2 секунд, то стираем экран
      }
      
      }
      
 }


  
}


void joy () {

joy_x_position = analogRead(joy_x_pin);
joy_y_position = analogRead(joy_y_pin);
joy_button_state = digitalRead(joy_button_pin);


if (mode_change == 0) {
  
  joy_quick_alarm_off();
  joy_quick_alarm_on();
  
}

if (melody_on == false) {
  
// alarm setup on/of
if (joy_button_state == 0 && old_mode_change == 1) 
{
  alarm_off = false;
  mode_change = 2;
  disp();               // enable clock display after alarm setup
  delay (300);
  
}

if (joy_button_state == 0 && old_mode_change == 2) 
{
  mode_change = 0;
  disp();
  delay (100);
}

if (joy_button_state == 0 && old_mode_change == 0) {

  delay (100);
  mode_change = 1;  // mode_change = 1 - enable alarm setup

}
old_mode_change = mode_change;


if (mode_change == 2) 
{
    clock_clear_display = false;
    tm1637_CLOCK.point(POINT_ON);
    clock_setup ();
    alarm_clear_display = true;
    tm1637_ALARM.point(POINT_OFF);
    tm1637_ALARM.clearDisplay();
  }

if (mode_change == 1) 
{
    alarm_setup ();
    clock_clear_display = true;
    tm1637_CLOCK.point(POINT_OFF);
    tm1637_CLOCK.clearDisplay();
  }
  
if (mode_change == 0)  {
  clock_clear_display = false;
  alarm_clear_display = false;
  tm1637_ALARM.point(POINT_ON);
  tm1637_CLOCK.point(POINT_ON);
  request_time(); 
  set_brigtness();
  disp(); 
  
}

}
}




void alarm_melody () {

melody_on = true;
pinMode(buzzer_pin, OUTPUT);

for (i = 0; i < melody_time; i++) {
  
if ((millis()- for_melody_delay_old_mills) >= for_melody_delay) {
 noTone(buzzer_pin);
 tone(buzzer_pin, 433, 200);
  for_melody_delay_old_mills = millis();
}

  
if ((millis()- for_melody_delay_old_mills) >= for_melody_delay) {
  tone(buzzer_pin, 333, 200);
  for_melody_delay_old_mills = millis();
}

joy ();    
if (joy_x_position > 1000 || joy_x_position < 20 || joy_y_position > 1000 || joy_y_position < 20)  {

  pinMode(buzzer_pin, INPUT); // костыль
  melody_on = false;
  break;  
}

if (joy_button_state == 0 && mode_change == 0) {
  alarm_off = true;

if (alarm_off == true) {
    tm1637_ALARM.point(POINT_OFF);
    tm1637_ALARM.clearDisplay();
    alarm_lcd[0] = 0x20; 
    alarm_lcd[1] = 0x00;
    alarm_lcd[2] = 0xF;
    alarm_lcd[3] = 0xF;
    tm1637_ALARM.display(alarm_lcd);
}


  pinMode(buzzer_pin, INPUT); // костыль
  melody_on = false;
  beep();
  
  break;
}
             
delay (1); 
}
pinMode(buzzer_pin, INPUT); // костыль
melody_on = false;
}

void beep () {

  pinMode(buzzer_pin, OUTPUT);
  noTone(buzzer_pin);
  tone(buzzer_pin, 533, 200);
  delay (300);
  pinMode(buzzer_pin, INPUT); // костыль
}

void check_alarm() {
  
if (alarm_hours == current_hours && alarm_minute == current_minute)  alarm_melody ();
if (alarm_hours_repeat_1 == current_hours && alarm_minute_repeat_1 == current_minute)  alarm_melody ();
if (alarm_hours_repeat_2 == current_hours && alarm_minute_repeat_2 == current_minute)  alarm_melody ();    

  }

void add_time () {

  alarm_minute_repeat_1 = alarm_minute + repeat_alarm_1; 
  alarm_minute_repeat_2 = alarm_minute + repeat_alarm_2;
  alarm_hours_repeat_1 = alarm_hours;
  alarm_hours_repeat_2 = alarm_hours;
  
  if (alarm_minute_repeat_1 >= 60){
    alarm_hours_repeat_1++;
    alarm_minute_repeat_1 = alarm_minute_repeat_1 - 60;
  }

  if (alarm_minute_repeat_2 >= 60){
    alarm_hours_repeat_2++;
    alarm_minute_repeat_2 = alarm_minute_repeat_2 - 60;
  }

 if (alarm_hours_repeat_1 == 24) alarm_hours_repeat_1 = 0;
 if (alarm_hours_repeat_2 == 24) alarm_hours_repeat_2 = 0;



}


void loop() {

 if ((millis()- joy_loop_old_mills) >= joy_loop_delay) {
 joy();
 
  joy_loop_old_mills=millis();

 }
 
 if ((millis()- main_loop_old_mills) >= main_loop_delay) {
 
  request_time(); 
  set_brigtness();
  disp(); 
  if (alarm_off == false) check_alarm();
  
  
  
  main_loop_old_mills=millis();
 }
 
} 
