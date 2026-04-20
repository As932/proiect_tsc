# ⌚ Smartwatch Hardware Design - nRF52840

Acest repository conține design-ul hardware complet (Schematic, PCB Layout, fișiere de fabricație) pentru un smartwatch ultra-low-power, optimizat pentru eficiență energetică și portabilitate.

Proiectul este construit în jurul ecosistemului Nordic Semiconductor și include un afișaj E-Paper, monitorizare inteligentă a bateriei și feedback haptic, toate integrate pe un PCB custom cu o topologie densă.

---

## 🛠️ 1. Arhitectura Hardware & Componente

Sistemul a fost împărțit în mai multe blocuri logice pentru a asigura o rutare curată și o izolare bună a semnalelor de radio-frecvență:

* **Microcontroller (MCU) & Conectivitate:** * **nRF52840 (Nordic Semiconductor):** Creierul sistemului. Oferă conectivitate Bluetooth 5.0 Low Energy (BLE) și un procesor ARM Cortex-M4F. A fost ales pentru consumul extrem de mic în stand-by și capabilitățile radio excelente.
    * **Antenă Chip (2450AT18B100E):** Antenă ceramică SMD de 2.4GHz, acompaniată de o rețea de adaptare a impedanței (Pi-matching network) pentru a maximiza raza de acțiune BLE.

* **Afișaj (Display):**
    * **E-Paper (e-Ink) Display:** Conectat printr-un conector FPC cu 24 de pini (503480-2400). S-a optat pentru tehnologia E-Paper deoarece consumă energie doar în momentul actualizării imaginii (0 mA pentru a menține imaginea afișată). Placa include circuitul complet de *boost* (cu diode MBR0530 și inductor) necesar panoului E-Paper.

* **Senzori & Feedback (UI):**
    * **IMU (BMA423 - Bosch):** Accelerometru cu 3 axe, optimizat special pentru *wearables* (include algoritmi hardware nativi pentru recunoașterea pașilor/pedomentru). 
    * **Haptic Driver (DRV2605):** Driver I2C capabil să controleze un motoraș cu vibrații (LRA/ERM) pentru alarme și notificări tactile.
    * **Butoane Fizice:** 4 butoane tactice (SW1 - SW4) pentru navigarea în meniu.

* **Power Management (Alimentare):**
    * **Încărcare LiPo (BQ25180):** PMIC dedicat pentru baterii mici, oferă un curent de încărcare sigur via portul USB-C (cu protecție ESD inclusă - USBLC6).
    * **Fuel Gauge (MAX17048):** Monitorizează precis tensiunea bateriei (State-of-Charge) și comunică procentul real prin I2C către MCU.
    * **DC/DC Buck-Boost (RT6150):** Asigură o tensiune stabilă de 3.3V pentru întreg sistemul, chiar și atunci când bateria LiPo scade sub acest prag (ex: 3.0V).

---

## 📌 2. Alocarea Pinilor (Pinout nRF52840)

Datorită flexibilității matricei de pini de pe nRF52840, pinii au fost alocați strategic pentru a minimiza încrucișările de trasee pe PCB și a menține o rutare curată (layout optimizat):

* **Interfața I2C (SDA / SCL):**
    * O singură magistrală I2C cu rezistențe de pull-up deservește comunicația cu: **IMU BMA423**, **Fuel Gauge MAX17048** și **Haptic Driver DRV2605**.
    * Traseele de I2C au fost ținute scurte și rutate departe de zona antenei pentru a preveni diafonia (crosstalk).
* **Interfața SPI (E-Paper):**
    * Comunicarea cu ecranul folosește magistrala SPI (MOSI, SCLK) plus pinii de control specifici E-Paper: `EPD_CS` (Chip Select), `EPD_DC` (Data/Command), `EPD_RST` (Reset) și `EPD_BUSY` (intrare pentru a ști când ecranul a terminat refresh-ul).
* **Interfață Utilizator (GPIO):**
    * Pinii pentru butoane folosesc rezistențele interne de pull-up ale MCU-ului pentru a economisi spațiu pe placă. Beneficiază de hardware debouncing.
* **Programare & Debug:**
    * `SWDIO` și `SWDCLK` expuși pe un conector *Tag-Connect* (TC2030) - economisește masiv spațiu pe placă nefiind nevoie de o mufă mamă lipită.

---

## 🔋 3. Profilul de Consum de Energie (Power Budget)

Sistemul a fost gândit pentru a oferi o autonomie de ordinul zilelor/săptămânilor, folosind o baterie LiPo standard de capacitate mică (ex: ~150-200 mAh).

* **System OFF / Deep Sleep:** < `2 µA` (doar RTC-ul din nRF52840 și senzorul BMA423 în mod low-power așteptând mișcare).
* **Idle / BLE Advertising:** `~20 - 40 µA` (vârfuri scurte la transmisie).
* **Afișaj E-Paper:** `~10 - 15 mA` (timp de 1-2 secunde doar în momentul schimbării ecranului, urmat de `0 mA` menținere statică).
* **Haptic Motor:** `~50 - 100 mA` (consumator major, dar activat doar în impulsuri de câteva sute de milisecunde pentru notificări).

---

## 📐 4. Design Log & Integrare Mecanică (Review Notes)

Pentru validarea HW s-au făcut o serie de decizii critice de design care merită menționate:

1.  **Reguli de Rutare și Compromisuri (Necking Down):**
    * S-a respectat o grosime a traseelor de semnal de `0.15mm`.
    * Pentru semnalele de putere (`VCC`, `VBUS`, `VBAT`) am încercat maximizarea la `0.3mm`. Totuși, în zonele foarte dense (în special sub MCU și PMIC), a fost aplicată tehnica de *neck-down* (îngustare pe porțiuni mici). Acest compromis a permis păstrarea componentelor (ex: condensatorii de decuplare) cât mai aproape de pinii IC-urilor, esențial pentru stabilitatea semnalului.
2.  **Integritate Electromagnetică (Via Stitching & Plane de Masă):**
    * Placa dispune de plane de masă continue (Polygon Pour pe Top și Bottom). Acestea sunt interconectate printr-o rețea densă de **Via Stitching** pentru a asigura un drum de întoarcere ferm (return path) pentru curenți și pentru a acționa ca o "cușcă Faraday" împotriva EMI.
    * În zona antenei (Top dreapta), planul de masă este strict interzis (Polygon Cutout) pe toate straturile, pentru a permite o propagare radio neobstrucționată.
3.  **Integrarea 3D și Asamblarea:**
    * Repository-ul include folderul *Mechanical*, conținând un **Exploded View** (fișier `.step`) și fișierul de arhivă `.f3z`. 
    * Pe parcursul modelării 3D (Push to 3D PCB în Fusion 360), modelele schematice generice tip „cărămidă neagră” au fost ascunse intenționat. S-au folosit componente 3D reale pentru asamblare, permițând o simulare mecanică precisă a alinierii PCB-ului cu ecranul, bateria și interiorul carcasei ceasului.
