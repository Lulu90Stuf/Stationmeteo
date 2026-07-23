# Câblage — BME280 (breakout Bluedot, I2C)

Valable sur Raspberry Pi 3 (dev/test) et Raspberry Pi Zero (cible) —
même connecteur 40 broches, mêmes numéros de pin physiques.

## Le module

Le breakout Bluedot BME280 est un module **I2C/SPI combiné** (la puce
BME280 supporte nativement les deux protocoles). Broches disponibles
sur le PCB : `VCC`, `GND`, `SCK`, `SD0`, `SDI`, `CS`.

Contrairement à un breakout I2C-only classique, **deux broches
doivent être forcées à un niveau précis** pour sélectionner le mode
I2C — sinon la puce reste en mode SPI (ou flotte) et ne répond à rien
sur le bus.

## Table de câblage

| BME280 | Pi (broche physique) | Pi (nom) | Rôle |
|---|---|---|---|
| VCC | **Pin 1** | 3.3V | Alimentation — **3.3V uniquement**, pas le 5V (pin 2/4) |
| GND | **Pin 6** (ou 9, 14, 20, 25...) | GND | Masse |
| **CS** | **Pin 1** (relié à VCC) | 3.3V | ⚠️ CS **haut** → sélectionne le mode **I2C** (CS bas/flottant = mode SPI) |
| **SD0** (SDO) | GND *(câblage retenu ici)* | GND | ⚠️ Ne doit pas flotter — fixe l'adresse I2C : GND → `0x76`, VCC → `0x77`. **Sur notre module, SD0 est câblé côté VCC → adresse effective `0x77`** |
| SDI | **Pin 3** | GPIO2 (SDA1) | Données I2C (SDA) |
| SCK | **Pin 5** | GPIO3 (SCL1) | Horloge I2C (SCL) |

## Schéma (broches du connecteur 40 pins)

```
        3V3  (1) (2)  5V
   BME280-SDI→GPIO2  (3) (4)  5V
   BME280-SCK→GPIO3  (5) (6)  GND ←BME280-GND
                GPIO4  (7) (8)  GPIO14
                  GND  (9) (10) GPIO15
                  ...
```
(pin 1 = coin le plus proche du connecteur d'alimentation du Pi)

## Notes

- Pas de résistances de pull-up à ajouter : le Broadcom du Pi active
  des pull-ups internes sur GPIO2/3 dès que l'I2C est activé, et la
  plupart des breakouts BME280 en ont déjà.
- Bus utilisé : `/dev/i2c-1` (`new I2c(1)` côté Amalgame), le bus par
  défaut activé par `dtparam=i2c_arm=on`.
- **Adresse effective validée sur le matériel : `0x77`** (pas `0x76`,
  qui est l'adresse par défaut la plus courante sur d'autres modules —
  dépend uniquement du câblage de `SD0`). Vérifier avec `i2cdetect -y 1`
  ou le scan intégré dans [`tests/bme280/bme280_test.am`](../tests/bme280/bme280_test.am)
  avant de coder en dur une adresse.

## Activer l'I2C sur le Pi (une fois, avant tout câblage utile)

```sh
sudo raspi-config nonint do_i2c 0   # active l'interface
# nécessite généralement un reboot pour que le pinmux matériel prenne effet
sudo apt-get install -y i2c-tools
i2cdetect -y 1                      # doit lister le(s) capteur(s) câblé(s)
```
