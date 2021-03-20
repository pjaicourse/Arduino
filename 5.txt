#include <Arduino.h>
#include <U8g2lib.h>

#include <SPI.h>
#include <Wire.h>
#include <MFRC522.h>

#define RST_PIN         9          
#define SS_PIN          10 //RC522卡上的SDA

MFRC522 mfrc522;   // 建立MFRC522實體

char *reference;

byte uid[]={0x49, 0xE5, 0xA0, 0xC1};  //這是我們指定的卡片UID，可由讀取UID的程式取得特定卡片的UID，再修改這行

//這行是NodeOLED用的
//U8G2_SSD1306_128X64_NONAME_1_SW_I2C u8g2(U8G2_R0, /* clock=*/ D2, /* data=*/ D1, /* reset=*/ U8X8_PIN_NONE); 
U8G2_SSD1306_128X64_NONAME_1_HW_I2C u8g2(U8G2_R0, /* reset=*/ U8X8_PIN_NONE);  //Arduino Uno+0.96吋OLED用這行
//U8G2_SH1106_128X64_NONAME_1_HW_I2C u8g2(U8G2_R0, /* reset=*/ U8X8_PIN_NONE); //如果用1.3吋OLED用這行

//OLED上方標題，用圖形表現
static const unsigned char PROGMEM  title[256] = { 
0XFF,0XFF,0XFF,0XFF,0XFF,0XFF,0XFF,0XFF,0XBF,0XEF,0XFE,0XFF,0XFF,0XFF,0XFF,0XFF,
0XFF,0XFF,0XFF,0XFF,0XFF,0XFF,0XFD,0XE7,0XBF,0XEF,0XFA,0XFF,0XFF,0XFF,0XFD,0XFE,
0XE3,0X3F,0XF8,0XC1,0X1F,0XFE,0XFB,0XEE,0X7F,0X83,0XFA,0XFF,0XE3,0XFF,0X7E,0XFE,
0XDB,0XBF,0XFF,0XF7,0XDF,0XFD,0XE0,0X82,0XFF,0XFF,0XF6,0X1F,0XFC,0XFF,0X12,0XE0,
0XBB,0XBF,0XFF,0XF7,0XDF,0XFB,0XFF,0XFE,0X0F,0XB6,0XFE,0XE1,0XFE,0X7F,0XBB,0XFF,
0XBB,0XBF,0XFF,0XF7,0XDF,0XF7,0XA5,0XB6,0XFF,0X03,0XF0,0X3F,0XFF,0XBF,0XBB,0XFB,
0XBB,0XBF,0XFF,0XF7,0XDF,0XF7,0XB5,0XD6,0X1F,0XFE,0XFE,0XDF,0XF3,0X1F,0XDC,0XFB,
0XBB,0XBF,0XFF,0XF7,0XDF,0XF7,0XB7,0XDE,0XFF,0X83,0XF6,0X07,0XFC,0XFF,0XDE,0XF3,
0XDB,0X3F,0XF8,0XF7,0XDF,0XF7,0XA0,0X82,0X1F,0XBA,0XFA,0X7F,0XE6,0X7F,0X0B,0XF4,
0XE3,0XBF,0XFF,0XF7,0XDF,0XF7,0XFB,0XEE,0XFF,0XBB,0XFA,0X9F,0XDF,0XBF,0XB9,0XE5,
0XDB,0XBF,0XFF,0XF7,0XDF,0XF7,0XFB,0XEE,0X1F,0X82,0XFD,0X03,0X80,0X1F,0XBE,0XFD,
0XDB,0XBF,0XFF,0XF7,0XDF,0XF7,0XE0,0X82,0XDF,0XBA,0XFD,0XFB,0XBE,0XFF,0XB3,0XFD,
0XDB,0XBF,0XFF,0XF7,0XDF,0XFB,0XFB,0XEE,0XDF,0XBA,0XFE,0XCF,0XF6,0XBF,0XB6,0XFD,
0XBB,0XBF,0XFF,0XF7,0XDF,0XFD,0X7B,0XEF,0XDF,0X82,0XF4,0XE7,0XCE,0XBF,0XD5,0XED,
0XBB,0XBF,0XFF,0XC1,0X1F,0XFE,0X7D,0XEF,0X1F,0X7A,0XF5,0XF9,0XBE,0XBF,0XDD,0XED,
0XFF,0XFF,0XFF,0XFF,0XFF,0XFF,0XBD,0XEF,0XDF,0XBF,0XFB,0X1F,0XFE,0XDF,0XE7,0XF3,
};

void setup()
{
  
  
  
  Serial.begin(9600);   
  
  SPI.begin();        // 初始化SPI介面
  
  u8g2.begin();
  
  mfrc522.PCD_Init(SS_PIN, RST_PIN); // 初始化MFRC522卡
  Serial.print(F("Reader "));
  Serial.print(F(": "));
  mfrc522.PCD_DumpVersionToSerial(); // 顯示讀卡設備的版本
  
  //測試RFID設備是否正常
  byte v = mfrc522.PCD_ReadRegister(mfrc522.VersionReg);
  if((v == 0x00) || (v == 0xFF)){
    reference = "Reader Error";
    showText1();
  }else{
    reference = "Reader Ready";
    showText1();
    delay(1500);
    showText2();
  }
}

void showText1(){
  u8g2.setFont(u8g2_font_samim_16_t_all); //字型
  u8g2.firstPage();
  do {
      u8g2.drawXBMP(0,0, 128, 16, title);
      u8g2.drawStr(0,40,"Reader Testing...");
      
    u8g2.drawStr(10,60,reference);
  } while ( u8g2.nextPage() );
}

void showText2(){
  char *reference;
  u8g2.setFont(u8g2_font_samim_16_t_all); //字型
  u8g2.firstPage();
  do {
      u8g2.drawXBMP(0,0, 128, 16, title);
      u8g2.drawStr(10,40,"Place RFID...");
  } while ( u8g2.nextPage() );
}

void loop() {
  //Serial.print("reading...");
  // 檢查是不是偵測到新的卡
  if (mfrc522.PICC_IsNewCardPresent() && mfrc522.PICC_ReadCardSerial()) {
      // 顯示卡片的UID
      Serial.print(F("Card UID:"));
      dump_byte_array(mfrc522.uid.uidByte, mfrc522.uid.size); // 顯示卡片的UID
      
      for(int i = 0; i < mfrc522.uid.size; i++)
      {
        u8g2.firstPage();
        do {
          u8g2.drawXBMP(0,0, 128, 16, title);
          u8g2.setCursor(10,40);
          u8g2.print("RFID UID:");
          for (byte i = 0; i < mfrc522.uid.size; i++) {

            if(mfrc522.uid.uidByte[i] < 0x10){
              u8g2.setCursor(10+i*25,60);
              u8g2.print("0");
              u8g2.setCursor(10+i*25+10,60);
              u8g2.print(mfrc522.uid.uidByte[i], HEX);
            }else{
              u8g2.setCursor(10+i*25,60);
              u8g2.print(mfrc522.uid.uidByte[i], HEX);
            }
            
            
          }
        } while ( u8g2.nextPage() );
      }
      
      
      Serial.println();
      Serial.print(F("PICC type: "));
      MFRC522::PICC_Type piccType = mfrc522.PICC_GetType(mfrc522.uid.sak);
      Serial.println(mfrc522.PICC_GetTypeName(piccType));  //顯示卡片的類型
      
      //把取得的UID，拿來比對我們指定好的UID
      bool they_match = true; // 初始值是假設為真 
      for ( int i = 0; i < 4; i++ ) { // 卡片UID為4段，分別做比對
        if ( uid[i] != mfrc522.uid.uidByte[i] ) { 
          they_match = false; // 如果任何一個比對不正確，they_match就為false，然後就結束比對
          break; 
        }
      }
      Serial.println(they_match);
      
      //在OLED顯示比對的結果
      u8g2.firstPage();
      do {
          u8g2.drawXBMP(0,0, 128, 16, title);
          if(they_match){
            u8g2.drawStr(10,40,"Access Granted");
          }else{
            u8g2.drawStr(10,40,"Access Denied");
          }
          
      } while ( u8g2.nextPage() );
      delay(1500);
      u8g2.firstPage();
      do {
          u8g2.drawXBMP(0,0, 128, 16, title);
          u8g2.drawStr(10,40,"Place RFID...");
      } while ( u8g2.nextPage() );
      mfrc522.PICC_HaltA();  // 卡片進入停止模式
    }
}

/**
 * 這個副程式把讀取到的UID，用16進位顯示出來
 */
void dump_byte_array(byte *buffer, byte bufferSize) {
  for (byte i = 0; i < bufferSize; i++) {
    Serial.print(buffer[i] < 0x10 ? " 0" : " ");
    Serial.print(buffer[i], HEX);
  }
}