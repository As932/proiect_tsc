# ⌚ Smartwatch Hardware Design - nRF52840

Acest repository conține stadiul actual al design-ului hardware (Schematic, PCB Layout și exporturi de fabricație) pentru un smartwatch ultra-low-power, optimizat pentru eficiență energetică.

Proiectul este construit în jurul ecosistemului Nordic Semiconductor și include un circuit de power management dedicat, un afișaj E-Paper și monitorizare a mișcării, toate integrate pe un PCB custom cu o topologie densă.

---

## 🛠️ 1. Arhitectura Hardware & Componente

Sistemul a fost împărțit în mai multe blocuri logice pentru a asigura o rutare curată:

* **Microcontroller (MCU) & Conectivitate:** * **nRF52840 (Nordic Semiconductor):** Creierul sistemului, ales pentru capabilitățile Bluetooth Low Energy (BLE) și consumul extrem de redus.
    * **Antenă Chip (2450AT18B100E):** Antenă ceramică SMD de 2.4GHz, acompaniată de o rețea de adaptare a impedanței pentru a maximiza raza de acțiune radio.

* **Afișaj (Display):**
    * **E-Paper (e-Ink) Display:** Conectat printr-un conector FPC cu 24 de pini (503480-2400). Placa include circuitul complet de *boost* necesar panoului E-Paper (cu inductor de 68uH și diode Schottky MBR0530). Am ales e-Ink pentru consumul zero de energie în menținerea imaginii statice.

* **Senzori:**
    * **IMU (BMA423 - Bosch):** Accelerometru cu 3 axe, optimizat special pentru *wearables* (include capabilități de pedometru). Comunică cu procesorul prin interfața I2C.

* **Power Management (Alimentare):**
    * **Încărcare LiPo (BQ25180YBGR):** IC dedicat pentru managementul bateriilor de capacitate mică, oferind un curent de încărcare stabil și sigur.
    * **DC/DC Buck-Boost (RT6150AWSC):** Asigură o tensiune stabilă pentru întreg sistemul (3.3V), garantând funcționarea chiar și atunci când tensiunea bateriei variază pe parcursul descărcării.

---

## 📌 2. Alocarea Pinilor & Interfețe

* **Interfața I2C (SDA / SCL):** Folosită pentru comunicația cu senzorul BMA423.
* **Programare & Debug:** Pentru a economisi spațiu critic pe PCB, pinii `SWDIO`, `SWDCLK` și `RESET` au fost scoși pe un footprint de tip *Tag-Connect* (TC2030-IDC), eliminând necesitatea unui conector voluminos sudat pe placă. 

---

## 📐 3. Design Log & Integrare Mecanică

Pentru validarea hardware s-au luat următoarele decizii de design:

1.  **Reguli de Rutare și Compromisuri (Necking Down):**
    * S-a respectat o grosime a traseelor de semnal de 0.15mm.
    * Pentru semnalele de alimentare, am încercat maximizarea grosimii la 0.3mm. Totuși, în zonele foarte dense, a fost aplicată tehnica de *neck-down* (îngustare pe porțiuni mici). Acest compromis a permis păstrarea componentelor (ex: condensatorii de decuplare) cât mai aproape de pinii IC-urilor, esențial pentru stabilitatea semnalului.
2.  **Integritate Electromagnetică (Via Stitching):**
    * Placa dispune de plane de masă continue pe Top și Bottom. Acestea sunt interconectate printr-o rețea de **Via Stitching** pentru a asigura un drum de întoarcere ferm pentru curenți și pentru a ecrana interferențele.
    * În zona antenei, planul de masă este complet decupat (Polygon Cutout) pentru a permite o propagare radio corectă.
3.  **Status Integrare 3D (WIP):**
    * **Partea de asamblare mecanică completă este momentan în lucru.** În acest stadiu, a fost generat doar modelul 3D brut al PCB-ului din mediul 2D (*Push to 3D PCB* în Fusion 360). Urmează ca în etapa următoare să importăm modelele reale pentru componentele lipsă, baterie și display, pentru a finaliza ansamblul în interiorul carcasei.
