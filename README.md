# Station Météo — Raspberry Pi Zero + Amalgame

Station météo auto-hébergée : un Raspberry Pi Zero (+ clef Wi-Fi USB,
moins chère qu'un Zero W/2W) lit des capteurs en GPIO/I2C, écrit dans
une base SQLite locale, et un dashboard web (PWA) sert les données —
le tout écrit en [Amalgame](https://github.com/amalgame-lang/Amalgame)
et [Mosaic](https://github.com/amalgame-lang/mosaic).

Le développement/test se fait sur un Raspberry Pi 3 (I2C identique,
plus simple à manipuler) avant déploiement sur le Pi Zero cible.

## Architecture

```
┌─────────────┐   I2C / GPIO   ┌──────────────┐   SQLite (WAL)   ┌──────────────┐
│  Capteurs   │───────────────▶│   capture/   │─────────────────▶│  dashboard/  │
│ BME280, ... │                │  (amc, amc   │   station.db     │  (Mosaic —   │
└─────────────┘                │   service)   │◀── lecture seule ──   web + PWA)  │
                                └──────────────┘                  └──────────────┘
```

- **capture/** — service Amalgame qui poll les capteurs (I2C via
  `amalgame-hardware-gpio`, drivers via `amalgame-hardware-sensor`) et
  insère dans SQLite (`amalgame-database-sqlite`, base vendored, pas
  de lib système). `PRAGMA journal_mode=WAL` pour permettre des
  lectures concurrentes pendant l'écriture. Purge FIFO (fenêtre de
  rétention glissante + `incremental_vacuum` périodique) pour ne pas
  faire grossir la base indéfiniment sur la carte SD.
- **dashboard/** — application Mosaic (page dynamique + PWA :
  `manifest.webmanifest` + `sw.js`, patron déjà utilisé sur
  `stats.amalgame.me`) qui lit `station.db` en lecture seule et sert
  une API + une UI.

## Capteurs

| Capteur | Bus | Statut | Mesure |
|---|---|---|---|
| BME280 (breakout Bluedot) | I2C (`0x77`) | ✅ validé sur Pi 3 | Température, pression, humidité |
| BH1750FVI | I2C (`0x23`) | ✅ validé sur Pi 3 | Luminosité (lux) |
| VEML6070 | I2C (`0x38`/`0x39`) | ✅ validé sur Pi 3 | UV (comptage brut, pas encore calibré en indice UV) |
| MQ135 | Analogique → ADS1115 | À tester | Qualité de l'air (CO2 éq., gaz) |
| WHSPRG (pluviomètre à auget) | Impulsions (GPIO17, polling) | ✅ validé sur Pi 3 | Pluviométrie (comptage brut, mm/bascule à déterminer) |
| CYC-FX1-KV-A1 | À déterminer | À tester | — |
| Capteur de vent | — | Non commandé | Vitesse/direction du vent |

Certains capteurs analogiques (MQ135, potentiellement le capteur de
vent) passeront par un **ADS1115** (ADC 16 bits I2C) plutôt que
directement en GPIO — le Pi n'a pas d'entrée analogique native.

## Toolchain

- `amc` ≥ 0.8.73 (le driver `Bme280` en dépend) — voir
  [github.com/amalgame-lang/Amalgame](https://github.com/amalgame-lang/Amalgame)
- `mosaic` — voir [github.com/amalgame-lang/mosaic](https://github.com/amalgame-lang/mosaic)
- `libgpiod` **v2** (≥ 2.0) — Raspberry Pi OS Bookworm ne fournit que
  la 1.6 en apt (incompatible), compiler depuis les sources :
  ```sh
  curl -sSLO https://mirrors.edge.kernel.org/pub/software/libs/libgpiod/libgpiod-2.2.4.tar.gz
  tar xzf libgpiod-2.2.4.tar.gz && cd libgpiod-2.2.4
  ./configure --prefix=/usr/local --enable-tools=yes
  make && sudo make install && sudo ldconfig
  ```
- Packages Amalgame : `hardware-gpio`, `hardware-sensor`,
  `database-sqlite` (`amc package add hardware-gpio hardware-sensor sqlite`)

## Câblage

Voir [`docs/cablage-bme280.md`](docs/cablage-bme280.md).

## Tests matériels

Voir [`tests/`](tests/) — programmes Amalgame de validation capteur
par capteur, exécutés sur le Pi 3 avant intégration dans `capture/`.

## Historique / notes

- Un bug de compensation pression/humidité a été trouvé et corrigé
  dans le driver `Bme280` de `amalgame-hardware-sensor` (pression
  ~35% trop basse, humidité ~4× trop haute) — voir le commit
  `95a5714` sur ce package. Toujours purger
  `/tmp/amc-pkg-<Nom>.o` après avoir modifié un package en cours de
  développement : `amc build` ne l'invalide pas automatiquement.
