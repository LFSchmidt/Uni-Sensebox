
# Sensebox mit Helligkeits-, UV-Licht-, Temperatur- und Feuchtigkeitssensoren

## Ziele
Dieses Projekt misst Helligkeit, UV-Licht, Temperatur und Luftfeuchtigkeit. Mit der Helligkeitsmessung kann nachts die Lichtverschmutzung gemessen werden und tagsüber die Sonneneinstrahlung an der Stelle wo die Sensebox steht. UV-Licht und Temperatur werden zwar auch von Ämtern gemessen, aber der Großteil der Daten wird interpoliert, da nicht genug Messstationen vorhanden sind. Insofern ist das Messen dieser Faktoren interessant, um lokale Varianzen zu messen. Luftfeuchtigkeit ist interessant, um gewisse Wetterphänomene wie z.B. Glatteis vorauszusagen, da diese nur unter bestimmten Bedingungen auftreten.

## Materialien (aus der senseBox:edu)
* Genuino UNO Board 
* Ethernet Shield
* Steckplatine
* 16 Kabel
* Ligth sensor TSL45315
* UV-Light-sensor VEML6070
* Temperature and Humidity sensor HDC 1008

## Setup Beschreibung

### Hardwarekonfiguration

Wir haben den Ethernet Shield auf das Genuino UNO Board gesteckt, die Sensoren mit der Steckplatine und die Steckplatine mit dem Ethernet Shield verbunden wie im Bild:




### Softwaresketch

Zu der Erstellung der sketch haben wir die Wire und die HDC100X library verwendet. Die Wire library sollte standartmässig bei Arduino enthalten sein und die HDC100X ist unter: https://github.com/RFgermany/HDC100X_Arduino_Library  zu finden.

```
#include <Wire.h>
#include <HDC100X.h> 
```

Folgender code sollte vor dem void setup() eingefügt werden und ist dafür verantwortlich die Variablen und Konstanten für die Nutzung der Sensoren zu deklarieren.

```
// Defines & variables for light sensor
#define I2C_ADDR        (0x29)
#define REG_CONTROL     0x00
#define REG_CONFIG      0x01
#define REG_DATALOW     0x04
#define REG_DATAHIGH    0x05
#define REG_ID          0x0A
float light;

// Defines & variables for UV sensor
#define I2C_ADDR2 0x38
#define IT_1_2 0x0 //1/2T
#define IT_1   0x1 //1T
#define IT_2   0x2 //2T
#define IT_4   0x3 //4T
float uvdata;

// variables for HDC (temperature and humidity)
HDC100X HDC(0x43);
float temp, humi;
```

Folgender code wurde in dem void setup() Teil eingefügt und ist dafür verantwortlich die Sensoren zu aktivieren.


```
  // HDC setup
  HDC.begin(HDC100X_TEMP_HUMI, HDC100X_14BIT, HDC100X_14BIT, DISABLE);
  
  // Light sensor setup
  Wire.begin();
  Wire.beginTransmission(I2C_ADDR);
  Wire.write(0x80 | REG_CONTROL);
  Wire.write(0x03); //Power on
  Wire.endTransmission();
  Wire.beginTransmission(I2C_ADDR);
  Wire.write(0x80 | REG_CONFIG);
  Wire.write(0x00); //400 ms
  Wire.endTransmission();

  // UV sensor
  Wire.begin();
  Wire.beginTransmission(I2C_ADDR2);
  Wire.write((IT_1<<2) | 0x02);
  Wire.endTransmission();
```

Folgender code ist in dem void loop() Teil enthalten und ist dafür verantwortlich die Messungen der Sensoren in die Variablen: humi(Feuchtigkeit), temp(Temperatur), light(Helligkeit) und uvdata(UV-Intensität) zu schreiben.

```
    // HDC variables converted to single string
    humi = HDC.getHumi();
    temp = HDC.getTemp();

    // Request data from light sensor
    Wire.beginTransmission(I2C_ADDR);
    Wire.write(0x80 | REG_DATALOW); 
    Wire.endTransmission();
    Wire.requestFrom(I2C_ADDR, 2); 
    uint16_t low = Wire.read();
    uint16_t high = Wire.read();
    // Catch data if sensor is still sending
    while(Wire.available()){ 
      Wire.read(); 
    }
    // Calculate data from light sensor in LUX
    uint32_t lux; 
    lux = (high << 8) | (low << 0);
    lux = lux * 1; 
    light = lux;

    byte msb=0, lsb=0;
    uint16_t uv;

    // Communicate with UV sensor and request data
    Wire.requestFrom(I2C_ADDR2+1, 1); //MSB
    delay(1);
    if(Wire.available())
      msb = Wire.read();

    Wire.requestFrom(I2C_ADDR2+0, 1); //LSB
    delay(1);
    if(Wire.available())
      lsb = Wire.read();

    // Calculate UV sensor result in uW/cm²
    uv = (msb<<8) | lsb;
    uvdata = 5.625 * uv;
```

Ansonsten haben wir den vorgegebenen code, den wir von der Sensebox-Registrierung erhalten haben genutzt.

## OpenSenseMap

Wir haben auf OpenSenseMap folgende Sensoren registriert:
| Phänomen        | Einheit           | Typ  |
| ------------- |:-------------:| -----:|
| Luftfeuchtigkeit      | % | HDC1008 |
| Temperatur      | °C      |   HDC1008 |
| UV-Intensität | µW/cm²      |    VEML6070 |
| Licht | LUX      |    TSL45315 |

## Stationsaufbau
Unsere Station haben wir in die Schutzbox verpackt und die Lichtsensoren mithilfe von Heißkleber am Deckel angebracht. Der Temperatur-und Feuchtigkeitssensor wurde außen in einer kleinen Plastikbox (als Schutz vor Regen) angebracht.
Danach wurde die Box längs und quer mit Kabelbinder verschnürt und, mithilfe einer Paketschnur und eines Hakens, außen am Fenster angebracht. So kann die Box auf das Dach hängen, fällt aber nicht herunter.



## Kontakt
L. Schmidt  l_schm61[at]uni-muenster.de

I. Wintzer   i_wint04[at]uni-muenster.de



