# Câblage — WHSPRG (pluviomètre à auget basculant)

Contrairement aux capteurs précédents (I2C), le pluviomètre à auget
est un simple **interrupteur reed magnétique** : chaque bascule de
l'auget (remplie par une quantité d'eau fixe) fait passer un aimant
devant le reed switch, qui ferme brièvement le circuit. Ce n'est donc
pas du I2C mais du **GPIO numérique avec détection de front**
(edge detection), déjà supporté en natif par `amalgame-hardware-gpio`
(Phase 2 — `Gpio.WatchEdge` / `WaitEdge` / `PollEdges`).

## Le module

2 fils seulement (le reed switch n'a pas de polarité, ni
d'alimentation à fournir — c'est un contact sec) :

- Fil 1 → une broche GPIO configurée en entrée avec pull-up interne
- Fil 2 → GND

Au repos le contact est ouvert (pin tirée au HIGH par le pull-up
interne). À chaque bascule, le reed switch ferme brièvement le
circuit → la pin passe à LOW → front descendant détecté.

## Table de câblage

| WHSPRG | Pi (broche physique) | Pi (nom) | Rôle |
|---|---|---|---|
| Fil signal | **Pin 11** | GPIO17 | Entrée, pull-up interne activé côté logiciel |
| Fil masse | **Pin 6** (ou 9, 14, 20...) | GND | Masse |

```
        3V3  (1) (2)  5V
              (3) (4)  5V
              (5) (6)  GND ←WHSPRG-masse
              (7) (8)
              (9) (10)
   WHSPRG-signal→GPIO17  (11) (12)
```

GPIO17 est libre (pas utilisée par les capteurs I2C sur pins 3/5) et
c'est la pin utilisée dans l'exemple `blink.am` du package — cohérent
avec le reste de la doc `hardware-gpio`. Change de pin si tu en as
déjà une occupée à cet endroit sur ton montage.

## Code (aperçu — driver à écrire, pas encore de classe dédiée)

```amalgame
import Amalgame.Hardware

let pin: int = 17
Gpio.PinMode(pin, PinMode.InputPullup)
Gpio.WatchEdge(pin, Edge.Falling)

// non-bloquant, à appeler en boucle dans le service capture :
let events: List<GpioEvent> = Gpio.PollEdges()
// events.Count() = nombre de bascules détectées depuis le dernier appel
```

## Points d'attention

- **Rebond de contact** : les reed switches peuvent rebondir
  (plusieurs fronts pour une seule bascule physique). Si les tests
  montrent des comptages en rafale improbables, ajouter un
  anti-rebond logiciel (ignorer les fronts trop rapprochés, ex. <
  50-100ms d'écart) — à valider empiriquement sur le matériel réel
  plutôt que deviner une constante.
- **Constante mm/bascule** : chaque modèle de pluviomètre a son propre
  volume d'auget (donc son propre mm de pluie par bascule). Je n'ai
  pas de fiche technique fiable pour le "WHSPRG" précis en main — à
  vérifier sur la doc/page produit avant de coder la conversion
  bascules → mm dans `capture/`. Le test matériel se contentera de
  compter les bascules brutes.
