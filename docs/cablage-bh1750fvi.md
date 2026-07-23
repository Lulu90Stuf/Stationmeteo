# Câblage — BH1750FVI (luminosité, I2C)

Même bus I2C-1 (pins 3/5) que le BME280 et le VEML6070 — les trois
peuvent cohabiter en fil volant sans se débrancher les uns les autres
(adresses différentes, pas de conflit).

## Le module

Breakout typique (GY-30 / GY-302) : `VCC`, `GND`, `SCL`, `SDA`, `ADDR`.
La broche `ADDR` sélectionne l'adresse I2C :

- `ADDR` à GND ou flottante (souvent tirée au bas en interne sur le
  PCB) → adresse **`0x23`** (défaut le plus courant)
- `ADDR` à VCC → adresse **`0x5C`**

Comme pour le BME280, **ne pas supposer** — câbler puis vérifier avec
un scan I2C, le comportement de la broche flottante dépend du module.

## Table de câblage

| BH1750FVI | Pi (broche physique) | Pi (nom) | Rôle |
|---|---|---|---|
| VCC | **Pin 1** | 3.3V | Alimentation |
| GND | **Pin 6** (ou 9, 14, 20...) | GND | Masse |
| SCL | **Pin 5** | GPIO3 (SCL1) | Horloge I2C — bus partagé |
| SDA | **Pin 3** | GPIO2 (SDA1) | Données I2C — bus partagé |
| ADDR | GND *(ou laisser flottant)* | GND | Fixe l'adresse à `0x23`. Relier à VCC à la place pour `0x5C` si tu préfères réserver `0x23` à un second module BH1750. |

```
        3V3  (1) (2)  5V
   BH1750-SDA→GPIO2  (3) (4)  5V
   BH1750-SCL→GPIO3  (5) (6)  GND ←BH1750-GND / ADDR
```

## Vérification

```sh
i2cdetect -y 1
```
Doit faire apparaître `23` (ou `5c` selon le câblage d'`ADDR`).

## Driver

Pas de protocole à écrire à la main ici — `amalgame-hardware-sensor`
fournit déjà `Bh1750(bus, addr)` / `.ReadLux()`, utilisé tel quel dans
[`../tests/bh1750/bh1750_test.am`](../tests/bh1750/bh1750_test.am).
