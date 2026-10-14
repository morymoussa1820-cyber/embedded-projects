# Guide CubeMX + Simulink — TP2 (NUCLEO-F446RE)

Ce guide reprend tous les réglages qu'on a dû trouver un par un pendant la session, avec ta vraie carte (**NUCLEO-F446RE**). En les faisant tous **dans l'ordre, d'un coup**, tu évites tous les allers-retours qu'on a eus.

⚠️ **Vérifie toujours le modèle écrit sur ta carte physique avant de commencer** (imprimé sur le circuit imprimé). Si CubeMX/Simulink ciblent une carte différente de celle réellement branchée, le déploiement échoue même quand toute la configuration logicielle est parfaite.

Pinout retenu pour les 3 LED (adaptable aux autres manips) :
- D2 = PA10
- D4 = PB5
- D7 = PA8

---

## Partie 1 — STM32CubeMX

### 1. Créer le projet
1. Ouvre STM32CubeMX → **New Project**
2. Onglet **Board Selector** → champ **Commercial Part Number** → tape `NUCLEO-F446RE` → sélectionne la carte
3. À la question "Initialize all peripherals with their default Mode?" → **Yes**
4. Dans la fenêtre "Board Project Options" qui suit, laisse coché par défaut (USER LED, USER BUTTON) → **OK**

### 2. Configurer les pins des LED
Dans **Pinout & Configuration**, clique sur chaque pin sur le schéma de la puce et choisis **GPIO_Output** :
- PA10 (D2)
- PB5 (D4)
- PA8 (D7)

Pour les manips suivantes (bouton, capteur, etc.), configure la pin voulue avec le mode correspondant (GPIO_Input pour un bouton, etc.) — le reste du guide ne change pas.

### 3. Changer la source de base de temps
1. **System Core → SYS**
2. Champ **Timebase Source** : change `SysTick` → **TIM1**

(Obligatoire : Simulink a besoin du SysTick pour lui-même.)

### 4. Configurer USART2 pour l'External Mode de Simulink
USART2 doit rester activé (c'est le port série virtuel du ST-Link, utilisé par Simulink pour monitorer la carte pendant qu'elle tourne) :

1. **Connectivity → USART2** → Mode = **Asynchronous** (normalement déjà le cas par défaut)
2. Toujours sur USART2, en bas dans **Configuration → DMA Settings** → clique **Add** → sélectionne **USART2_RX**

### 5. Réglages du projet
1. **Project Manager → Project**
2. Remplis **Project Name** (ex. `Manip_1Led_F446`) et **Project Location** (Browse → choisis ton dossier de travail)
3. Coche **"Do not generate the main()"**
4. **Décoche "Generate Under Root"** (coché par défaut, mais doit être décoché)
5. Vérifie **Toolchain / IDE = STM32CubeIDE**

### 6. Passer les drivers en LL
1. **Project Manager → Advanced Settings**
2. Tableau **Driver Selector** en haut :
   - Déroule **USART** (flèche `>`) → ligne **USART2** → change `HAL` en **LL**
   - Ligne **DMA** → change `HAL` en **LL**
   - (RCC reste en HAL, GPIO peut rester tel quel)

### 7. Décocher "Visibility (Static)"
Toujours dans **Advanced Settings**, tableau **Generated Function Calls** en bas :
- Décoche la case **Visibility (Static)** sur **toutes les lignes** (RCC, GPIO, DMA, USART2)

### 8. Générer le code
1. Clique **GENERATE CODE** (en haut à droite)
2. Attends **"The Code is successfully generated"**
3. Clique **Close**
4. Sauvegarde le projet (**Ctrl+S**)

Ton `.ioc` est maintenant prêt pour toutes les manips du TP qui utilisent ces pins.

---

## Partie 2 — Simulink

### 1. Placer les blocs
Dans la **Library Browser** → **STM32 Microcontroller Blockset** → **STM32F4xx Based Boards** :
- Glisse un bloc **Digital Port Write** par sortie utilisée (3 pour les 3 LED)
- Configure chaque bloc : `Port` = GPIOA ou GPIOB selon la pin, `Pin` = le numéro correspondant, et **décoche** "Access port pins as array"
- Relie chaque bloc à sa source (ex. Pulse Generator pour un clignotement)

### 2. Configurer le matériel cible
1. Onglet **HARDWARE** → **Hardware Board** = **STM32F4xx Based**
2. **Hardware Settings → Hardware Implementation → Target hardware resources → Build options**
3. **Browse...** → sélectionne ton fichier `.ioc` configuré en Partie 1
4. Vérifie que **Device id** affiche bien **STM32F446R(C-E)Tx** et **Device family = STM32F4**

### 3. Vérifier External Mode (ne pas désactiver)
1. Dans **Hardware Settings**, barre de recherche → tape `external`
2. Sous **Code Generation → Interface**, case **"External mode"** → doit rester **cochée**
3. OK

### 4. Lancer
1. Dans le ruban HARDWARE, groupe **MODE**, vérifie que **"Run on board (External mode)"** est sélectionné (petite flèche sous "Run on board")
2. Clique **"Build, Deploy & Start"** (section DEPLOY) — **pas** "Monitor & Tune"

**Build, Deploy & Start** compile, flashe le code sur la carte, et démarre l'exécution — c'est le bouton à utiliser à chaque fois.
**Monitor & Tune** sert uniquement à se connecter à un programme déjà flashé pour visualiser des signaux en direct — il ne compile ni ne flashe rien.

---

## Pense-bête : erreurs rencontrées et leur solution

| Erreur | Solution |
|---|---|
| MBED OS tool is not installed | Ne pas utiliser les blocs MBED (obsolètes) — utiliser STM32 Microcontroller Blockset |
| Generation of main() must be disabled | Project Manager → Project → cocher "Do not generate the main()" |
| SysTick timer is selected as timebase | System Core → SYS → Timebase Source → TIM1 |
| No parameters... Visibility(static) | Advanced Settings → décocher Visibility (Static) pour toutes les lignes |
| Incorrect driver layer for USART2 / DMA | Advanced Settings → Driver Selector → passer en LL |
| Add DMA settings for USART2_RX | Connectivity → USART2 → DMA Settings → Add → USART2_RX |
| Mode for USART2 should not be Disable | Ne pas désactiver USART2, il sert à l'External Mode |
| Generate code under root must be disabled | Project Manager → Project → décocher "Generate Under Root" |
| STM32CubeProgrammer tool is not installed | Dans MATLAB : `stm32setup` → étape CubeProgrammer → Browse vers le sous-dossier **bin** de son installation → Validate |
| No STM32 MCU connected | Vérifier que la carte CIBLÉE dans CubeMX/Simulink correspond bien au modèle PHYSIQUE branché (regarder le texte imprimé sur la carte) |

---

## Pour les manips suivantes (3 à 6)

La base ci-dessus (main() désactivé, TIM1, USART2+DMA+LL, visibility décochée, Generate Under Root décoché) ne change plus une fois faite. Pour ajouter un nouveau périphérique (bouton, PWM, ADC, etc.) :

1. Configure la pin dans **Pinout & Configuration**
2. Si CubeMX ajoute une nouvelle ligne dans **Generated Function Calls**, décoche sa case **Visibility (Static)**
3. Si CubeMX te demande un driver layer pour ce nouveau périphérique, mets-le en **LL**
4. **GENERATE CODE**, puis ajoute le bloc Simulink correspondant (ex. **Digital Port Read** pour un bouton, **PWM Output** pour du PWM) et relie-le

Si une nouvelle erreur apparaît malgré tout, note le message exact — on la réglera rapidement, elle fera probablement partie du même type d'enchaînement.
