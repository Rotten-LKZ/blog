---
title: ESP8266 + MAX7219 做一个精确到秒的简易自动同步的倒计时时钟
date: 2022-03-14 18:17:04
categories:
- 裤子的小经验
- 编程
- 嵌入式
tags: 
- Arduino
- ESP8266
- MAX7219
---

书接上文。我们上一回做了一个比较鸡肋的只能具体到天的倒计时，那种倒计时根本体会不到时间的变化🙄，所以做了个能精确到秒的倒计时。

## 引脚接法

跟上一回一样，还是怼在一起的接法

其实没什么好说的，如果会接的话根据自己的想法接上去然后修改代码就好了

当然，给出来的代码是这样接的（左 MAX7219 右 ESP8266）：

>VCC --- 3V<br/>
>GND --- GND<br/>
>DIN --- D5<br/>
>CS --- D6<br/>
>CLK --- D7

至于为什么我会这么接

因为刚刚好可以一排线怼一起了


![真·怼一起](https://cdn.jsdelivr.net/gh/Rotten-LKZ/cdn@main/images/content/esp8266-f8bae8.png)

## 具体实现

```cpp
#include "LedControl.h"
#include <ESP8266WiFi.h>
#include <ESP8266HTTPClient.h>
#include <WiFiClient.h>
#include <ArduinoJson.h>

int nowTime = -1;
int loopTimes = 0;
bool showDaysLeft = false;

LedControl lc = LedControl(14, 13, 12, 1);
const String url = "http://api.m.taobao.com/rest/api3.do?api=mtop.common.getTimestamp";

void getTime() {
  if (WiFi.status() == WL_CONNECTED) {
    WiFiClient client;
    HTTPClient http;

    http.begin(client, url.c_str());

    int httpResponseCode = http.GET();
    String res = "1646884954100";

    if (httpResponseCode > 0) {
      Serial.print("HTTP Response code: ");
      Serial.println(httpResponseCode);

      res = http.getString();
      
      Serial.print("Got: ");
      Serial.println(res);
    } else {
      Serial.print("Error code: ");
      Serial.println(httpResponseCode);
    }

    http.end();

    int jsonBeginAt = res.indexOf("{");
    int jsonEndAt = res.lastIndexOf("}");

    if (jsonBeginAt != -1 && jsonEndAt != -1) {
      res = res.substring(jsonBeginAt, jsonEndAt + 1);
      DynamicJsonDocument doc(200);
      deserializeJson(doc, res);

      const char* resTime = doc["data"]["t"];

      Serial.print("resTime: ");
      Serial.println(String(resTime));

      nowTime = String(resTime).substring(0, 10).toInt();
    }
  }
}

void showDigits(int targetTime) {
  lc.clearDisplay(0);

  const int diffSeconds = targetTime - nowTime;

  Serial.print("Now time: ");
  Serial.println(nowTime);

  Serial.print("Diff seconds: ");
  Serial.println(diffSeconds);
  
  if (diffSeconds < 0) {
    for (int i = 0; i < 8; i++) {
      lc.setLed(0, i, 0, false);
    }
    return;
  }

  if (showDaysLeft) {
    // Show how many days to go
    String daysStr = String(int(diffSeconds / 86400));
    if (daysStr.length() == 2) {
      lc.setDigit(0, 4, daysStr.substring(0, 1).toInt(), false);
      lc.setDigit(0, 3, daysStr.substring(1, 2).toInt(), false);
    } else if (daysStr.length() == 1) {
      lc.setDigit(0, 4, daysStr.toInt(), false);
    } else if (daysStr.length() == 3) {
      lc.setDigit(0, 5, daysStr.substring(0, 1).toInt(), false);
      lc.setDigit(0, 4, daysStr.substring(1, 2).toInt(), false);
      lc.setDigit(0, 3, daysStr.substring(2, 3).toInt(), false);
    } else if (daysStr.length() == 4) {
      lc.setDigit(0, 5, daysStr.substring(0, 1).toInt(), false);
      lc.setDigit(0, 4, daysStr.substring(1, 2).toInt(), false);
      lc.setDigit(0, 3, daysStr.substring(2, 3).toInt(), false);
      lc.setDigit(0, 2, daysStr.substring(3, 4).toInt(), false);
    }
  } else {
    // Show how long the day will end
    String hours = String(int(diffSeconds % 86400 / 3600));
    String minutes = String(int(diffSeconds % 86400 % 3600 / 60));
    String seconds = String(int(diffSeconds % 86400 % 3600 % 60));

    if (hours.length() == 1) {
      lc.setDigit(0, 7, 0, false);
      lc.setDigit(0, 6, hours.toInt(), false);
    } else {
      lc.setDigit(0, 7, hours.substring(0, 1).toInt(), false);
      lc.setDigit(0, 6, hours.substring(1, 2).toInt(), false);
    }
    
    if (minutes.length() == 1) {
      lc.setDigit(0, 4, 0, false);
      lc.setDigit(0, 3, minutes.toInt(), false);
    } else {
      lc.setDigit(0, 4, minutes.substring(0, 1).toInt(), false);
      lc.setDigit(0, 3, minutes.substring(1, 2).toInt(), false);
    }
    
    if (seconds.length() == 1) {
      lc.setDigit(0, 1, 0, false);
      lc.setDigit(0, 0, seconds.toInt(), false);
    } else {
      lc.setDigit(0, 1, seconds.substring(0, 1).toInt(), false);
      lc.setDigit(0, 0, seconds.substring(1, 2).toInt(), false);
    }
  }
}

void setup() {
  Serial.begin(9600);

  WiFi.begin("WiFiName", "WiFiPasssword");

  Serial.println("Waiting for connecting ...");

  int i = 0;
  while (WiFi.status() != WL_CONNECTED) {
    delay(100);
    Serial.print(".");
  }

  lc.shutdown(0, false);
  lc.setIntensity(0, 8);
  lc.clearDisplay(0);
}

void loop() {
  if (nowTime == -1 || loopTimes == 86400) {
    loopTimes = 0;
    getTime();
  }
  if (loopTimes % 8 == 0) {
    // Switch what to display every 8 seconds
    showDaysLeft = !showDaysLeft;
  }

  showDigits(1655683200);

  loopTimes++;
  nowTime++;
  delay(1000);
}
```

1. 每八秒会轮换显示天数或者精确到的秒数
2. 每 86400 秒会自动同步一次时间（其实就是 24 小时也就是 1 天）
3. 同步时间 API：<http://api.m.taobao.com/rest/api3.do?api=mtop.common.getTimestamp>
4. 对了用的时候别忘了改里面 `WiFiName` 和 `WiFiPassword` 为自己家的 WiFi 账号密码

## 效果展示

*这次套了个用不上的相框（不重要），背景图片是 [熊熊勇闯异世界](https://kumakumakumabear.com/)，风傲天！！！女主超可爱！！！超强！！！超爽！！！*

咳咳跑题了。

![天数显示时效果图](https://cdn.jsdelivr.net/gh/Rotten-LKZ/cdn@main/images/content/timer2effort-d5b8a4.jpg)

![秒钟显示时效果图](https://cdn.jsdelivr.net/gh/Rotten-LKZ/cdn@main/images/content/timer2effort2-ad4a1c.jpg)
