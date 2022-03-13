---
title: ESP8266 + MAX7219 做一个简易的自动同步的倒计时时钟
date: 2022-03-09 08:16:00
categories:
- 裤子的小经验
- 编程
- 嵌入式
tags: 
- Arduino
- ESP8266
- MAX7219
---

正好最近有大事情想需要倒计时

然后翻遍了家里的犄角旮旯，发现还有个很古老的 ESP8266 还闲置着

所以就拿过来用了

## 引脚接法

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

废话不多说，直接上代码好了

```cpp
#include "LedControl.h"
#include <ESP8266WiFi.h>
#include <ESP8266HTTPClient.h>
#include <WiFiClient.h>
#include <ArduinoJson.h>

LedControl lc = LedControl(14,13,12,1); 

unsigned long delaytime=250;
const String url = "http://quan.suning.com/getSysTime.do";

int GetDays(int iYear1, int iMonth1, int iDay1, int iYear2, int iMonth2, int iDay2)   //1. 确保 日期1 < 日期2
{
  unsigned int iDate1 = iYear1 * 10000 + iMonth1 * 100 + iDay1;
  unsigned int iDate2 = iYear2 * 10000 + iMonth2 * 100 + iDay2;
  if (iDate1 > iDate2)
  {
    iYear1 = iYear1 + iYear2 - (iYear2 = iYear1);
    iMonth1 = iMonth1 + iMonth2 - (iMonth2 = iMonth1);
    iDay1 = iDay1 + iDay2 - (iDay2 = iDay1);
  }
  
  //2. 开始计算天数
  //计算 从 iYear1年1月1日 到 iYear2年1月1日前一天 之间的天数
  int iDays = 0;
  for (int i = iYear1; i < iYear2; i++)
  {
    iDays += (IsLeapYear(i) ? 366 : 365);
  }
  
  //减去iYear1年前iMonth1月的天数
  for (int i = 1; i < iMonth1; i++)
  {
    switch (i)
    {
    case 1: case 3: case 5: case 7: case 8: case 10: case 12:
      iDays -= 31;
      break;
    case 4: case 6: case 9: case 11:
      iDays -= 30;
      break;
    case 2:
      iDays -= (IsLeapYear(iYear1) ? 29 : 28);
      break;
    }
  }
  //减去iYear1年iMonth1月前iDay1的天数
  iDays -= (iDay1 - 1);
//加上iYear2年前iMonth2月的天数
for (int i = 1; i < iMonth2; i++)
  {
    switch (i)
    {
    case 1: case 3: case 5: case 7: case 8: case 10: case 12:
      iDays += 31;
      break;
    case 4: case 6: case 9: case 11:
      iDays += 30;
      break;
    case 2:
      iDays += (IsLeapYear(iYear2) ? 29 : 28);
      break;
    }
  }
  //加上iYear2年iMonth2月前iDay2的天数
  iDays += (iDay2 - 1);
  return iDays;
}

  //判断给定年是否闰年
bool IsLeapYear(int iYear)
{
  if (iYear % 100 == 0)
    return ((iYear % 400 == 0));
  else
    return ((iYear % 4 == 0));
}

String GetTime() {
  if (WiFi.status() == WL_CONNECTED) {
    WiFiClient client;
    HTTPClient http;

    http.begin(client, url.c_str());

    int httpResponseCode = http.GET();
    String res = "202103082332";

    if (httpResponseCode > 0) {
      Serial.println("HTTP Response code: " + String(httpResponseCode));
      res = http.getString();
      Serial.println("Got: " + res);
    } else {
      Serial.println("Error code: " + String(httpResponseCode));
    }
    
    http.end();

    return DateHandle(res);
  } else {
    Serial.println("WiFi Connected Error");
    return "202103082332";
  }
}

String DateHandle(String payload) {
  int jsonBeginAt = payload.indexOf("{");
  int jsonEndAt = payload.lastIndexOf("}");

  if (jsonBeginAt != -1 && jsonEndAt != -1) {
    payload = payload.substring(jsonBeginAt, jsonEndAt + 1);
    const size_t capacity = JSON_OBJECT_SIZE(2) + 60;
    DynamicJsonDocument doc(capacity);
    deserializeJson(doc, payload);

    const char* sysTime1 = doc["sysTime1"];
    const char* sysTime2 = doc["sysTime2"];

    return sysTime1;
  }
  return "202103082332";
}

void setup() {
  Serial.begin(9600);
  
  WiFi.begin("WiFiName", "WiFiPassword");

  Serial.println("Waiting for connecting ...");

  int i = 0;
  while (WiFi.status() != WL_CONNECTED) {
    delay(100);
    Serial.print(".");
  }
  
  lc.shutdown(0,false);
  lc.setIntensity(0,8);
  lc.clearDisplay(0);
}

void ShowDigits(int num) {
  lc.clearDisplay(0);
  String numStr = String(num);
  Serial.println("There is " + numStr + " day(s) to go");
  if (numStr.length() == 1) {
    lc.setDigit(0, 4, numStr.toInt(), false);
    lc.setChar(0, 3, 'd', false);
  } else if (numStr.length() == 2) {
    lc.setDigit(0, 4, numStr.substring(0, 1).toInt(), false);
    lc.setDigit(0, 3, numStr.substring(1, 2).toInt(), false);
  } else if (numStr.length() == 3) {
    lc.setDigit(0, 5, numStr.substring(0, 1).toInt(), false);
    lc.setDigit(0, 4, numStr.substring(1, 2).toInt(), false);
    lc.setDigit(0, 3, numStr.substring(2, 3).toInt(), false);
    lc.setChar(0, 2, 'd', false);
  } else if (numStr.length() == 4) {
    lc.setDigit(0, 5, numStr.substring(0, 1).toInt(), false);
    lc.setDigit(0, 4, numStr.substring(1, 2).toInt(), false);
    lc.setDigit(0, 3, numStr.substring(2, 3).toInt(), false);
    lc.setDigit(0, 2, numStr.substring(3, 4).toInt(), false);
  }
}

void loop() { 
  String Date = GetTime();
  int NowYear = Date.substring(0, 4).toInt();
  int NowMonth = Date.substring(4, 6).toInt();
  int NowDay = Date.substring(6, 8).toInt();
  Serial.println("Now time is " + String(NowYear) + "." + String(NowMonth) + "." + String(NowDay));
  ShowDigits(GetDays(NowYear, NowMonth, NowDay, 2022, 06, 20));

  delay(3600000);
}
```

1. 计算两日期之差代码来源：[Arduino如何实现日期差的计算-Arduino中文社区 - Powered by Discuz!](https://www.arduino.cn/thread-22107-1-1.html)

2. 替换 WiFiName 和 WiFiPassword 成自己家的 WiFi 就可以了

3. 然后修改倒数第五行的后面那个日期，就是倒计时的目标日期

4. 默认休眠一个小时自动获取

5. 当前时间获取 API：[http://quan.suning.com/getSysTime.do](http://quan.suning.com/getSysTime.do)

6. 为了偷懒，请求失败我没有重新请求，只单单返回了一个固定的时间

## 效果展示

为了居中

- 1 位数字会显示：Xd
- 2 位数字会显示：XX
- 3 位数字会显示：XXXd
- 4 位数字会显示：XXXX
- 5 位数字及以上没有显示

![成品效果图](https://cdn.jsdelivr.net/gh/Rotten-LKZ/cdn@main/images/content/max7219-83a9a7.png)

## 预告

关于这个倒计时其实我已经做了另一个版本的了，至于什么时候发出来敬请期待。
