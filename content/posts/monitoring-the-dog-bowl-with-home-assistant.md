---
title: "monitoring the dog bowl with home assistant"
date: "2018-07-07"
slug: "2018/07/07/monitoring-the-dog-bowl-with-home-assistant"
cover: "assets/2018/fountains-ft-1218-64_1000.jpg"
---

We have two dogs and three cats. Long ago we gave up on the idea of using a
normal sized water bowl and switched to a large salad bowl. This wasn't ideal
for a variety of reasons, including aesthetics, so we recently swapped it out
for a decorative fountain in the corner of our kitchen.

This has been going great and with it's 7 gallon capacity has let us go as long
as a week between refills. The downside is it's easier to forget to refill it
when it doesn't need to be done as often. Coupled with most of the water being
stored in the base, you can't tell it's getting low at a glance as you walk by.

Technology to the rescue!
-------------------------

I installed a liquid level sensor manufactured by Milone Technologies purchased
from [Adafruit](https://www.adafruit.com/?q=etape). If you go this route, note
that the enclosed models aren't just the eTape in a tube, they include a
resistance to variable voltage board with a small MCU operating at 5V. I'm
sampling this from an [Adafruit HUZZAH32](https://www.adafruit.com/product/3405),
an ESP32 based board running at 3.3 V. According to the data-sheet, a 3.3 voltage
from the 12" eTape is around 9".  In my case, this would be overflowing so I can
read the voltage directly, if that doesn't work in your situation, you'll need
a voltage divider to bring the max value below 3.3. The data-sheet also says the
sensor needs a minimum of 6 volts to run but I haven't had any trouble running it
from the 5 V USB pin on the HUZZAH32.

My source code for the ESP32 is on [Github](https://github.com/mikegrb/FountainMQTT/blob/master/FountainMQTT.ino)
but I've reproduced the important bit here:

``` c FountainMQTT.ino
int currentTotal = 0;
for (int i = 0; i < numReadings; i++) {
  currentTotal = currentTotal + analogRead(level_pin);
  delay(1000);
}

total = total - readings[readIndex];
readings[readIndex] = currentTotal / numReadings;
total = total + readings[readIndex];
int average = total / numReadings;

readIndex = readIndex + 1;
if (readIndex >= numReadings) {
  readIndex = 0;
}

int level = map(average, 0, 4095, 0, 1200); // convert to ~mm of liquid * 10
char msg[5];
snprintf(msg, 5, "%i", level);
mqtt_client.publish(topic, msg);
```

The top of the water in the base of the fountain is rather turbulent with
constant water splashing down as it drains from the base so we do some averaging.
We take a few readings one second apart and average those.  We then average that
reading with the readings over the last few minutes and fire it off to MQTT for
use in Home Assistant.

Home Assistant Configuration
----------------------------

[Home Assistant](https://www.home-assistant.io/) configuration is straight
forward with the
[MQTT sensor component](https://www.home-assistant.io/components/sensor.mqtt/).
I've added a [template sensor](https://www.home-assistant.io/components/sensor.template/)
to convert the raw sensor reading into a percentage for easier human digestion.

``` yaml config.yaml
sensor:
  - platform: mqtt
    state_topic: kitchen/fountain/level
    name: Fountain Water Level
    unit_of_measurement: mm
date: ""
slug: "/monitoring-the-dog-bowl-with-home-assistant"
    availability_topic: kitchen/fountain/available
    icon: mdi:fountain
  - platform: template
    sensors:
      fountain_fill:
        friendly_name: Fountain Fill
        entity_id: sensor.fountain_water_level
        unit_of_measurement: "%"
        value_template: "{{ ((float(states.sensor.fountain_water_level.state) - 895) / 305 * 100) | round(0) }}"
        icon_template: mdi:fountain
```

![Home Assistant Screen Shot](/assets/2018/ha_kitchen.png)
Of course the real win of Home Assistant is using automations.

``` yaml config.yaml
automation:
  - alias: 'Fountain Water Level Evening'
    trigger:
      platform: time
      at: '18:29:00'
    condition:
      condition: numeric_state
      entity_id: sensor.fountain_water_level
      below: 900
    action:
      - service: script.notify_tts_all_t
        data:
          message: The fountain is low on water.
```

Our evening voice notifications come around 6:30pm when we are likely to hear
them. `script.notify_tts_all_t` sends a push notification to my wife and my
phones, speaks the notification on Google Home and Alexa speakers throughout
the house, and posts a tweet on [my house's twitter account](https://twitter.com/mikegrbs_house). The source for that script as well
as the rest of my Home Assistant config is available on [Github](https://github.com/mikegrb/home-assistant-config).
