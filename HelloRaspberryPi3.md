# Raspberry Pi 3 環境構築メモ
### 2017年度 後期 情報学序論 インテリジェントシステム研究室
### 担当：馬場博巳
### 資料作成: 2017.10.03
### 資料改定: 2017.10.10

---

## 1. Raspberry Pi を使うための準備
### 1-1. OSのダウンロード
  + RASPBIAN STRETCH WITH DESKTOP 2017-09-07
  + original: https://www.raspberrypi.org/downloads/
  + mirror: ftp.jaist.ac.jp/pub/raspberrypi/raspbian/images/

### 1-2. microSD の準備（Macの場合）
  1. SDをFATでフォーマット（ディスクユーティリティを使用）
  2. $ diskutil list (IDENTIFIERを調査．例：disk3)
  3. $ diskutil unmountDisk /dev/disk3
  4. $ sudo dd bs=1m if=_filename_ of=/dev/rdisk3
  5. $ diskutil eject /dev/disk3

### 1-3. update OS & Famaware
  + $ sudo apt-get update           // パッケージリスト更新
  + $ sudo apt-get upgrade -y       // インストール済パッケージ更新
  + $ sudo apt-get dist-upgrade -y  // カーネルアップグレード
  + $ sudo rpi-update               // ファームウェア更新
  + $ sudo reboot                   // 再起動
  + **【注意!!】 apt-getで新しいライブラリをインストールする場合，失敗を避けるために，上記の作業を行った方がよい．**

### 1-4. Localization

  + RaspberryPiの環境設定メニューを開く
  + 最初の画面で，起動時の外観や動作形態などを設定する．
  + ネットワークのコネクター部分に貼ってあるシールを参考にしてHostnameを入力しておく．
  + 起動時のエラーチェックのために，SplashScreen は Disabled
  + 画面周囲の黒枠はいらないので，Underscan は Disabled

<img src="img/Screen/00_Config.jpg" width=45%>
<img src="img/Screen/01_Config_System.jpg" width=45%>

  + 日本語が使えるように，以下の環境を設定する．
  + Locale : ja(Japanese), JP(Japan), UTF-8

<img src="img/Screen/02_Config_Localisation.jpg" width=45%>
<img src="img/Screen/03_Config_Localisation_Locale.jpg" width=45%>

  + Timezone : Asia, Tokyo
  + Keyboard : Japan, Japanese
  + Wi-Fi : JP(Japan)

<img src="img/Screen/04_Config_Localisation_TimeZone.jpg" width=25%>
<img src="img/Screen/05_Config_Localisation_KeyboardLayout.jpg" width=35%>
<img src="img/Screen/06_Config_Localisation_WiFiCountry.jpg" width=30%>

  + 再起動(Shutdownのメニューのなかで，ShutdownかRebootかLogoutを選ぶ)

<img src="img/Screen/07_Shutdown.jpg">
<img src="img/Screen/08_Shutdown_Options.jpg">



### 1-5. サウンド設定
  + 以下の設定は，必要に応じて行う．
  + 自動判別： $ sudo amixer cset numid=1 0
  + ライン出力： $ sudo amixer cset numid=1 1
  + HDMI出力： $ sudo amixer cset numid=1 2
  + 音量調節： $ sudo amixer cset numid=1 90%

### 1-6. USB port 電源供給量UP
  + こちらの設定も必要に応じて行う．
  + /boot/config.txt の編集（以下を追加）
    - max_usb_current=1
  + 再起動

## 2.日本語環境の構築
### 2-1. 日本語フォントのインストール
  + 現在のRaspberry Pi は，日本語を使用する設定にすれば，日本語が表示される．（日本語用のフォントがインストール済み．）
  + しかし，標準の日本語フォントはあまり美しくないので，好みでフォントをインストールする人もいる．
  + 以下の例は，長いので改行して記述しているが，実際は１行ですべてを入力する．（フォント1種類ずつ「sudo apt-get install ...」とやってもいいけど...面倒なら１行で全部書く.)
  + $ sudo apt-get install fonts-vlgothic
    + ttf-kochi-gothic
    + fonts-takao
    + fonts-ipafont
    + xfonts-intl-japanese
    + xfonts-intl-japanese-big
    + xfonts-kaname
  + Googleのフォントが，線が細いものの美しいという評判だが，「apt-get install」では入れられないので，次回以降で紹介する．

### 2-2. 日本語入力
  + 日本語は表示されても，日本語の入力ソフトが無い．それは別途インストールが必要．
  + 日本語入力ソフトもいろいろあるが，今回は fcitx-mozc をインストールする．
  + $ sudo apt-get install fcitx-mozc -y
  + このソフトをインストールすると，標準のキーボード入力が「英語(UK)」と設定されており，最初に行った日本語キーボードの設定が無効になってしまう．
  + mozcの設定は別途変更する必要がある．
  + 画面右上のmozcのアイコンを右クリックして設定画面を開く
  + 「キーボード - 日本語」を追加後，優先順位を一番上にして，「キーボード - 英語(UK)」を削除する．

<img src="img/Screen/10_Mozc_Config.jpg" width=25%>
<img src="img/Screen/11_Mozc_Keyboard.jpg" width=70%>


## 3.Arduino IDE のインストール
  + この講義では，Raspberry Pi 上で Arduino のスケッチを作成するので，ArduinoIDEをインストールする．
  + $ sudo apt-get install arduino -y
  + インストールコマンド実行後は，メニューの中に Arduino IDE が追加される．
 
 <img src="img/Screen/09_ArduinoIDE.jpg" width=50%>
 
  + １年生後期の科学的問題解決法で学習したので，ArduinoIDEの基本的な使い方の解説は省略する．
  + Arduino のピン配置は以下の通り．
 
 <img src="img/ArduinoPin/ArduinoUNO.jpg" width=95%>
  

## 4. PCA9632DP1(I2C 4ch LEDドライバ基板)の実験
 + 簡単な配線とArduinoIDEの利用方法の学習教材として，PCA9632DP1 （I2C 4ch LEDドライバ基板）を利用する．
 + 以下の左写真は基板の実際の様子，右はピン配置である．

<img src="img/PCA9632DP1/PCA9632DP1A.jpg" width=40%>
<img src="img/PCA9632DP1/PCA9632DP1_Pin.jpg" width=50%>

+ この基板は，通常のLED点灯実験として使えるし，基盤上のPCA9632DP1を通してI2CでLEDの状態をコントロールすることもできる．
+ どちらの方法で使う場合も，VDDに5V，GNDにGNDを接続する.
+ LED0〜3は，基板上に抵抗が接続されて電源に接続されているので，入力端子に「Low」を入力すれば電流が流れて LED が点灯し，「HIGH」を入力すれば電流が流れずに LED が消灯する．

### 4-1. 単純なLEDの点灯・消灯実験

【練習01-1】以下の表のように PCA9632DP1基板とArduinoを接続し，LED0〜LED3を点灯・消灯させてみよう．

《ピンの接続》

| ArduinoUNO | PCA9632DP1基板 |
|:--:|:--:|
| 5V | VDD |
| GND | GND |
| 13 | LED0 |
| 12 | LED1 |
| 11 | LED2 |
| 10 | LED3 |

《LED点灯・消灯実験のプログラム例》

```
#define LED0 13
#define LED1 12
#define LED2 11
#define LED3 10

void setup() {
  pinMode(LED0, OUTPUT);	// ピンモードの設定
  pinMode(LED1, OUTPUT);
  pinMode(LED2, OUTPUT);
  pinMode(LED3, OUTPUT);
  
  digitalWrite(LED0, HIGH);	// 初期状態（全部消灯）
  digitalWrite(LED1, HIGH);
  digitalWrite(LED2, HIGH);
  digitalWrite(LED3, HIGH);
}

void loop() {
  digitalWrite(LED0, LOW);		// １個ずつ点灯
  delay(300);
  digitalWrite(LED1, LOW);
  delay(300);  
  digitalWrite(LED2, LOW);
  delay(300);
  digitalWrite(LED3, LOW);
  delay(300);
  
  digitalWrite(LED0, HIGH;		// １個ずつ消灯
  delay(300);
  digitalWrite(LED1, HIGH);
  delay(300);  
  digitalWrite(LED2, HIGH);
  delay(300);
  digitalWrite(LED3, HIGH);
  delay(300);
}
```

【練習01-2】上記のプログラムを参考に，自分で点灯・消灯のパターンを考えて実行させてみよう．


### 4-2. I2Cを利用したLEDの点灯・消灯実験

 + I2C とは，Inter-Integrated Circut の略称で，本来は，「I<sup>2</sup>C」と書き，「アイ・スクエアド・シー」と読むが，日本では「IIC」や「I2C」と書いたり，「アイ・ツー・シー」と読んだりする．
 + この講義では，I2Cと書き，「アイ・ツー・シー」と読むことにする．
 + I2Cは，マスターとスレーブがSCL(Sirial CLock)とSDA(Sirial DAta)の2本の線で連結されている．
 + 通信の基本的な流れは，マスターからスレーブに命令を与えて，スイレーブからその反応が返ってくるというものだが，すべてのスレーブが同一の線で連結されているので，スレーブにはアドレスが付与されている．
 + 今回使用するPCA9632DP1は7bitのアドレスで，「1100010」の固定となっている．

<img src="img/I2C/I2C.jpg" width=45%>
<img src="img/PCA9632DP1/I2C_7bit_Address.jpg" width=45%>

【練習01-3】基板販売メーカーHPのサンプルプログラムで，I2C通信の動作実験をしてみよう．

《ピンの接続》

| ArduinoUNO | PCA9632DP1基板 |
|:--:|:--:|
| 5V | VDD |
| GND | GND |
| SCL | SCL |
| SDA | SDA |


《サンプルプログラム1》

```
#include <Wire.h> //I2C通信用

#define PCA9632_DEV_ID 0x62 //全体制御用のI2Cアドレス
void setup()
{
  pinMode(13, OUTPUT); //debug用
  Wire.begin(); //Arduinoがマスター
  /*起動時の処理*/
  Wire.beginTransmission(PCA9632_DEV_ID);//送信先のアドレスを指定して通信開始
  Wire.write(0x00); //MODE制御用のレジスタ
  Wire.write(B10000001); //デフォルトがSLEEP:ENABLEでB100[1]0001になってるので注意
  Wire.endTransmission(); //通信終了
  delay(500);
}

void loop()
{
  /*点灯*/
  digitalWrite(13, HIGH);        //debug用
  Wire.beginTransmission(PCA9632_DEV_ID);
  Wire.write(0x08);             //LED制御用のレジスタアドレス
  Wire.write(B01010101);        //2bit×4個分の点灯制御//00=消灯//01=強制点灯//10=個別PWM制御もON//11=全体PWM制御もON
  Wire.endTransmission();
  delay(500);

  /*消灯*/
  digitalWrite(13, LOW);
  Wire.beginTransmission(PCA9632_DEV_ID);
  Wire.write(0x08);
  Wire.write(0B00000000);
  Wire.endTransmission();
  delay(500);
}
```

《サンプルプログラム2》


```
#include <Wire.h> //I2C通信用

#define PCA9632_DEV_ID 0x62 //全体制御用のI2Cアドレス

void setup()
{
  pinMode(13, OUTPUT); //debug用
  Wire.begin(); //Arduinoがマスター
  /*起動時の処理*/
  sendI2C_single(0x00, B10000001); //MODE1//デフォルトがSLEEP:ENABLEでB100[1]0001になってるので注意
  sendI2C_single(0x01, B00100001); //MODE2//全体PWM制御を点滅モードに設定 DMBLNK=1
  sendI2C_single(0x02, B00001111); //PWM0//暗め(データシートp12式1)
  sendI2C_single(0x03, B11111111); //PWM1//明るめ(p12式1)
  sendI2C_single(0x04, B00001111); //PWM2//暗め(p12式3)
  sendI2C_single(0x05, B11111111); //PWM3//明るめ(p12式3)
  sendI2C_single(0x06, B01111111); //GRPPWM//全体PWMのduty比を1:1に(p13式5)
  sendI2C_single(0x07, B00010111); //GRPFREQ//全体PWMの周期を約1secに(p14式7)
  sendI2C_single(0x08, B11111010); //LEDOUT//3,2,1,0//00=消灯//01=強制点灯//10=個別PWM制御もON//11=全体PWM制御もON
  delay(1);
}

void loop()
{
  /**************  LED3,2は10で個別PWMにより調光されている  LED1,0は11で個別PWMによって調光された上で、点滅モードの全体PWMで点滅している  **************/
}

/*データ送信用簡易関数*/
void sendI2C_single(byte REG, byte DATA)
{
  Wire.beginTransmission(PCA9632_DEV_ID);//送信先のアドレスを指定して書き込み用に通信開始
  Wire.write(REG);//データを書き込むレジスタアドレスを送信
  Wire.write(DATA);//データを送信
  Wire.endTransmission(); //通信終了
}
```

