 Симулятор ПИД / Simulator PID
### (для начала уточним, что в код лучше не вглядываться)
* Симулятор ПИД без бубна с ручной подборкой коэффициентов для инкубатора на С++ to С# WF
* Рекомендуется погуглить с чем едят ПИД (я конечно этого не делал)
* Играя с коэффициентами экономит время и ресурс ардуинки не прошивая +100500 раз (eeprom затащит,я уверен, если постараться реализовать save && load)
* Подбор времени с отображением графика
* Адаптация под ШИМ для Ардуино/ESP и др... (0-255 но можно изменить) 
* Выбор dt и скорость loop()
* Кнопками UP DW имитируем нагрев и энерцию
* Мне удалось подобрать Kpid что точность с датчиком AHT10 составила +/- 0.02 по цельсию на лампочках 220v (последовательно подключенные конечно)
###  Пример "работы"
 
![SCREEN](https://github.com/ValeraFurman/PID/blob/main/PID%20TEST.gif?raw=true)
* Целевая рабочая среда MVS2019 NET Framework 4.7.2
* Используется для отрисовки графика - https://oxyplot.github.io/ - https://www.nuget.org/packages/OxyPlot.WindowsForms/
* Алгоритм ПИД (снимаю шапку перед автором) - https://alexgyver.ru/gyverpid/

```c#
using OxyPlot;
using OxyPlot.Series;
using OxyPlot.Axes;
//////////////........code
        float err; float integral = 0, prevErr = 0;
        float D;
//////////////........code
        public  float computePID(float input, float setpoint, float kp, float ki, float kd, float dt, int minOut, int maxOut)
        {
            err = setpoint - input;
            integral = Constrain(integral + (float)err * dt * ki, minOut, maxOut);
            D = (err - prevErr) / dt;
            prevErr = err;
            return Constrain(err * kp + integral + D * kd, minOut, maxOut);
        }
        // \hardware\arduino\avr\cores\arduino\Arduino.h
        // #define constrain(amt,low,high) ((amt)<(low)?(low):((amt)>(high)?(high):(amt)))
        public float Constrain(float amt, float low, float high)
        {
           return ((amt) < (low) ? (low) : ((amt) > (high) ? (high) : (amt)));
        }
//////////////........code
        private void timer2_Tick(object sender, EventArgs e)
        {
            // Выводим шим в лэйбу для наглядности 8bit                                        PMW->       MIN   MAX
            PMW_OUT.Text = Convert.ToString(computePID(Sensor, float.Parse(tSetup.Text),                      
float.Parse(KpSetup.Text), float.Parse(KiSetup.Text), float.Parse(KdSetup.Text), Int32.Parse(Dt_TIME.Text), 0, 255));
            // code?
        }
//////////////........code
```
