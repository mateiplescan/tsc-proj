# OpenBook &copy; Matei-Bogdan Plescan 2025


## Descriere Hardware

OpenBook este bazat pe microcontrollerul ESP32-C6. Dispozitivul integrează un display E-Paper, senzori de mediu, stocare de date pe card SD și memorie flash externă, un ceas de timp real (RTC), management avansat al alimentării cu baterie Li-Po și conector de expansiune Qwiic.

### 1. Microcontroller

* **Componentă:** Modul ESP32-C6-WROOM-1-N8 de la Espressif.
* **Procesor:** Nucleu RISC-V single-core pe 32 de biți, până la 160 MHz.
* **Conectivitate:**
    * Wi-Fi 6 (802.11ax)
    * Bluetooth 5 / Bluetooth Low Energy (BLE)
    * Thread/Zigbee (802.15.4)
* **Memorie:** Include SRAM intern și suportă memorie flash externă (utilizată în design).
* **Interfețe:** Dispune de multiple periferice hardware, inclusiv SPI, I2C, SDIO, USB Serial/JTAG, GPIO, ADC, etc., utilizate extensiv în acest design pentru a comunica cu celelalte componente.
* **Rol:** Acționează ca unitate centrală de procesare, controlând toate celelalte module, colectând date de la senzori, gestionând afișajul, stocarea datelor și comunicațiile wireless.

### 2. Managementul Alimentării

* **Intrare Principală:** Conector USB Type-C.
    * Furnizează 5V (VBUS) pentru alimentarea sistemului și încărcarea bateriei.
    * Include circuite de protecție ESD (USBLC6-2SCISY).
* **Încărcare Baterie:**
    * **Componentă:** MCP73831T-2ACI/OT (Li-Po Battery Charging Controller).
    * **Funcționalitate:** Gestionează procesul de încărcare pentru o baterie de la sursa de 5V (USB).
    * **Indicator Status:** Pinul STAT (conectat la ESP32-C6) indică starea procesului de încărcare. Un LED (CHG_LED) oferă indicație vizuală.
* **Baterie:** Conector standard pentru baterie Li-Po.
* **Regulator de Tensiune:**
    * **Componentă:** XC8220A331MR-G (LDO Voltage Regulator).
    * **Funcționalitate:** Primește tensiune fie de la VBUS (USB 5V), fie de la bateria Li-Po (VBAT, ~3.0V-4.2V) și furnizează o tensiune stabilă de **3.3V** pentru alimentarea ESP32-C6 și a majorității perifericelor.
* **Monitorizare Baterie:**
    * **Componentă:** MAX17048G+T10.
    * **Funcționalitate:** Măsoară tensiunea bateriei și estimează starea de încărcare (SoC) și alte informații relevante despre baterie.
    * **Interfață:** Comunică cu ESP32-C6 prin **I2C**.
* **Supervizor Tensiune și Reset:**
    * **Componentă:** BD5229G-TR (Voltage Supervisor).
    * **Funcționalitate:** Monitorizează tensiunea de 3.3V. Dacă tensiunea scade sub un prag prestabilit, generează un semnal de RESET activ-low pe pinul EN al ESP32-C6, asigurând o pornire și funcționare stabilă. Permite și reset manual prin butonul RESET.

### 3. Afișaj (Display)

* **Tip:** E-Paper Display (EPD).
* **Conector:** FH34SR-245-0.5SH_99 (conector FPC cu 24 pini, pas 0.5mm).
* **Circuit Driver:** Include componente discrete (tranzistori MOSFET precum DMG2305UX-7, drivere de poartă precum 5123-11-GE3) necesare pentru a genera tensiunile și semnalele specifice EPD-urilor.
* **Interfață:** Circuitul driver este controlat de ESP32-C6 prin **SPI**.

### 4. Stocare Date

* **Card SD:**
    * **Slot:** Conector standard pentru card microSD.
    * **Interfață:** Conectat la ESP32-C6 folosind interfața **SPI**. Permite stocarea unor volume mari de date (ex: log-uri, fișiere de configurare, imagini).
* **Memorie Flash Externă:**
    * **Componentă:** W25Q512JVEIQ (NOR Flash, 64 Mbit).
    * **Interfață:** Conectată la ESP32-C6 prin **SPI**.
    * **Rol:** Utilizată pentru stocarea firmware-ului principal, a sistemului de fișiere, a configurațiilor sau a datelor critice.

### 5. Senzori și Timp Real

* **Senzor Ambiental:**
    * **Componentă:** BME688 de la Bosch Sensortec.
    * **Funcționalitate:** Măsoară temperatura, umiditatea relativă, presiunea barometrică și compuși organici volatili (VOC), oferind un indice al calității aerului (IAQ).
    * **Interfață:** Comunică cu ESP32-C6 prin **SPI**.
* **Ceas Timp Real (RTC):**
    * **Componentă:** DS3231SN.
    * **Funcționalitate:** Menține data și ora exactă independent de starea de alimentare a ESP32-C6, datorită unei surse de backup (Supercapacitor CPH3225A - C10). Esențial pentru timestamp-uri precise în log-uri.
    * **Interfață:** Comunică cu ESP32-C6 prin **I2C**.

### 6. Interfață Utilizator și Expansiune

* **Butoane:**
    * **RESET:** Buton tactil conectat la intrarea supervizorului de tensiune (și implicit la pinul EN al ESP32-C6) pentru reset manual.
    * **BOOT (IO9):** Buton tactil conectat la GPIO9. Folosit în mod uzual pentru a pune ESP32 în modul bootloader (pentru programare firmware). Poate fi folosit și ca buton de uz general în aplicație.
    * **CHANGE (IO15):** Buton tactil conectat la GPIO15. Destinat utilizării generale în aplicație (ex: schimbare mod afișaj, trigger acțiune).
* **Conector Expansiune:**
    * **Tip:** Conector Qwiic / Stemma QT (standard SparkFun / Adafruit).
    * **Funcționalitate:** Expune magistrala **I2C** a ESP32-C6 (SDA, SCL), împreună cu 3.3V și GND, permițând conectarea ușoară a numeroase module și senzori externi compatibili.

### 7. Alocarea Pinilor ESP32-C6 (Modul U2)

| **Pin** | **Semnal** | **Componentă** | **Rol** |
|---------|----------------------|----------------|---------|
| EN      | RESET                | Buton Reset    | Intrare pentru resetarea sistemului |
| IO0     | INT_RTC              | RTC DS3231SN   | Întrerupere de la RTC pentru evenimente temporale |
| IO1     | 32KHZ                | RTC DS3231SN   | Semnal de ceas 32kHz de la modulul RTC |
| IO2     | MISO                 | Magistrala SPI | Intrare de date de la periferice SPI |
| IO3     | EPD_BUSY             | Afișaj E-Paper | Monitorizarea stării de disponibilitate a afișajului |
| IO4     | SS_SD                | Card SD        | Selectare chip pentru cardul SD |
| IO5     | EPD_DC               | Afișaj E-Paper | Selectare date/comandă pentru afișaj |
| IO6     | SCK                  | Magistrala SPI | Semnal de ceas pentru comunicația SPI |
| IO7     | MOSI                 | Magistrala SPI | Ieșire de date către periferice SPI |
| IO8     | GPIO8                | Uz general     | Funcții auxiliare |
| IO9     | IO/BOOT              | Buton Boot     | Selectare mod boot/programare |
| IO10    | EPD_CS               | Afișaj E-Paper | Selectare chip pentru afișajul E-Paper |
| IO11    | FLASH_CS             | Flash NOR extern | Selectare chip pentru memoria flash |
| IO12    | USB_D-               | Interfață USB  | Linie de date USB negativă |
| IO13    | USB_D+               | Interfață USB  | Linie de date USB pozitivă |
| IO15    | IO/CHANGE            | Buton Change   | Detectare schimbare de stare |
| IO16/TXD0 | TX                 | Debug Serial   | Transmitere UART |
| IO17/RXD0 | RX                 | Debug Serial   | Recepție UART |
| IO18    | RTC_RST              | RTC DS3231SN   | Control reset pentru RTC |
| IO19    | I2C_PW               | Magistrala I2C | Control alimentare pentru dispozitivele I2C |
| IO20    | EPD_3V3_C            | Afișaj E-Paper | Control alimentare pentru afișaj |
| IO21    | SDA                  | Magistrala I2C | Linie de date I2C |
| IO22    | SCL                  | Magistrala I2C | Linie de ceas I2C |
| IO23    | EPD_RST              | Afișaj E-Paper | Control reset pentru afișaj |

### Justificarea utilizării pinilor ESP32-C6 în DeskAssistant v19:

Alocarea pinilor ESP32-C6 în acest design a fost făcută cu atenție pentru a optimiza funcționalitatea și a gestiona eficient resursele limitate ale microcontrolerului:

- **Grupare funcțională**: Pinii sunt organizați pentru a grupa funcționalitățile similare. De exemplu, pinii pentru interfața SPI (MISO, MOSI, SCK) sunt plasați strategic pentru a facilita rutarea PCB și a reduce interferențele.

- **Pini speciali pentru interfețe standard**: IO21 și IO22 sunt utilizați pentru SDA și SCL, fiind pinii standard recomandați pentru I2C pe ESP32. Similar, IO16 și IO17 sunt utilizați pentru funcțiile UART datorită implementării hardware dedicate.

- **Pini GPIO cu funcționalități speciale pentru afișajul E-Paper**: Afișajul E-Paper necesită control complex, de aceea are alocați mai mulți pini (EPD_BUSY, EPD_DC, EPD_CS, EPD_RST, EPD_3V3_C) pentru funcționarea corectă și eficientă.

- **Management energie**: Pinul IO19 este dedicat controlului alimentării dispozitivelor I2C, permițând economisirea energiei prin oprirea acestora când nu sunt utilizate, esențial pentru un dispozitiv alimentat de baterie.

- **Comunicație cu memoria externă**: Memoria flash NOR necesită acces rapid și fiabil, de aceea utilizează un pin dedicat pentru selecție (IO11) pe lângă liniile standard SPI.

- **Conectivitate USB**: Pinii IO12 și IO13 sunt utilizați pentru USB_D- și USB_D+, permițând conectivitatea directă prin USB fără circuite externe suplimentare.

- **Integrare RTC**: Modulul RTC are multiple conexiuni (INT_RTC, 32KHZ, RTC_RST) pentru o funcționalitate completă, oferind atât ceas precis, cât și capacitatea de a trezi microcontrolerul din starea de somn profund.
