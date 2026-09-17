# Magnetic Transmitting Loop Antenna Controller (ML Controller)

**Contrôleur d'Antenne Boucle Magnétique d'Émission**

* **Auteur :** Loftur E. Jonasson (TF3LJ / VE2AO / VE2LJX)
* **Version du firmware :** 4.10 (21 Juin 2020)
* **Plateforme :** Teensy 3.1 / 3.2 (Microcontrôleur ARM Cortex-M4)
* **Licence :** GNU General Public License v3.0 (GPLv3)

---

## 📖 Description Générale

Ce projet contient le code source du **contrôleur automatique d'antenne boucle magnétique d'émission** (*Magnetic Transmitting Loop Antenna Controller*).

Une antenne boucle magnétique possédant une bande passante très étroite, chaque changement de fréquence nécessite un réajustement précis de son condensateur d'accord. Ce contrôleur automatise l'accord de l'antenne en entraînant un moteur pas-à-pas connecté au condensateur variable (sous vide, papillon, etc.), synchronisé en temps réel avec le transcepteur radioamateur via son interface CAT et assisté par une mesure automatique du ROS (*SWR Autotune*).

---

## ✨ Fonctionnalités Principales

- **Commande Moteur Pas-à-Pas de Précision :**
  - Support de drivers bipolaires : Allegro A4975 (paire) ou modules type StepStick/Pololu (TI DRV8825, Allegro A4988).
  - Gestion du microstepping variable (pas entier, 1/2, 1/4, 1/8 de pas) et vitesse adaptative selon la distance.
  - Compensation automatique du jeu mécanique (*Backlash compensation*).
  - Détection optionnelle de fins de course (*End-stop switches*) ou gestion intelligente pour condensateur sous vide et papillon.

- **Intégration Radio / Protocoles CAT (Suivi Automatique de Fréquence) :**
  - Connexion série UART (TTL ou RS232 inversé, 1200 à 115200 bps) avec plus de 17 configurations de radios :
    - **ICOM :** CI-V Auto / Poll
    - **Kenwood :** TS-440, TS-870, TS-480, TS-590, TS-2000
    - **Yaesu :** FT-100, FT-747, FT-817/847/857/897, FT-920, FT-990, FT-1000MP / Mk-V, FT-450 / 950 / 1200 / 2000 / 3000 / 5000
    - **Elecraft :** K3, KX3 (Auto / Poll)
    - **TenTec :** Modes binaire et ASCII
    - **Mode Pseudo-VFO :** Permet une utilisation autonome sans radio connectée, où l'encodeur rotatif simule le VFO de la radio.
  - Prise en charge de 4 profils radio sauvegardés en EEPROM.

- **Wattmètre, ROS-mètre et Accord Automatique (*SWR Autotune*) :**
  - Mesure de la puissance directe/réfléchie et du ROS (SWR) :
    - Option détecteur logarithmique AD8307 (haute dynamique).
    - Option pont à diodes (Tandem Match ou pont de Bruene).
  - **SWR Autotune :** Recherche automatique du creux de ROS (*SWR Dip Hunt*) en commandant automatiquement la radio (passage temporaire en mode AM, baisse de la puissance d'émission, activation PTT, recherche du ROS minimum, puis restauration des réglages initiaux).

- **Gestion des Mémoires EEPROM & Multi-antennes :**
  - Stockage de jusqu'à 200 couples (Fréquence, Position du moteur).
  - Interpolation linéaire entre les points de calibration pour un accord fluide sur toutes les bandes.
  - Prise en charge de la gestion de 2 ou 3 antennes sélectionnées automatiquement selon la fréquence ou manuellement.
  - Commutation de condensateurs fixes ou de prises de couplage via sorties numériques (bandswitching).

- **Interface Utilisateur & Affichage :**
  - Écran LCD 20x4 (HD44780 ou OLED) piloté par la bibliothèque à haut débit `LiquidCrystalFast`.
  - Bargraphe haute résolution sur LCD simulé type LP-100 avec indicateur de puissance crête (*Peak Hold*).
  - Économiseur d'écran paramétrable avec message personnalisé ou fréquence.
  - Navigation complète par menu via encodeur rotatif et boutons poussoirs.

- **Interface USB & Contrôle à Distance :**
  - Port USB virtuel (série) pour configuration, sauvegarde/restauration des mémoires (`$memoryget`, `$memoryset`), contrôle du SWR tune, télémétrie en continu (`$pcont`), et débogage.

---

## 🛠️ Architecture Matérielle (Hardware)

* **Microcontrôleur :** PJRC Teensy 3.1 ou Teensy 3.2 (32-bit ARM Cortex-M4 à 72 MHz).
* **Pilote de Moteur Pas-à-Pas :**
  - Soit 2x Allegro A4975.
  - Soit 1x Pololu DRV8825 / A4988 (`#define DRV8825STEPPER 1`).
* **Affichage :** Écran LCD 20x4 caractères (4 bits de données + RS, RW, E).
* **Interface Radio :** Port UART (Pins 0/RX1 et 1/TX1) avec inverseur RS232 / TTL optionnel.
* **Mesure RF :**
  - Entrées analogiques A10 (Pfwd) et A11 (Pref) pour la mesure de puissance directe et réfléchie.
  - Option I2C (AD7991) sur les broches 29/30.
* **Commandes Utilisateur :**
  - Encodeur rotatif quadrature (Pins 9/10).
  - Bouton menu / validation (EnactSW, Pin 23 / A9).
  - Boutons Up / Down (Pins 11 et 12).
  - Capteurs de fin de course (Pins 21 et 22, si activés).

---

## 📁 Structure des Fichiers du Projet

| Fichier | Description |
| :--- | :--- |
| `ML.h` | Fichier d'en-tête principal : définitions matérielles, options de compilation (`#define`), structures de données et constantes. |
| `ML_v410.ino` | Fichier principal : initialisation (`setup()`), boucle principale (`loop()`), tâches périodiques et suivi de fréquence. |
| `ML_Display.ino` | Gestion de l'écran LCD 20x4, du buffer virtuel, de l'économiseur d'écran et des bargraphes de puissance/ROS. |
| `ML_Menu.ino` | Arborescence et gestion du menu de configuration utilisateur (réglages moteur, radios, calibration, mémoires). |
| `ML_PSWR.ino` | Échantillonnage ADC, calculs de la puissance directe/réfléchie (mW, W), du PEP, du ROS et étalonnage (AD8307 / diodes). |
| `ML_Pos_Mgmnt.ino` | Gestion des mémoires de position, tri des présélections, interpolation fréquence/position et démultiplication variable. |
| `ML_SWRtune.ino` | Algorithme de recherche automatique du creux de ROS (*SWR Autotune*), contrôle automatique du transcepteur en PTT/AM. |
| `ML_TRX.ino` | Pilote de communication CAT pour les différents transcepteurs (ICOM, Kenwood, Yaesu, Elecraft, TenTec, Pseudo-VFO). |
| `ML_USB.ino` | Interprète de commandes série USB pour le contrôle distant, l'export/import des mémoires et le débogage. |
| `ML__Switches_and_Stepper.ino` | Gestion matérielle des boutons poussoirs (anti-rebond, détection appui court/long) et génération des impulsions du moteur pas-à-pas. |
| `_EEPROMAnything.h` | Fonctions génériques (templates) de lecture/écriture de structures en EEPROM. |
| `gpl.txt` | Texte complet de la licence publique générale GNU (GPLv3). |

---

## ⚙️ Configuration (`ML.h`)

Toutes les options matérielles et logicielles sont configurables dans le fichier `ML.h` avant la compilation :

```c
// Choix du driver moteur pas-à-pas (0 = 2x A4975, 1 = DRV8825 / A4988)
#define DRV8825STEPPER     1

// Détecteur logarithmique AD8307 (0 = Non, 1 = Oui)
#define AD8307_INSTALLED   0

// Activation de l'autotune SWR et du wattmètre (0 = Désactivé, 1 = Activé)
#define PSWR_AUTOTUNE      0

// Gestion des fins de course (1 = Condensateur sous vide sans fin de course, 2 = Avec fin de course, 3 = Papillon)
#define ENDSTOP_OPT        1

// Radio par défaut à l'allumage (0 = ICOM CI-V Auto, 1 = ICOM Poll, 4 = Kenwood TS-480/2000, 7 = Yaesu FT-8x7, 13 = Elecraft K3, 17 = Pseudo-VFO...)
#define DEFAULT_RADIO      1
```

---

## 🔨 Compilation et Installation

1. **Prérequis matériels & logiciels :**
   - [Arduino IDE](https://www.arduino.cc/en/software) (version 1.8.x recommandée).
   - Extension [Teensyduino](https://www.pjrc.com/teensy/td_download.html) installée dans l'Arduino IDE.
   - Carte sélectionnée : **Teensy 3.1** ou **Teensy 3.2**.

2. **Bibliothèques requises :**
   - `LiquidCrystalFast`
   - `Metro`
   - `Encoder`
   - `ADC`
   - `EEPROM`
   - `i2c_t3` (si `WIRE_ENABLED` est activé)

3. **Procédure de compilation :**
   - Ouvrir `ML_v410.ino` dans l'Arduino IDE.
   - Ajuster les paramètres souhaités dans `ML.h` (Type de driver moteur, type de radio, options SWR).
   - Sélectionner la carte : *Outils > Type de carte > Teensy3.2/3.1*.
   - Régler la fréquence d'horloge sur *72 MHz*.
   - Cliquer sur **Vérifier / Compiler** puis télverser sur le Teensy via USB.

---

## 💬 Commandes USB (Série)

Le contrôleur expose un port série virtuel USB permettant le contrôle à distance. Toutes les commandes commencent par `$`.

### Commandes Générales & Fréquence
- `$frqget` : Renvoie la fréquence active en Hz.
- `$frqset <Hz>` : Définit la fréquence active (ex: `$frqset 14100000` pour 14.1 MHz).
- `$version` : Affiche la version et la date du firmware.
- `$help` : Affiche la liste d'aide des commandes disponibles.

### Gestion de la Mémoire EEPROM
- `$memoryget` : Exporte toutes les présélections enregistrées (`Index Fréquence Position`).
- `$memoryset <Index> <Fréquence_Hz> <Position>` : Enregistre une présélection.
- `$memoryclear` : Efface toutes les présélections fréquence/position.
- `$memorywipe` : Reconstitution complète de l'EEPROM (remise à zéro d'usine).

### SWR & Autotune
- `$swrtune` : Lance une procédure automatique d'accord SWR.
- `$swrtuneup` / `$swrtunedown` : Lance un accord SWR vers le haut ou vers le bas.
- `$swrtunestatus` : Renvoie le statut de l'accord (En cours, Succès, Échec, Pas de puissance).
- `$toggleautotune` : Active/désactive l'accord automatique sur ROS élevé.
- `$recalibrate` : Effectue un réalignement de position (recalibration).

### Télémétrie Wattmètre / ROS-mètre
- `$ppoll` : Rapport instantané unique de puissance et ROS.
- `$pinst`, `$ppk`, `$ppep`, `$plong` : Rapports sous forme lisible (instant, crête 100ms, PEP 1s, détaillé).
- `$pcont` : Mode de rapport continu à 10 Hz via USB.

---

## 📜 Licence

Ce projet est distribué sous la licence **GNU General Public License v3.0**. Consultez le fichier [`gpl.txt`](./gpl.txt) pour plus de détails.
