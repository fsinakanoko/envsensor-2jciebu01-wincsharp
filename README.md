
# envsensor-2jciebu01-wincsharp

## [1] 概要
OMRON製環境センサ(2JCIE-BU01)のデータをWindows C#.netで受信するWindowsフォームプログラムです。  
2JCIE-BU01を使用したシステムを構築する際の参考資料(サンプルプログラム)としてお使いください。

![全体.PNG](/img/全体.PNG)

## [2]使用方法
### (1) 使用COM番号の確認  
2JCIE-BU01とWindowsの通信時に使用するCOM番号の確認手順を記します。  
COM番号は、本アプリを使用する際に必要になります。  

1. 2JCIE-BU01をWindowsPCに指して、デバイスマネージャを開いてください。  
1. 2JCIE-BU01で使用しているCOM番号を確認してください。  
参考) デバイスマネージャ画面  
![com確認.PNG](/img/com確認.PNG)

### (2) 起動方法  

1. usb_es_winform.exe をクリックしてください。

### (3) 操作説明

![プログラム画面.PNG](/img/プログラム画面.PNG)

## (参考) 通信仕様解説

### (1) 概要
本プログラムはオムロンが公開している2JCIE-BU01の通信仕様を元にC#で実装したサンプルプログラムです。  
オムロン社環境センサを用いた独自プログラムを作成したいユーザが、実装時に参照するために作成されました。  
ただ、2JCIE-BU01の公開されている仕様は量が多いため、ユーザが本プログラムのソースコードが仕様書のどこを元に実装したものか理解するのに、時間がかかるという課題が発生すると予想されます。  
本項では理解するための時間を少しでも短縮するために、通信仕様の解説を行います。  
尚、オムロンが公開している通信仕様は以下のサイトからダウンロードできますので、独自プログラムの実装時は確認願います。  
https://www.fa.omron.co.jp/data_pdf/mnu/cdsc-016a-web1_2jcie-bu01.pdf?id=3724



### (2) 通信仕様全体像
環境センサからデータを受信するために、まずどの情報が欲しいのかを示すデータ(Commandデータ)を2JCIE-BU01に送信します。  
2JCIE-BU01は受け取ったコマンドに対応するデータを送信(Responseデータ)します。

参考 : 4.2. Communication procedure  
![4_2_Communication_procedure.PNG](/img/4_2_Communication_procedure.PNG)



### (3) Command/Responseデータ共通仕様について
Command/Responseデータはソースコード上、バイナリ型の配列として扱うことができます。  
配列は番地ごとに"Header","Length","Payload","CRC-16"の値が入っており、そのうち"Payload"以外の3つの仕様はCommand/Response共通となっています。  
共通仕様の内容については、以下を参照ください。


参考 : 4.3.1 Common frame format、4.3.2 CRC-16 calculation  
![4_3_1_Common_frame_format.PNG](/img/4_3_1_Common_frame_format.PNG)  
![4_3_2_CRC-16_calculation.PNG](/img/4_3_2_CRC-16_calculation.PNG)

### (3) Commandデータ仕様について
**1. CommandデータのPayload部の仕様は以下の通りです。**

参考 : 4.3.3 Payload frame format [Command from Host-Controller]  
![4_3_3_Payload_frame_format.PNG](/img/4_3_3_Payload_frame_format.PNG)

**2. 役割と実データ例は以下の通りです。**

|番地|役割|実データ例|説明|
|:---|:---|:---|:---|
|0|Header|0x52|固定|
|1|Header|0x42|固定|
|2|Length|0x05|PayloadからCRCまでのデータ長の1バイト目。(Payload(3) + CRC(2) = 0x0005)|
|3|Length|0x00|PayloadからCRCまでのデータ長の2バイト目。(Payload(3) + CRC(2) = 0x0005)|
|4|Payload|0x01|Read,Writeを指定。(0x01 : Read 0x02 : Write)|
|5|Payload|0x12|実行する内容に応じたAddressの1バイト目。(0x5012 :Latest sensing data)|
|6|Payload|0x50|実行する内容に応じたAddressの2バイト目。(0x5012 :Latest sensing data)|
|7|CRC-16|0xbb|計算したCRC値の1バイト目。|
|8|CRC-16|0xf6|計算したCRC値の２バイト目。|

** 3. 実際のサンプルプログラムの送信処理は以下の通りです。**  
```csharp
byte[] data = new byte[] { 0x52, 0x42, 0x05, 0x00, 0x01, 0x12, 0x50 };
List<byte> param = new List<byte>();
param.AddRange(data);
// CRC計算
ushort num = Calc_crc(data, data.Length);
byte numH = (byte)(num >> 8);
byte numL = (byte)(num & 0x00FF);
param.Add(numL);
param.Add(numH);
// データ送信
serialPort1.Write(param.ToArray(), 0, param.ToArray().Length);
```

### (4) Responseデータ仕様について

**1. ResponseデータのPayload部の仕様は以下の通りです。**

参考 : 4.3.4 Payload frame format [Normal Response from 2JCIE-BU01]  
![4_3_4_Payload_frame_format.PNG](/img/4_3_4_Payload_frame_format.PNG)

**2. 役割と実データ例は以下の通りです。**  

|番地|役割|実データ例|説明|
|:---|:---|:---|:---|
|0|Header|0x52|固定|
|1|Header|0x42|固定|
|2|Length|0x16|PayloadからCRCまでのデータ長の1バイト目。(Payload(20) + CRC(2) = 0x0016)|
|3|Length|0x0|PayloadからCRCまでのデータ長の2バイト目。(Payload(20) + CRC(2) = 0x0016)|
|4|Payload|0x1|Read,Writeを指定。(0x01 : Read 0x02 : Write)|
|5|Payload|0x12|実行する内容に応じたAddressの1バイト目。(0x5012 :Latest sensing data)|
|6|Payload|0x50|実行する内容に応じたAddressの2バイト目。(0x5012 :Latest sensing data)|
|7|Payload|0x91|Sequence number|
|8|Payload|0xEB|Temperatureの1バイト目|
|9|Payload|0x9|Temperatureの2バイト目|
|10|Payload|0x6D|Relative humidityの1バイト目|
|11|Payload|0x12|Relative humidityの2バイト目|
|12|Payload|0xAF|Ambient lightの1バイト目|
|13|Payload|0x1|Ambient lightの2バイト目|
|14|Payload|0xB3|Barometric pressureの1バイト目|
|15|Payload|0x95|Barometric pressureの2バイト目|
|16|Payload|0xF|Barometric pressureの3バイト目|
|17|Payload|0x0|Barometric pressureの4バイト目|
|18|Payload|0x4B|Sound noiseの1バイト目|
|19|Payload|0x15|Sound noiseの2バイト目|
|20|Payload|0x2B|eTVOCの1バイト目|
|21|Payload|0x0|eTVOCの2バイト目|
|22|Payload|0xAB|eCO2の1バイト目|
|23|Payload|0x2|eCO2の2バイト目|
|24|CRC-16|0x1A|計算したCRC値の1バイト目。|
|25|CRC-16|0x38|計算したCRC値の２バイト目。|

参考 : 4.5.2 Latest sensing data (Address: 0x5012)  
![4_5_2_Latest_sensing_data.PNG](/img/4_5_2_Latest_sensing_data.PNG)  

**3. 取得したデータの単位は以下の通りです。**

参考 : 5.1. Output range  
![5_1_Output_range.PNG](/img/5_1_Output_range.PNG)  


**4.実際のサンプルプログラムの受信及び単位変換処理は以下の通りです。**  


```csharp
List<byte> received = new List<byte>();
while (serialPort1.BytesToRead > 0)
{
    // 1 バイト受信してバッファに格納
    received.Add((byte)serialPort1.ReadByte());
}

string data = "";

//Short Data選択時のデータ出力
//温度(℃)
Double temperature = Convert.ToDouble(Int64.Parse(Convert.ToString(received[9], 16) + Convert.ToString(received[8], 16), System.Globalization.NumberStyles.HexNumber)) / 100;
//相対湿度(％RH)
Double relativeHumidity = Convert.ToDouble(Int64.Parse(Convert.ToString(received[11], 16) + Convert.ToString(received[10], 16), System.Globalization.NumberStyles.HexNumber)) / 100;
//照度(lx)
Double ambientLight = Convert.ToDouble(Int64.Parse(Convert.ToString(received[13], 16) + Convert.ToString(received[12], 16), System.Globalization.NumberStyles.HexNumber));
//気圧(hPa)
Double pressure = Convert.ToDouble(Int64.Parse(Convert.ToString(received[17], 16) + Convert.ToString(received[16], 16) + Convert.ToString(received[15], 16) + Convert.ToString(received[14], 16), System.Globalization.NumberStyles.HexNumber)) / 1000;
//騒音(dB)
Double soundNoise = Convert.ToDouble(Int64.Parse(Convert.ToString(received[19], 16) + Convert.ToString(received[18], 16), System.Globalization.NumberStyles.HexNumber)) / 100;
//室内環境下における総揮発性有機化合物濃度(ppb)
Double eTVOC = Convert.ToDouble(Int64.Parse(Convert.ToString(received[21], 16) + Convert.ToString(received[20], 16), System.Globalization.NumberStyles.HexNumber));
//室内環境下におけるCO2濃度(ppm)
Double eCO2 = Convert.ToDouble(Int64.Parse(Convert.ToString(received[23], 16) + Convert.ToString(received[22], 16), System.Globalization.NumberStyles.HexNumber));

```

以上
