# ⌚ Smartwatch Hardware Design - nRF52840

Acest repository conține documentația și fișierele de fabricație (Schematic, PCB Layout, BOM, Pick and Place, Gerbers) pentru un prototip de smartwatch ultra-low-power. Proiectul este construit în jurul ecosistemului Nordic Semiconductor și pune accent pe eficiența energetică extremă (standby de ordinul microamperilor) și pe o topologie PCB densă.

---

## 🛠️ 1. Arhitectura Hardware & Componente

Sistemul este împărțit în patru blocuri logice majore, conform schemei electrice:

### 1.1 Procesare & Conectivitate Radio
* **Microcontroller (MCU):** Nordic **nRF52840** (ARM® Cortex®-M4F la 64 MHz). Ales pentru consumul excelent în Deep Sleep și suportul nativ Bluetooth 5.0 Low Energy (BLE).
* **Cristal RTC:** 32.768 KHz (cu condensatoare de sarcină de 12pF) pentru menținerea timpului real cu un consum minim de energie.
* **Antenă:** Antenă ceramică SMD tip Chip, model **2450AT18B100E** (Johanson Technology), acompaniată de o rețea pasivă (Pi-matching network) pentru adaptarea impedanței la 2.4 GHz.

### 1.2 Interfață Utilizator (Afișaj)
* **Ecran E-Paper (e-Ink):** Conectat la PCB printr-un conector FPC cu 24 de pini (**503480-2400**).
* **Circuit de Boost E-Paper:** Panourile e-Ink necesită tensiuni specifice pentru acționarea stratului de cerneală. Placa include circuitul complet de comandă recomandat, folosind un inductor de **68uH** (L5) și diode Schottky **MBR0530** (D1, D2, D3), alături de condensatorii de filtraj asociați (10uF, 47uF).

### 1.3 Senzor de Mișcare
* **IMU (Inertial Measurement Unit):** Bosch **BMA423**. Este un accelerometru digital pe 3 axe optimizat special pentru *wearables*. Conține algoritmi hardware integrați (low-power) pentru numărarea pașilor (pedometru) și detectarea gesturilor (ex: *tilt-to-wake*), scutind MCU-ul de calcule complexe.

### 1.4 Power Management (Alimentare)
* **LiPo Charger:** **BQ25180YBGR**. Un IC dedicat managementului bateriilor mici de tip litiu-polimer. Oferă un control precis al curentului de încărcare (esențial pentru bateriile de smartwatch) și terminare sigură a ciclului de încărcare.
* **Regulator DC/DC Buck-Boost:** **RT6150AWSC**. Aceasta este o componentă critică a designului. Deoarece bateria LiPo variază de la 4.2V (plină) la 3.0V (goală), acest convertor menține tensiunea sistemului (VOUT) la un nivel stabil de 3.3V, trecând automat din modul *Buck* (coborâtor) în modul *Boost* (ridicător) pe măsură ce bateria se descarcă.

---

## 📌 2. Alocarea Pinilor nRF52840 (Interfețe)

Conexiunile fizice între MCU și periferice au fost rutate pentru a minimiza încrucișările și a asigura integritatea semnalelor.

| Componentă | Pin MCU | Interfață / Rol | Justificare & Funcționalitate |
| :--- | :---: | :---: | :--- |
| **BMA423 (IMU)** | `[P0.XX]` | I2C (SDA) | Magistrală de comunicație date. |
| **BMA423 (IMU)** | `[P0.XX]` | I2C (SCL) | Magistrală de clock (SCL). BMA423 lucrează ca slave I2C. |
| **BMA423 (IMU)** | `[P0.XX]` | INT | Pin de interrupt hardware. Trezește MCU-ul când detectează o mișcare a încheieturii. |
| **E-Paper (EPD)** | `[P0.XX]` | SPI (MOSI) | Magistrala SPI rapidă pentru trimiterea frame-buffer-ului (imaginii) către ecran. |
| **E-Paper (EPD)** | `[P0.XX]` | SPI (SCK) | Clock pentru magistrala SPI. |
| **E-Paper (EPD)** | `[P0.XX]` | GPIO (BUSY) | Pin de stare (Input). MCU-ul intră în sleep și așteaptă ca acest pin să semnalizeze terminarea refresh-ului fizic al ecranului. |
| **SWD / Debug** | N/A | SWDIO / SWDCLK | Programarea se face via un conector de suprafață **TC2030-IDC** (Tag-Connect) cu 6 pad-uri, economisind masiv spațiu pe placă. |

---

## 🔋 3. Analiza Consumului de Energie (Power Budget Estimativ)

Ecosistemul hardware a fost ales pentru a maximiza autonomia unei baterii de capacitate redusă (ex: 150-200 mAh).

1.  **Standby / Deep Sleep (Regim majoritar):** * nRF52840 (RTC activ) + BMA423 (Low-power mode) + RT6150 (Quiescent) = **Aprox. 5 - 10 µA**.
    * *E-Paper-ul consumă 0 µA pentru a menține imaginea afișată.*
2.  **Evenimente Active (Scurte):**
    * **Refresh Ecran:** Curentul absorbit de circuitul de boost este de **aprox. 8 - 15 mA** timp de ~1-2 secunde, doar în momentul actualizării.
    * **Transmisie BLE:** Vârfuri scurte de curent (Tx Peak) la transmiterea pachetelor radio.
3.  **Concluzie:** Curentul mediu de descărcare este dominat de cât de des se actualizează ecranul (ex: la minut vs. la secundă) și de cât de des se sincronizează Bluetooth-ul. Într-un regim optimizat de ceas de mână, hardware-ul permite funcționarea de ordinul săptămânilor.

---

## 📐 4. Design Log & Integrare Mecanică

### Layout PCB & Integritate Electromagnetică
* **Planuri de Masă & Via Stitching:** Pentru a ecrana zgomotul (EMI) și a asigura un nivel de referință stabil pentru circuitul RF (Antenă), placa dispune de planuri de masă (`GND`) solide pe ambele straturi (TOP și BOTTOM). Acestea sunt cusute între ele (Via Stitching) pe toată suprafața disponibilă.
* **Zona RF:** S-a creat un *Polygon Cutout* (zonă liberă de cupru) strict sub și în jurul antenei ceramice 2450AT18B100E pentru a nu ecrana și a nu dezacorda propagarea undelor de 2.4 GHz.
* **Necking Down pe Alimentare:** Traseele de putere au fost rutate pe cât posibil cu o lățime de 0.3mm. Totuși, în zonele foarte congestionate (în proximitatea PMIC-ului BQ25180 și a procesorului), traseele au fost îngustate local (*neck-down*) pentru a permite amplasarea condensatorilor de decuplare (ex: C38, C39) cât mai aproape de pini.

### Status Integrare 3D & Carcasă (Mechanical)
1. **Modelul Nativ:** Fișierul `.f3z` din folderul *Mechanical* conține designul nativ din Fusion 360 al plăcii. Au fost rezolvate erorile vizuale de extrudare a pachetelor lipsă (cărămizile negre) specifice componentelelor custom.
2. **Exploded View (WIP):** Etapa de asamblare este în desfășurare. S-a realizat exportul plăcii (Switch to 3D PCB). Amplasarea conceptuală a dispozitivului urmează un design tip "sandwich":
   * **Bateria:** Plasată sub PCB (pe fața BOTTOM).
   * **Ecranul E-Paper:** Poziționat deasupra PCB-ului (pe fața TOP), conectat prin cablul FPC îndoit.
   * *Notă:* Modelele 3D exacte pentru baterie și display urmează să fie importate și aliniate pentru exportul final de ansamblu tip `.step`.
