# Câblage — VEML6070 (capteur UV, I2C)

Comme le BME280 : même connecteur 40 broches sur Pi 3 (dev) et Pi Zero
(cible), même bus I2C-1 partagé.

## Le module

Le VEML6070 (Vishay) est plus simple que le BME280 côté câblage : pas
de broche de sélection de protocole ni d'adresse à choisir, l'adresse
I2C est **fixée en silicium**. La plupart des breakouts (type
GY-VEML6070 / DFRobot) n'exposent que 4 broches : `VCC`, `GND`, `SCL`,
`SDA` (parfois une 5e broche `ACK`/`INT`, optionnelle, à laisser en
l'air pour une lecture par polling — pas besoin d'interruption).

## Table de câblage

| VEML6070 | Pi (broche physique) | Pi (nom) | Rôle |
|---|---|---|---|
| VCC | **Pin 1** | 3.3V | Alimentation (la plupart des breakouts acceptent 3.3–5V, mais reste sur 3.3V pour matcher la logique I2C du Pi sans te poser de question de compatibilité) |
| GND | **Pin 6** (ou 9, 14, 20...) | GND | Masse |
| SCL | **Pin 5** | GPIO3 (SCL1) | Horloge I2C — **même bus que le BME280** |
| SDA | **Pin 3** | GPIO2 (SDA1) | Données I2C — **même bus que le BME280** |
| ACK / INT *(si présente)* | — non câblée | — | Optionnelle, utile seulement en mode interruption. Laisse-la flottante pour une lecture par polling simple. |

```
        3V3  (1) (2)  5V
   VEML-SDA→GPIO2   (3) (4)  5V
   VEML-SCL→GPIO3   (5) (6)  GND ←VEML-GND
```

## Partage du bus avec le BME280

Le VEML6070 utilise deux adresses I2C fixes : **`0x38`** (écriture du
registre de commande + lecture de l'octet bas) et **`0x39`** (lecture
de l'octet haut). Aucun conflit avec le BME280 (`0x77`) — les deux
capteurs peuvent cohabiter sur le même bus `SDA`/`SCL` en parallèle
(mêmes pins 3/5), pas besoin de les débrancher l'un pour tester
l'autre si tu préfères les laisser tous les deux en fil volant.

## Vérification

```sh
i2cdetect -y 1
```
Doit faire apparaître `38` et `39` dans la grille (le VEML6070 n'a pas
de registre d'identification comme le BME280 — `i2cdetect` montre
juste qu'un périphérique ACK à ces deux adresses, ça ne confirme pas
le modèle exact, seulement le câblage).
