# Console L-20R : ce qui est décodé (20 nov. 2026)

Sources lues : tecs/zoom-l20-control (MIT, maj 18 nov.), grischa2211/L20R-Controller-Performance-Mixer
(Apache-2.0, testé sur une L-20R firmware 1.13), PimDoos/zoom-L20-control (Apache-2.0),
LagZeppelin/livetrak-l20-web (doc `docs/protocol-l20-ble.md`, issue du PR #8 et suivants ; **aucune licence**
dans le dépôt, donc on n'en copie pas de code, on s'en sert seulement comme référence de faits).

## Décodé (utilisé dans l'app)

| Fonction | Comment | Source |
|---|---|---|
| Connexion | BLE-MIDI standard, nom `L-20…` | tous |
| Lire tout l'état | SysEx `F0 52 00 00 2B F7` → réponse `2A` de 656 octets | tous |
| **Noms des pistes (lire)** | dans l'état, 9 octets par piste à partir de l'octet 9 | tecs, PimDoos |
| **Noms des pistes (écrire)** | SysEx `F0 52 00 00 31 02 <canal 1-20> 09 00 <9 octets ASCII> F7`, 8 lettres max, sans accents | grischa (L-20R 1.13) |
| Volume, pan, mute, solo, phase | CC par canal (canal MIDI = piste ; 17/18 et 19/20 = CC+1 sur canal 0 et 2) | tous |
| EQ 3 bandes, fréquence des médiums, coupe-bas, EQ on/off | CC 14/1A/2C, 18, 2E, 0E | tecs, LagZeppelin |
| Envois FX1 / FX2 | CC 34 / 36 | tous |
| Mix de retour A à F (envoi par canal) | CC 3E…48 | tous |
| Volume de sortie A à F, Master, mute Master | `B0…B5 53`, `BA 54`, `B9 54` | tecs, LagZeppelin |
| Effets : type, 2 réglages, retour, mute | CC 4E, 50-52 | tecs, LagZeppelin |
| EQ graphique 15 bandes | CC 55 | tecs, LagZeppelin |
| Scènes 1 à 9 : rappeler / sauvegarder | « boutons » CC 56/57 valeur 03 : RECALL ou SAVE, puis le numéro | tecs |
| VU-mètres | SysEx `31 04`, ~15 par seconde | tous |
| Enregistreur (play/stop/rec) | CC 57 | tecs, LagZeppelin |

## Pas décodé

| Quoi | Note |
|---|---|
| Gain, compresseur, +48V, PAD | **analogique** : ne passe pas par Bluetooth. Même l'app iPad ne peut pas. |
| Couleur des pistes (écrire) | on la lit dans l'état, mais personne n'a trouvé comment l'écrire |
| Scènes 10 à 99 | la liste des noms se lit (`07 00`), mais rappeler/sauver au-delà de 9 = inconnu |
| Renommer une scène | inconnu |
| Navigateur de fichiers de l'enregistreur, menus | inconnu |
| Écriture des noms sur L-20R **autre firmware que 1.13** | à tester chez nous |

## Méthode pour décoder le reste

Image : la console et l'app iPad se parlent en code morse. Pour apprendre un mot, on enregistre la
conversation pendant qu'on fait **un seul geste**, puis on compare.

1. **Méthode facile (pas besoin de l'iPad)** : notre app a un onglet « Journal ». On bouge un réglage
   **sur la console** (ou dans l'app) et le journal montre les messages qui passent. Ça marche pour tout ce que la
   console renvoie toute seule.
2. **Méthode iPad (pour ce que seule l'app iPad sait faire, ex. : couleur, scènes 10+)** :
   - Sur un Mac avec Xcode : brancher l'iPad de Mathieu, installer le profil « Bluetooth logging » d'Apple sur
     l'iPad, faire les gestes un à la fois en notant l'heure, puis récupérer le fichier `.pklg` (sysdiagnose)
     et l'ouvrir dans Wireshark ou PacketLogger. C'est exactement ce que LagZeppelin a fait.
   - Sans Mac : un petit sniffer Bluetooth (ex. nRF52840 dongle, ~15 $) + Wireshark sur le PC de Tony.
3. Règle : **un geste à la fois**, noter l'heure et la valeur avant/après.
