# MinisforumN5Air
Minisforum N5 Air considerations, 

Je signale un bug du BIOS v1.01 livré sur le Minisforum N5 Air (N5A) :

  le contrôleur Ethernet 10GbE Realtek RTL8127 intégré n'est pas exposé
  par le BIOS lors de l'énumération PCIe et reste donc totalement
  inutilisable, quel que soit l'OS ou le driver.

  === Identification matérielle ===
  - Modèle           : N5A (N5 Air)
  - Fabricant        : Micro Computer (HK) Tech Limited
  - Numéro de série  : MZ108L5P3AQEMLE00212
  - UUID système     : 0f557e80-0644-11f1-9909-1e0e5541dc00
  - SKU              : MGF8NAB
  - Carte mère       : F8NAB (Meigao Innovation Technology - Shen Zhen)
  - N° série carte mère : F8NABR25506D1080241

  === BIOS actuel ===
  - Vendeur          : American Megatrends International, LLC.
  - Version          : 1.01
  - Date de release  : 24/10/2025
  - BIOS Revision    : 5.29

  === Description du bug ===
  Le N5 Air est annoncé avec deux contrôleurs Ethernet Realtek :
    - RTL8126 (5GbE) — VID:DID = 10ec:8126
    - RTL8127 (10GbE) — VID:DID = 10ec:8127

  Sous Linux (kernel 6.19, qui prend nativement en charge le RTL8127
  depuis le mainline 6.15), seul le RTL8126 est visible. Le contrôleur
  RTL8127 n'apparaît PAS dans l'énumération PCIe — il n'est ni listé
  par lspci, ni présent dans /sys/bus/pci/devices/, ni mentionné par le
  journal du noyau au démarrage.

  Ce n'est donc PAS un problème de pilote (le pilote r8127 mainline est
  disponible et chargeable), mais bien un défaut d'initialisation /
  d'exposition du device par le BIOS lui-même.

  === Preuves diagnostiques ===

  # Kernel
  $ uname -r
  6.19.5-2-pve

  # Énumération PCIe — recherche de tous les devices Realtek
  $ lspci -nn | grep -i "10ec:"
  02:00.0 Ethernet controller [0200]: Realtek Semiconductor Co., Ltd.
          RTL8126 5GbE Controller [10ec:8126] (rev 01)

  # Recherche explicite du RTL8127 dans tout l'arbre PCIe
  $ lspci -nn -vv | grep -i "8127"
  (aucune occurrence — le device n'existe pas du point de vue PCIe)

  # Modules noyau chargés
  $ lsmod | grep -E "r8126|r8127"
  r8126   204800   0

  Tests complémentaires effectués :
  - Désactivation de PCIe ASPM via paramètres noyau (pcie_aspm=off,
    pcie_port_pm=off, pci=nommconf) → aucun effet, le device reste
    invisible.
  - Modprobe explicite du module r8127 → le module se charge mais
    ne trouve aucun device à piloter.

  === Demande ===
  1. Confirmation officielle de ce bug sur le BIOS v1.01 du N5 Air
  2. Calendrier prévisionnel de publication d'un BIOS corrigé (v1.02 ou
     ultérieur) qui exposera correctement le RTL8127 sur le bus PCIe
  3. Si une version beta est disponible en interne, je suis volontaire
     pour la tester et remonter les résultats.

  À ma connaissance, des mises à jour BIOS ont été publiées pour les
  modèles N5 et N5 Pro, mais aucune pour le N5 Air à ce jour
  (28 avril 2026). Le modèle étant relativement récent, je suppose
  qu'une release est en préparation.

  Je vous remercie pour votre aide et reste à disposition pour tout
  diagnostic complémentaire.

  Cordialement,
  [Votre nom]

  ---
  🇬🇧 English version

  Subject: BIOS v1.01 bug — N5 Air: 10GbE RTL8127 controller not enumerated on the PCIe bus

  Hello,

  I am reporting a bug in BIOS v1.01 shipped on the Minisforum N5 Air
  (N5A): the on-board 10GbE Realtek RTL8127 Ethernet controller is not
  exposed by the BIOS during PCIe enumeration, making it completely
  unusable regardless of the OS or driver.

  === Hardware identification ===
  - Model              : N5A (N5 Air)
  - Manufacturer       : Micro Computer (HK) Tech Limited
  - Serial Number      : MZ108L5P3AQEMLE00212
  - System UUID        : 0f557e80-0644-11f1-9909-1e0e5541dc00
  - SKU                : MGF8NAB
  - Motherboard        : F8NAB (Meigao Innovation Technology - Shen Zhen)
  - Motherboard serial : F8NABR25506D1080241

  === Current BIOS ===
  - Vendor             : American Megatrends International, LLC.
  - Version            : 1.01
  - Release Date       : 2025-10-24
  - BIOS Revision      : 5.29

  === Bug description ===
  The N5 Air is advertised with two Realtek Ethernet controllers:
    - RTL8126 (5GbE) — VID:DID = 10ec:8126
    - RTL8127 (10GbE) — VID:DID = 10ec:8127

  Under Linux (kernel 6.19, which natively supports the RTL8127 since
  mainline 6.15), only the RTL8126 is visible. The RTL8127 controller
  does NOT appear in PCIe enumeration — it is not listed by lspci, not
  present in /sys/bus/pci/devices/, and never mentioned in the kernel
  boot log.

  This is therefore NOT a driver issue (the mainline r8127 driver is
  available and loadable), but a defect in the way the BIOS itself
  initializes / exposes the device.

  === Diagnostic evidence ===

  # Kernel
  $ uname -r
  6.19.5-2-pve

  # PCIe enumeration — search for any Realtek device
  $ lspci -nn | grep -i "10ec:"
  02:00.0 Ethernet controller [0200]: Realtek Semiconductor Co., Ltd.
          RTL8126 5GbE Controller [10ec:8126] (rev 01)

  # Explicit search for the RTL8127 in the entire PCIe tree
  $ lspci -nn -vv | grep -i "8127"
  (no occurrence — the device does not exist from the PCIe standpoint)

  # Loaded kernel modules
  $ lsmod | grep -E "r8126|r8127"
  r8126   204800   0

  Additional tests performed:
  - Disabled PCIe ASPM via kernel parameters (pcie_aspm=off,
    pcie_port_pm=off, pci=nommconf) → no effect, the device remains
    invisible.
  - Explicit modprobe of the r8127 module → the module loads but
    finds no device to bind to.

  === Request ===
  1. Official confirmation of this bug on the v1.01 BIOS of the N5 Air.
  2. Estimated release date for a corrected BIOS (v1.02 or later) that
     will properly expose the RTL8127 on the PCIe bus.
  3. If a beta version is available internally, I would be happy to
     test it and report back.

  To my knowledge, BIOS updates have been published for the N5 and N5
  Pro models, but none yet for the N5 Air as of 2026-04-28. Given the
  relative novelty of the model, I assume a release is in preparation.

  Thank you for your help. Please let me know if you need any further
  diagnostic information.
