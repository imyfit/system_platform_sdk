# SDK接口文档

| 版本  | 日期     | 作者   | 备注            |
| ----- | -------- | ------ | --------------- |
| 1.0.0 | 20220418 | 舒德军 | 初步完善SDK文档 |

**致开发者：**

1. 若需要硬件请联系销售人员,  https://www.imyfit.com
2. 本PDF协议文档可能不是最新版本
3. 若协议有更新，不再另行通知，强烈建议开发者到 https://imyfit.gitee.io 获取最新协议
4. 若对协议不理解或发现协议有bug请到issues区域提出（或者提交一个 pull request），愿我们的付出对您的开发事半功倍。

------

## 1.简介

### 1.1.概述

该文档用于使用者对接深圳吾控健康科技有限公司的终端设备，通过蓝牙网关，UWB，NB以及CAT1等数据方式进行数据的对接，便于客户直接使用解析后的数据，用于自己的业务系统，免去客户因解析数据带来诸多麻烦，同时便于客户节约大把时间专注于自己的业务系统。

------

## 2.相关数据类定义

### 2.1.MqttType枚举

该枚举用于判断当需要MQTT服务器发送数据的时候，判断发送给那个服务器，如果客户的服务器分了两个（本地针对室内，云端针对室外），如果只有一个服务器则不需要判断，直接发送即可。

| 属性  | 含义                                         |
| :---: | -------------------------------------------- |
|  NB   | 针对NB,CAT1数据的服务器                      |
| LOCAL | 针对室内定位数据的服务器如：蓝牙、UWB、AOA等 |

### 2.2.MessageFlag枚举

该枚举用于判断所输出的数据类型是什么，有一个属性值type（类型String）。

|    属性     |   type值    | 含义                                       |
| :---------: | :---------: | ------------------------------------------ |
|   INDOOR    |   indoor    | 室内数据，即蓝牙、aoa、uwb、beacon定位数据 |
|   OUTDOOR   |   outdoor   | NB,CAT1上传的实时定位数据                  |
| HEART_RATE  |  heartRate  | NB,CAT1定时上传的心率包                    |
|    SLEEP    |    sleep    | NB,CAT1上传的睡眠数据                      |
|  LOCATION   |  location   | NB,CAT1上传的纯定位数据                    |
|    WI-FI    |    wi-fi    | NB,CAT1上传的Wifi定位数据                  |
|   SYSTEM    |   system    | NB,CAT1各种事件数据                        |
|   CALORIE   |   calorie   | NB,CAT1上传的热量包数据包                  |
|    SPO2     |    spo2     | NB,CAT1上传的血氧数据包                    |
| TEMPERATURE | temperature | NB,CAT1上传的温度数据包                    |
|     BP      |     bp      | NB,CAT1上传的血压数据包                    |
|  SPORT_CWM  |  sport-cwm  | CWM运动数据                                |

### 2.3.LocationType枚举

该枚举用于判断定位类型使用，主要用于判断使用那种介子定位，有两个属性code（Integer类型），type（String类型）

|   属性    | code |   type    | 含义                          |
| :-------: | :--: | :-------: | ----------------------------- |
|   OTHER   |  -1  |   other   | 其他定位类型                  |
|    IOT    |  1   |    iot    | iot定位，即移动、联通基站定位 |
|   CAT1    |  2   |   cat1    | cat1定位                      |
|   WIFI    |  3   |   wifi    | 通过wifi定位                  |
|    GPS    |  4   |    gps    | gps定位                       |
| BLUETOOTH |  5   | bluetooth | 蓝牙网关定位                  |
|    UWB    |  6   |    uwb    | UWB网关定位                   |
|    AOA    |  7   |    aoa    | AOA网关定位                   |
|  BEACON   |  8   |  beacon   | beacon锚点定位                |

**注：**该枚举还包含了一个方法public static LocationType find (Integer code)，用于查找对应的定位类型

------

## 3.JAR包对接

客户使用**java**或者**kotlin**开发业务系统，则可以直接导入jar包进行开发使用。

以maven项目为例：

将我方给的jar包导入自己本地的maven仓库，然后在项目中导入使用

```xml
<dependency>
    <groupId>com.imyfit.terminal.tool</groupId>
    <artifactId>imyfit-terminal-tool</artifactId>
    <version>1.0.0</version>
</dependency>
```

**注：**版本根据我方给的jar包版本而定

1. ### 导入后，需要做以下操作，java为例：

  1. 新建一个CallBack类继承AbstractImyFitCallBack抽象类

  	```java
  	import org.jetbrains.annotations.NotNull;
  	
  	public class CallBack extends AbstractImyFitCallBack {
  	
  	    /**
  	     * 发布数据到mqtt服务器
  	     * 发送消息到mqtt服务器，type的类型时LOCAL，原封不动转发即可
  	     * @param type 发送到哪个服务器
  	     * @param topic 发布主题
  	     * @param message 发送的消息类型
  	     */
  	    public void sendMessage(@NotNull MqttType type, @NotNull String topic, @NotNull String message, int qos) {
  	        
  	    }
  	
  	    /**
  	     * 处理消息，在此处获得消息并进行业务处理
  	     * @param flag 消息类型
  	     * @param message 具体的消息类型
  	     */
  	    public void processMessage(@NotNull MessageFlag flag, @NotNull Object message) {
  	        //根据 flag 处理不通类型的message，后续将讲解
  	    }
  	}
  	```

    2. 新建一个配置类

  	```java
  	import com.imyfit.terminal.option.Option
  	Option option=new Option();
  	// 是否过滤蓝牙网关，默认是过滤
  	option.setFilter(FilterType.True);
  	//回调函数，即第一步的继承类
  	option.setCallBack(new CallBack());
  	// 是否去重，目的是用于去除统一条上报数据中同一设备多条数据
  	option.setDistinct(true);
  	//网关上传数据时间，默认1秒钟最大10秒钟
  	option.setCycle(1);
  	```

    3. 在接收mqtt消息的地方处理数据

  	```java
  	new TopicMessage(topic,message.toString(),option);
  	```
  	
  	Mqtt处理数据例子
  	
  	```java
  	import com.imyfit.terminal.entity.FilterType;
  	import com.imyfit.terminal.entity.message.TopicMessage;
  	import com.imyfit.terminal.option.Option;
  	import org.eclipse.paho.client.mqttv3.IMqttDeliveryToken;
  	import org.eclipse.paho.client.mqttv3.MqttCallback;
  	import org.eclipse.paho.client.mqttv3.MqttMessage;
  	
  	public class ImyFitMqttCallBack implements MqttCallback {
  	    private Option option() {
  	        Option option = new Option();
  	        // 是否过滤蓝牙网关，默认是过滤
  	        option.setFilter(FilterType.True);
  	        //回调函数，即第一步的继承类
  	        option.setCallBack(new CallBack());
  	        // 是否去重，目的是用于去除统一条上报数据中同一设备多条数据
  	        option.setDistinct(true);
  	        //网关上传数据时间，默认1秒钟最大10秒钟
  	        option.setCycle(1);
  	        return option;
  	    }
  	
  	    @Override
  	    public void connectionLost(Throwable throwable) {
  	        // 处理连接丢失
  	    }
  	
  	    @Override
  		public void messageArrived(String s, MqttMessage mqttMessage) throws Exception {
  	    	// 此次用线程处理，因为数据量可能很大，不要做阻塞式处理
  	        new TopicMessage(s, mqttMessage.toString(), option());
  	    }
  	
  	    @Override
  	    public void deliveryComplete(IMqttDeliveryToken iMqttDeliveryToken) {
  	
  	    }
  	}
  	```

    4. UWB、AOA处理消息

  	该两种方式接收消息，是采用UDP协议接收、解析、处理消息的
  	
  	```java
  	private Option option(type) {
  	    Option option = new Option();
  	    //回调函数，即第一步的继承类
  	    option.setCallBack(new CallBack());
  	    /**
  	    * 设备类型，UWB和AOA必用，蓝牙忽略该字段
  	    * UWB：具体类型的type
  	    * AOA：固定值为：大写AOA
  	    */
  	    option.type = type;
  	    return option;
  	}
  	// 再handler接收到的消息处理
  	new TopicMessage(s, messag, option());问问
  	```


### 3.1.室内数据对接

即使用蓝牙网关、AOA网关、UWB网关以及Beacon获取到的生理健康数据等信息

标识为：flag == MessageFlag.INDOOR

以下各个设备的属性的具体含义

#### 3.1.1. B10系列

message对象是**B10Data**直接调用属性处理：

|       属性       |  类型   | 含义                                                         |
| :--------------: | :-----: | ------------------------------------------------------------ |
|       mac        | String  | 终端设备mac地址                                              |
|   terminalType   | String  | 终端设备类型（B10C）                                         |
|     battery      | Integer | 电池电量                                                     |
|    gatewayMac    | String  | 蓝牙网关：蓝牙网关mac地址<br>AOA网关：AOA网关mac地址<br>Beancon：Beancon的mac地址 |
|       rssi       | Integer | 网关的rssi值                                                 |
|    localType     | String  | 定位类型，查看2.3                                            |
|    createTime    | String  | 创建时间（2022-04-08 00:00:00）                              |
|   locationTime   | String  | 数据上传的时间                                               |
|    heartRate     | Integer | 心率                                                         |
|       step       | Integer | 步数                                                         |
|      silent      | String  | 运动状态：动、静                                             |
|       wear       | String  | 是否佩戴：佩戴、未佩戴                                       |
|      sleep       | String  | 睡眠状态：浅睡、深睡、清醒                                   |
| temperatureEnvn  | Double  | 环境温度                                                     |
| temperatureSkin  | Double  | 体表温度                                                     |
| temperatureCore  | Double  | 体核温度                                                     |
|        x         | Double  | 定位数据：A0A值才有用                                        |
|        y         | Double  | 定位数据：A0A值才有用                                        |
|     systolic     | Integer | 收缩压                                                       |
|    diastolic     | Intger  | 舒张压                                                       |
|       spo2       | Integer | 血氧                                                         |
|       hrv        | Integer | HRV值                                                        |
| restingHeartRate | Integer | 静息心率                                                     |
|       sos        | String  | SOS求救：on、off                                             |
|    sportMode     | String  | 是否开启运动模式\|：on、off                                  |
|       fall       | String  | 是否跌倒：跌倒、正常                                         |

#### 3.1.2.X6

message对象是**X6Data**直接调用属性处理：

|       属性       |  类型   | 含义                                                         |
| :--------------: | :-----: | ------------------------------------------------------------ |
|       mac        | String  | 终端设备mac地址                                              |
|   terminalType   | String  | 终端设备类型（B10C）                                         |
|     battery      | Integer | 电池电量                                                     |
|    gatewayMac    | String  | 蓝牙网关：蓝牙网关mac地址<br>AOA网关：AOA网关mac地址<br>Beancon：Beancon的mac地址 |
|       rssi       | Integer | 网关的rssi值                                                 |
|    localType     | String  | 定位类型，查看2.3                                            |
|    createTime    | String  | 创建时间（2022-04-08 00:00:00）                              |
|   locationTime   | String  | 数据上传的时间                                               |
|    heartRate     | Integer | 心率                                                         |
|       step       | Integer | 步数                                                         |
|      silent      | String  | 运动状态：动、静                                             |
|       wear       | String  | 是否佩戴：佩戴、未佩戴                                       |
|      sleep       | String  | 睡眠状态：浅睡、深睡、清醒                                   |
| temperatureEnvn  | Double  | 环境温度                                                     |
| temperatureSkin  | Double  | 体表温度                                                     |
| temperatureCore  | Double  | 体核温度                                                     |
|        x         | Double  | 定位数据：A0A才有用                                          |
|        y         | Double  | 定位数据：A0A值才有用                                        |
|     systolic     | Integer | 收缩压                                                       |
|    diastolic     | Intger  | 舒张压                                                       |
|       spo2       | Integer | 血氧                                                         |
|      charge      | Integer | 充电状态：充电中、未充电                                     |
| chargeIntoStatus | Integer | 充电器：充电器插入、充电器未插入                             |
|       sos        | String  | SOS求救：on、off                                             |
|    sportMode     | String  | 是否开启运动模式\|：on、off                                  |

#### 3.1.3. X3W

message对象是**X3WData**直接调用属性处理：

|      属性       |  类型   | 含义                                                         |
| :-------------: | :-----: | ------------------------------------------------------------ |
|       mac       | String  | 终端设备mac地址                                              |
|  terminalType   | String  | 终端设备类型（B10C）                                         |
|     battery     | Integer | 电池电量                                                     |
|   gatewayMac    | String  | 蓝牙网关：蓝牙网关mac地址<br>AOA网关：AOA网关mac地址<br>Beancon：Beancon的mac地址 |
|      rssi       | Integer | 网关的rssi值                                                 |
|    localType    | String  | 定位类型，查看2.3                                            |
|   createTime    | String  | 创建时间（2022-04-08 00:00:00）                              |
|  locationTime   | String  | 数据上传的时间                                               |
|    heartRate    | Integer | 心率                                                         |
|      step       | Integer | 步数                                                         |
|     silent      | String  | 运动状态：动、静                                             |
|      wear       | String  | 是否佩戴：佩戴、未佩戴                                       |
|      sleep      | String  | 睡眠状态：浅睡、深睡、清醒                                   |
| temperatureEnvn | Double  | 环境温度                                                     |
| temperatureSkin | Double  | 体表温度                                                     |
| temperatureCore | Double  | 体核温度                                                     |
|        x        | Double  | 定位数据：A0A值才有用                                        |
|        y        | Double  | 定位数据：A0A值才有用                                        |
|    sportMode    | String  | 是否开启运动模式：on、off                                    |

#### 3.1.4. C5S

message对象是**C5SData**直接调用属性处理：

|      属性       |  类型   | 含义                                                         |
| :-------------: | :-----: | ------------------------------------------------------------ |
|       mac       | String  | 终端设备mac地址                                              |
|  terminalType   | String  | 终端设备类型（B10C）                                         |
|     battery     | Integer | 电池电量                                                     |
|   gatewayMac    | String  | 蓝牙网关：蓝牙网关mac地址<br>AOA网关：AOA网关mac地址<br>Beancon：Beancon的mac地址 |
|      rssi       | Integer | 网关的rssi值                                                 |
|    localType    | String  | 定位类型，查看2.3                                            |
|   createTime    | String  | 创建时间（2022-04-08 00:00:00）                              |
|  locationTime   | String  | 数据上传的时间                                               |
|    heartRate    | Integer | 心率                                                         |
|      step       | Integer | 步数                                                         |
|     silent      | String  | 运动状态：动、静                                             |
|      wear       | String  | 是否佩戴：佩戴、未佩戴                                       |
|      sleep      | String  | 睡眠状态：浅睡、深睡、清醒                                   |
| temperatureEnvn | Double  | 环境温度                                                     |
| temperatureSkin | Double  | 体表温度                                                     |
| temperatureCore | Double  | 体核温度                                                     |
|        x        | Double  | 定位数据：A0A才有用                                          |
|        y        | Double  | 定位数据：A0A值才有用                                        |
|    systolic     | Integer | 收缩压                                                       |
|    diastolic    | Intger  | 舒张压                                                       |
|      spo2       | Integer | 血氧                                                         |
|       hrv       | Integer | hrv值                                                        |
|    sportMode    | String  | 是否开启运动模式\|：on、off                                  |

#### 3.1.5. R9

message对象是**R9Data**直接调用属性处理：

|       属性       |  类型   | 含义                                                         |
| :--------------: | :-----: | ------------------------------------------------------------ |
|       mac        | String  | 终端设备mac地址                                              |
|   terminalType   | String  | 终端设备类型（B10C）                                         |
|     battery      | Integer | 电池电量                                                     |
|    gatewayMac    | String  | 蓝牙网关：蓝牙网关mac地址<br>AOA网关：AOA网关mac地址<br>Beancon：Beancon的mac地址 |
|       rssi       | Integer | 网关的rssi值                                                 |
|    localType     | String  | 定位类型，查看2.3                                            |
|    createTime    | String  | 创建时间（2022-04-08 00:00:00）                              |
|   locationTime   | String  | 数据上传的时间                                               |
|    heartRate     | Integer | 心率                                                         |
|       step       | Integer | 步数                                                         |
|      silent      | String  | 运动状态：动、静                                             |
|       wear       | String  | 是否佩戴：佩戴、未佩戴                                       |
|      sleep       | String  | 睡眠状态：浅睡、深睡、清醒                                   |
| temperatureEnvn  | Double  | 环境温度                                                     |
| temperatureSkin  | Double  | 体表温度                                                     |
| temperatureCore  | Double  | 体核温度                                                     |
|        x         | Double  | 定位数据：A0A和UWB值才有用                                   |
|        y         | Double  | 定位数据：A0A和UWB值才有用                                   |
|     systolic     | Integer | 收缩压                                                       |
|    diastolic     | Intger  | 舒张压                                                       |
|       spo2       | Integer | 血氧                                                         |
|      charge      | Integer | 充电状态：充电中、未充电                                     |
| chargeIntoStatus | Integer | 充电器：充电器插入、充电器未插入                             |
|       sos        | String  | SOS求救：on、off                                             |
|     calorie      | Integer | 消耗的热量                                                   |
|       band       | String  | 是否断带：on（断带）、off（未断带）                          |
|     version      | String  | UWB模式下存在版本                                            |
|      panid       | String  | UWB的区域，用于定位用户所在区域                              |

### 3.2. 室外数据对接

用于对接室外的NB、CAT1、IOT的室外数据，获取用于健康体征以及相关的室外定位信息。

#### 3.2.1. 心率数据

标识为：flag == MessageFlag.HEART_RATE

message对象是**HeartOfflineData**直接调用属性处理：

|       属性       |  类型   | 含义                                                         |
| :--------------: | :-----: | ------------------------------------------------------------ |
|       mac        | String  | 终端设备mac地址                                              |
|      total       | Integer | 总数据包数（288），一天数据的总包数可通过心率数据间隔时间和上报个数计算出来，固定间隔时间5s，固定每包个数为60，总包数为86400(一天秒数)/5/60=288包数据；默认写死288，该属性主要是用于客户处理逻辑灵活使用 |
|       pack       | Integer | 当前包数，即第几包                                           |
|       date       | String  | 日期                                                         |
|    createTime    | String  | 本地服务器获取数据时间（2022-04-09）                         |
| heartOfflineData | String  | 一串心率数据值，用逗号隔开                                   |
|      count       | Integer | 上报心率个数（默认60）                                       |
|     interval     | Integer | heartOfflineData每个心率值的间隔时间（默认5s）               |

#### 3.2.2. 室外上传实时数据

标识为：flag == MessageFlag.OUTDOOR

message对象是**OutdoorData**直接调用属性处理：

|      属性       |  类型   | 含义                                                         |
| :-------------: | :-----: | ------------------------------------------------------------ |
|       mac       | String  | 终端设备mac地址                                              |
|       lat       | Double  | 纬度数据                                                     |
|       lon       | Double  | 经度数据                                                     |
|       snr       | Integer | 噪音比，不用关心的值                                         |
|   createTime    | String  | 本地服务器获取数据时间（2022-04-09）                         |
|     battery     | Integer | 电池电量                                                     |
|      step       | Integer | 步数                                                         |
| temperatureEvn  | Double  | 环境温度                                                     |
| temperatureSkin | Double  | 体表温度                                                     |
| temperatureCore | Double  | 体核温度                                                     |
|      wear       | String  | 是否佩戴：yes、no                                            |
|      spo2       | Integer | 血氧，部分终端支持                                           |
|    diastolic    | Integer | 舒张压，部分终端支持                                         |
|    systolic     | Integer | 收缩压，部分终端支持                                         |
|    pressure     | Double  | 大气压，部分终端支持                                         |
|      heart      | Integer | 心率                                                         |
|  locationTime   | String  | 终端上传数据时间                                             |
|       hrv       | String  | hrv值，部分终端支持；<br>格式："float,float,float,float,float",分别表示SDNN，TP，LF，HF，VLF |
|     calorie     | Integer | 热量，部分终端支持                                           |
|       csq       | Integer | 通信模板信号值                                               |

#### 3.2.3. 室外定位数据

标识为：flag == MessageFlag.LOCATION

message对象是**LocationData**直接调用属性处理：

|     属性     |  类型   | 含义                           |
| :----------: | :-----: | ------------------------------ |
|     mac      | String  | 终端mac地址                    |
|     lat      | Double  | 纬度                           |
|     lon      | Double  | 经度                           |
| locationTime | String  | 数据传输过来的时间即设备时间   |
|  createTime  | String  | 服务器本地时间                 |
|   accuracy   | Integer | 定位精度                       |
| locationType | String  | 定位类型：gps，wifi，cat1，iot |

#### 3.2.4. 睡眠数据

标识为：flag == MessageFlag.SLEEP

message对象是SleepData直接调用属性处理：

|    属性    |  类型   | 含义                                                         |
| :--------: | :-----: | ------------------------------------------------------------ |
|    mac     | String  | 终端设备mac地址                                              |
|   total    | Integer | 总数据包数（288）                                            |
|    pack    | Integer | 当前包数，即第几包                                           |
|    date    | String  | 日期                                                         |
| createTime | String  | 本地服务器获取数据时间（2022-04-09）                         |
|   sleep    | String  | 数据逗号隔开<br>sleepType为1时：<br>0:活动，1：浅度，2：深度 3：未监测<br>sleepType为2时：<br>0 清醒, 1 浅睡, 2 深睡, 3 快速动眼, 255 无效数据 |
| sleepType  | Integer | 睡眠数据类型：1、2                                           |
| sleepDate  | String  | 睡眠数据日期：20220411                                       |
| sleepTime  | String  | 数据逗号隔开<br>sleepType为1时：每个时间点对应每个sleep的状态<br>sleepType为2时：每个时间点对应的sleep，表示该时间点切换的睡眠状态 |
|  allSleep  | Integer | 总睡眠时长（分钟）                                           |
| deepSleep  | Integer | 深度睡眠时长（分钟）                                         |
| lightSleep | Integer | 浅度睡眠时长（分钟）                                         |
|    eye     | Integer | 快速动眼时长（分钟），仅sleepType为2的时候该值有效           |

#### 3.2.5. 事件上报

标识为：flag == MessageFlag.SYSTEM

message对象是**SystemData**直接调用属性处理：

|     属性      |       类型        | 含义                                                         |
| :-----------: | :---------------: | ------------------------------------------------------------ |
|      mac      |      String       | 终端设备mac地址                                              |
|     nbver     |      String       | nb模组版本号【开关机、低电量事件字段】                       |
|     lver      |      String       | 固件版本【开关机、低电量事件字段】                           |
|   protocol    |      String       | 协议版本标识【开关机、低电量事件字段】                       |
|     ccid      |      String       | 物联网卡号【开关机 、低电量事件字段】                        |
|     imei      |      String       | 设备IMEI号码【开关机、低电量事件】                           |
|     imsi      |      String       | 国际移动用户识别码                                           |
|     sport     |  List<SportInfo>  | 具体查看下文SportInfo类型【运动事件触发】                    |
|     heart     |      Integer      | 当前心率值【心率告警触发】                                   |
|     temp      |      Double       | 当前温度值【温度异常触发】                                   |
|     spo2      |      Integer      | 当前血氧值【血氧异常触发】                                   |
|   pressure    |      Integer      | 当前大气压值【气压异常触发】                                 |
|   diastolic   |      Integer      | 当前舒张压【血压低压异常触发】                               |
|   systolic    |      Integer      | 当前收缩压【血压高压异常触发】                               |
|     check     |      String       | 打卡类型和打卡事件，逗号隔开                                 |
|      sos      |      String       | sos告警信息【SOS事件触发】；数据格式：纬度,经度,定位方式,定位时间 |
|   spo2Test    |      String       | 手动测量血氧值（血氧值，测量时间）【手动测量血氧时触发】     |
| bPressureTest |      String       | 手动测试血压值（高压、低压、测量时间）【手动测量血压触发】   |
|  strokeTest   |      String       | 手动测量中风风险值（百分比、测量时间）【手动测试中风风险值触发】 |
|     fence     |      String       | 离开围栏范围：无内容<br>进入围栏范围：直接上报围栏序号+时间+MAC+信号，中间逗号隔开。 |
|   fallAlarm   |      String       | 跌倒时候触发的跌倒事件；数据格式：纬度,经度,定位方式,定位时间 |
|   manualGps   |      String       | 主动请求GPS触发事件；数据格式：纬度,经度,定位方式,定位时间   |
|   sportCwm    | List<Lat2LonData> | CWM运动事件触发，Lat2LonData具体看下文，该字段数据，目前传给客户是空的，需要客户从其他类型数据钟说即MessageFlag.SPORT_CWM获取组装 |
|    status     |      String       | 表示上报的事件有哪些，如心率和摔倒事件，则：heart\|fall，用"\|"隔开 |

##### 3.2.5.1. **SportInfo类**

|   属性    |  类型  | 含义                                                         |
| :-------: | :----: | ------------------------------------------------------------ |
| startTime | String | 运动开始时间                                                 |
|  endTime  | String | 运动结束时间                                                 |
|  calorie  |  Long  | 消耗的卡路里                                                 |
|   step    |  Long  | 走的步数                                                     |
| sportType | String | 运动类型：无运动类型、徒步、跑步、爬山、球类运动、力量训练、有氧训练、自定义运动 |

##### 3.2.5.2.Lat2LonData类

|     属性     |  类型   | 含义             |
| :----------: | :-----: | ---------------- |
| locationTime | String  | 该定位运动的时间 |
|     lat      | Double  | 坐标纬度         |
|     lon      | Double  | 坐标经度         |
|   accuray    | Integer | 定位经度         |
| locationType | String  | 定位方式         |

#### 3.2.6.CWM运动事件

标识为：flag == MessageFlag.SPORT_CWM

message对象是**SportCwm**直接调用属性处理：

| 属性 |       类型        | 含义                                                      |
| :--: | :---------------: | --------------------------------------------------------- |
| mac  |      String       | 终端设备mac地址                                           |
| data | List<Lat2LonData> | CWM运动事件触发，Lat2LonData具体查看3.2.5.2.Lat2LonData类 |

## 4. TCP协议对接

该对接方式用于客户是非java开发的项目，或者客户不像打破原有的业务系统架构，则采用该对接方式，建立一个tcp服务，客户作为客户端，我方是服务端，以json的方式传输数据，采集获取数据后，客户把数据用于自己的业务分析使用

### 4.1. 室内数据对接

即使用蓝牙网关、AOA网关、UWB网关以及Beacon获取到的生理健康数据等信息

格式形如：

```json
{
	"type": "indoor",
	"data": {
		"battery": 32,
		"createTime": "2022-04-12 09:18:28",
		"diastolic": 0,
		"fall": "正常",
		"gatewayMac": "8cd495001269",
		"heartRate": 0,
		"hrv": 0,
		"localType": "bluetooth",
		"locationTime": "2022-03-29 01:10:15",
		"mac": "d5a414249659",
		"restingHeartRate": 65,
		"rssi": 34,
		"silent": "静",
		"sleep": "清醒",
		"sos": "off",
		"spo2": 0,
		"sportMode": "off",
		"step": 58,
		"systolic": 0,
		"temperatureCore": 35.1,
		"temperatureEnvn": 0.0,
		"temperatureSkin": 0.0,
		"terminalType": "B10C",
		"wear": "佩戴",
		"x": 0.0,
		"y": 0.0
	}
}
```

**注：**各个属性可参考JAR包对应类的数据，即你使用什么设备采集数据，就查看对应类，然后查看对应属性进行解析使用，这里就不再累述了

### 4.2. 室外数据对接

用于对接室外的NB、CAT1、IOT的室外数据，获取用于健康体征以及相关的室外定位信息。

1. 室外定位数据，如果没有lat和lon则表示该地段没有gps信号

	```json
	{
		"type": "outdoor",
		"data": {
			"battery": 78,
			"calorie": 784,
			"createTime": "2022-04-10 15:49:08",
			"csq": 15,
			"diastolic": 0,
			"heart": 0,
			"hrv": "0.000,0.000,0.000,0.000,0.000",
			"mac": "da3d83dbc873",
			"spo2": 0,
			"step": 0,
			"systolic": 0,
			"temperatureCore": 0.0,
			"temperatureEvn": 26.28,
			"temperatureSkin": 0.0,
			"wear": "No"
		}
	}
	```

2. 离线心率数据

	```json
	{
		"type": "heartRate",
		"data": {
			"count": 60,
			"createTime": "2022-04-10 15:49:11",
			"date": "20220410",
			"heartOfflineData": "96,97,95,96,97,95,94,93,96,97,95,98,112,107,102,98,98,99,102,101,96,96,94,92,93,91,93,93,93,91,92,92,92,93,93,90,92,88,85,86,89,90,92,92,93,95,92,91,96,96,95,89,87,86,90,92,92,92,90,91",
			"interval": 5,
			"mac": "ce28f7bb70c6",
			"pack": 189,
			"total": 288
		}
	}
	```

3. 室外定位数据

	```json
	{
		"type": "location",
		"data": {
			"accuracy": 9730,
			"createTime": "2022-04-10 16:03:03",
			"lat": 32.163272,
			"locationTime": "2022-04-10 19:02:57",
			"locationType": "cat1",
			"lon": 34.817759,
			"mac": "c42612684dc6"
		}
	}
	```

4. 睡眠数据

	```json
	{
		"type": "sleep",
		"data": {
			"allSleep": 286,
			"createTime": "2022-04-10 16:17:37",
			"date": "",
			"deepSleep": 44,
			"eye": 31,
			"lightSleep": 208,
			"mac": "c42612684dc6",
			"pack": 1,
			"sleepDate": "1,0,1,3,1,2,1,2,1,0",
			"sleepTime": "00:00,01:17,01:20,01:33,02:04,03:07,03:27,04:06,04:30,04:46",
			"sleepType": 2,
			"total": 2
		}
	}
	```

5. 事件上报数据，有则上报，没有的话，该字段为null，不显示

	```json
	{
		"type": "system",
		"data": {
			"createTime": "2022-04-10 16:47:51",
			"fallAlarm": "32.1303459,34.7958939,cat1",
			"mac": "c42612684dc6",
			"status": "fall"
		}
	}
	```

