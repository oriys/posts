---
title: "基础::计算机系统中的时间"
date: 2021-10-26T22:32:54+08:00
draft: false
---

<!-- ![ChFaWMj](https://i.imgur.com/ChFaWMj.png) -->

### 背景

时间是计算机系统中一个非常重要的概念，很多系统调用与时间有关，很多定时任务依赖时间，甚至内核本身也需要定时器按照一定频率来产生时钟中断来驱动它能够正常运行下去。

### 硬件时间

操作系统需要在硬件的帮助下才能管理时间，体系结构主要提供了两种硬件设备，一个是时钟源设备，位于主板上的一小块存储空间，有单独的供电，即使断电也可以保持计时。当操作系统启动时，会从这里读取时间，把系统时间与硬件时间设置成一致，此后两个时钟独立，操作系统维护自己的时钟，因为查看硬件既慢又复杂。另一个硬件是时钟事件设备，用于单次或周期性触发系统中断，每次中断，操作系统会进行刷新系统时间，管理进程时间片等操作。

### 系统时间

计算机科学与计算机编程中, 系统时间表示在计算机系统中的时间与日期。通常用系统时钟从某个时间起点的嘀嗒数。时间起点就是上文提到的在启动时读取的硬件时间，滴答数则是每次中断的累加，定时器是按照一定频率产生中断的，所以系统是可以知道两次中断之间过去了多久的。类 Unix 系统通常采用 Unix Epoch 来表示时间。

### 时间同步

因为操作系统采用了自己维护时间的方式，所以在运行一段时间后必然会产生硬件时间和软件时间不一致的情况，系统时钟会开始落后。这时候可以使用“hwclock“将两者进行同步，通过指定不同的参数将硬件时间同步到系统时间或者把系统时间同步到硬件时间。

### NTP 协议

在单机系统中尚且可以解决时间不一致的问题，但是无法解决时间不准确的问题。于是就有了 NTP 协议，全称网络时间协议（Network Time protocol）。通过与另一台保持准确时间的设备通信来调整自身的系统时间，这就是 NTP 协议所做的事情，协议基于 UDP 实现，通过该协议，可以将时间误差保持在 UTC 时间的毫秒级误差。操作系统会周期性通过 NTP 协议更新系统时间。

### UTC 时间

协调世界时是最主要的世界时间标准，其以原子时秒长为基础，在时刻上尽量接近于格林威治标准时间。因为在英国的缩写是 CUT，而法国的缩写是 TUC，所以双方妥协一下，缩写统一成 UTC 时间。协调世界时是世界上调节时钟和时间的主要时间标准，它与 0 度经线的平太阳时相差不超过 1 秒，并不遵守夏令时。协调世界时是最接近格林威治标准时间（GMT）的几个替代时间系统之一。对于大多数用途来说，UTC 时间被认为能与 GMT 时间互换，但 GMT 时间已不再被科学界所确定。这套时间系统被应用于许多互联网和万维网的标准中，例如，网络时间协议（NTP, Network Time Protocol）就是协调世界时在互联网中使用的一种方式。

### GMT 时间

格林尼治平均时间（英语：Greenwich Mean Time，GMT）是指位于英国伦敦郊区的皇家格林尼治天文台当地的平太阳时，因为本初子午线被定义为通过那里的经线。目前使用的世界时测算标准又称 UT1。在 UT1 之前人们曾使用过 UT0，但由于 UT0 没有考虑极移导致的天文台地理坐标变动的问题，因此测出的世界时不准确，现在已经不再被使用。在 UT1 之后，由于人们发现，因为地球自转本身不均匀的问题，UT1 定义的时间的流逝仍然不均匀，于是人们又发展了一些对 UT1 进行平滑处理后的时间标准，包括 UT1R 和 UT2，但它们都未能彻底解决定义的时间的流逝不均匀的问题，这些时间标准现在都不再被使用。

后来的人们为了解决地球自转产生了时间流逝不均匀的问题，开始采用原子钟定义时间。人们首先用全世界的原子钟共同为地球确立了一个均匀流动的时间，称为国际原子时（International Atomic Time, TAI）。然后，为了使定义的时间与地球自转相配合，人们通过在 TAI 的基础上不定期增减闰秒的方式，使定义的时间与世界时（UT1）保持差异在 0.9 秒以内，这样定义的时间就是协调世界时（Coordinated Universal Time, UTC）。UTC 是目前全世界使用的时间标准。UTC 与 UT1 之间的差异被称为 DUT1。

### 时区

时区是地球上的区域使用同一个时间定义。以前，人们通过观察太阳的位置（时角）决定时间，这就使得不同经度的地方的时间有所不同（地方时）。1863 年，首次使用时区的概念。通过设立一个区域的标准时间部分地解决了这个问题。世界各国位于地球不同位置上，因此不同国家，特别是东西跨度大的国家日出、日落时间必定有所偏差。这些偏差就是所谓的时差。

计算机系统中时常采用 UTC 来表示时间，只需要在时间后面添加不带空格的“Z“即可，“Z“是协调世界时中 0 时区的标志。因此，“09:30 UTC“就写作“09:30Z“或是“0930Z“。“14:45:15 UTC“则为“14:45:15Z“或“144515Z“。UTC 偏移量用以下形式表示：±[hh]:[mm]、±[hh][mm]、或者 ±[hh]。如果所在区时比协调世界时早 1 个小时（例如柏林冬季时间），那么时区标识应为“+01:00“、“+0100“或者直接写作“+01“。这也同上面的“Z“一样直接加在时间后面。“UTC+8“表示当协调世界时（UTC）时间为凌晨 2 点的时候，当地的时间为 2+8 点，即早上 10 点。

时区通常都用字母缩写形式来表示，例如“EST、WST、CST”等。但是它们并不是 ISO 8601 标准的一部分，不应单独用它们作为时区的标识。一些缩写可能意义模糊，例如“BST”应当是英国夏令时，但在 1968 年到 1971 年间被重命名为“英国标准时”，这只是因为立法者不愿称其为中欧时间。在该法案中还确认英国的标准时间仍然为格林威治标准时。

一般来说，如果你有需求要在计算机中表示时间，尽量用 UTC 表示法，用 UTC 偏移量表示带时区的时间，如果不带任何时区信息，默认 UTC 零时区。有部分数据库支持带时区的时间存储。

### 夏令时

夏令时，表示为了节约能源，人为规定时间的意思。一般在天亮早的夏季人为将时间调快一小时，可以使人早起早睡，减少照明量，以充分利用光照资源，从而节约照明用电。大多数实行过夏令时的国家都废弃了这个无聊的制度，然而世界上有一些部分国家还在实行夏令时，包括美国、加拿大、部分西欧国家，但是美国和加拿大并非全境都实行夏令时，而是由各个联邦政府自行决定。一般计算机会自动处理夏令时，不需要开发人员干预。

### 计算机相关的时间标准

目前有两个标准用来在计算机和网络中表示时间，分别是 ISO 8601 和 rfc3399，两个标准有部分重合，不存在一个可以完全取代另一个，一个明显区别就是是否支持表示 24 点。时间表示是编程过程中非常值得吐槽的点，因为历史原因，至今各个编程语言没有把时间统一起来，所以在开发过程中时常可以看见各种不同的时间格式，有时候很难想到正确解析时间，以及还有用时间戳表示时间的，这是非常善良的，另时间戳是没有时区概念的，地球上任何一个地点在同一个时刻获取的时间戳都是一样的。目前 Java 是采用了 ISO-8601。

### 格式符号说明

| Symbol | Meaning                    | Presentation | Examples                                      |
| ------ | -------------------------- | ------------ | --------------------------------------------- |
| G      | era                        | text         | AD; Anno Domini; A                            |
| u      | year                       | year         | 2004; 04                                      |
| y      | year-of-era                | year         | 2004; 04                                      |
| D      | day-of-year                | number       | 189                                           |
| M/L    | month-of-year              | number/text  | 7; 07; Jul; July; J                           |
| d      | day-of-month               | number       | 10                                            |
| g      | modified-julian-day        | number       | 2451334                                       |
| Q/q    | quarter-of-year            | number/text  | 3; 03; Q3; 3rd quarter                        |
| Y      | week-based-year            | year         | 1996; 96                                      |
| w      | week-of-week-based-year    | number       | 27                                            |
| W      | week-of-month              | number       | 4                                             |
| E      | day-of-week                | text         | Tue; Tuesday; T                               |
| e/c    | localized day-of-week      | number/text  | 2; 02; Tue; Tuesday; T                        |
| F      | day-of-week-in-month       | number       | 3                                             |
| a      | am-pm-of-day               | text         | PM                                            |
| h      | clock-hour-of-am-pm (1-12) | number       | 12                                            |
| K      | hour-of-am-pm (0-11)       | number       | 0                                             |
| k      | clock-hour-of-day (1-24)   | number       | 24                                            |
| H      | hour-of-day (0-23)         | number       | 0                                             |
| m      | minute-of-hour             | number       | 30                                            |
| s      | second-of-minute           | number       | 55                                            |
| S      | fraction-of-second         | fraction     | 978                                           |
| A      | milli-of-day               | number       | 1234                                          |
| n      | nano-of-second             | number       | 987654321                                     |
| N      | nano-of-day                | number       | 1234000000                                    |
| V      | time-zone ID               | zone-id      | America/Los_Angeles; Z; -08:30                |
| v      | generic time-zone name     | zone-name    | Pacific Time; PT                              |
| z      | time-zone name             | zone-name    | Pacific Standard Time; PST                    |
| O      | localized zone-offset      | offset-O     | GMT+8; GMT+08:00; UTC-08:00                   |
| X      | zone-offset 'Z' for zero   | offset-X     | Z; -08; -0830; -08:30; -083015; -08:30:15     |
| x      | zone-offset                | offset-x     | +0000; -08; -0830; -08:30; -083015; -08:30:15 |
| Z      | zone-offset                | offset-Z     | +0000; -0800; -08:00                          |
| p      | pad next                   | pad modifier | 1                                             |
| '      | escape for text            | delimiter    |                                               |
| ''     | single quote               | literal      | '                                             |
| [      | optional section start     |              |                                               |
| ]      | optional section end       |              |                                               |
| #      | reserved for future use    |              |                                               |
| {      | reserved for future use    |              |                                               |
| }      | reserved for future use    |              |                                               |

java 并不完整支持以上所有字符，以下是 java 支持的部分

```java
    int PATTERN_ERA                  =  0; // G
    int PATTERN_YEAR                 =  1; // y
    int PATTERN_MONTH                =  2; // M
    int PATTERN_DAY_OF_MONTH         =  3; // d
    int PATTERN_HOUR_OF_DAY1         =  4; // k
    int PATTERN_HOUR_OF_DAY0         =  5; // H
    int PATTERN_MINUTE               =  6; // m
    int PATTERN_SECOND               =  7; // s
    int PATTERN_MILLISECOND          =  8; // S
    int PATTERN_DAY_OF_WEEK          =  9; // E
    int PATTERN_DAY_OF_YEAR          = 10; // D
    int PATTERN_DAY_OF_WEEK_IN_MONTH = 11; // F
    int PATTERN_WEEK_OF_YEAR         = 12; // w
    int PATTERN_WEEK_OF_MONTH        = 13; // W
    int PATTERN_AM_PM                = 14; // a
    int PATTERN_HOUR1                = 15; // h
    int PATTERN_HOUR0                = 16; // K
    int PATTERN_ZONE_NAME            = 17; // z
    int PATTERN_ZONE_VALUE           = 18; // Z
    int PATTERN_WEEK_YEAR            = 19; // Y
    int PATTERN_ISO_DAY_OF_WEEK      = 20; // u
    int PATTERN_ISO_ZONE             = 21; // X
    int PATTERN_MONTH_STANDALONE     = 22; // L
```

### 常见格式化

| Pattern                                           | Format                                           |
| ------------------------------------------------- | ------------------------------------------------ |
| yyyy-MM-dd                                        | 2021-10-27                                       |
| yyyy-MM-dd hh:mm:ss                               | 2021-10-27 05:06:55                              |
| yyyy-MM-dd hh:mm:ss z                             | 2021-10-27 05:11:21 CST                          |
| yyyy-MM-dd hh:mm:ss zzzzzz                        | 2021-10-27 05:12:13 China Standard Time          |
| yyyy-MM-dd'T'hh:mm:ss                             | 2021-10-27T16:58:35                              |
| yyyy-MM-dd'T'hh:mm:ss.SSS                         | 2021-10-27T16:58:46.445                          |
| yyyy-MM-dd'T'hh:mm:ss.SSSZ                        | 2021-10-27T16:58:46.445+0800                     |
| yyyy-MM-dd'T'hh:mm:ss.SXXX                        | 2021-10-27T16:58:46.445CST                       |
| yyyy-MM-dd'T'hh:mm:ss.SSSXXX                      | 2021-10-27T05:05:31.766+08:00                    |
| yyyyyyyy-MMMM-dddd hhhhhh:mmmmmm:ssssss ZZZZZZZZZ | 00002021-October-0027 000005:000026:000004 +0800 |

### yyyy 与 YYYY 的区别

YYYY 是以周来计算年的，意思是当天所在周属于的年份，一周从周日开始算计算，周六结束，只要本周跨年，那么这一周就算下一年的。 也就是说：年份如果用 Y 会是这周的年份，y 才是标准的年份。
