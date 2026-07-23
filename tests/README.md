# Tests matériels

Programmes Amalgame de validation, un dossier par capteur. Compilés et
exécutés directement sur le Pi (3 pour le dev, Zero pour la cible) —
pas de mock, ce sont des tests contre le vrai matériel en I2C/GPIO.

## bme280/

- `bme280_test.am` — scan I2C + lecture température/pression/humidité.
  Validé sur Pi 3B+ avec un breakout Bluedot BME280 : 23.83°C /
  967.4 hPa / 49.1% RH (câblage : [`../docs/cablage-bme280.md`](../docs/cablage-bme280.md)).
- `divshift_semantics.am` — diagnostic de sémantique du langage
  (troncature `/` vs floor `>>`) qui a mené à la découverte d'un bug
  de compensation dans `amalgame-hardware-sensor` (corrigé en amont,
  commit `95a5714`).

## Build générique

```sh
amc package add hardware-gpio hardware-sensor
amc build <fichier>.am -o <nom>
sudo ./<nom>       # ou groupes gpio/i2c/spi
```

## Piège connu

`amc` cache les objets de package compilés dans `/tmp/amc-pkg-<Nom>.o`
— un chemin **fixe, pas lié au projet**. Si tu modifies un package
localement (ex. `~/.amalgame/packages/.../facade.am`) pendant le dev,
`amc build` ne détecte pas toujours le changement : purge le cache
avant de rebuilder :

```sh
rm -f /tmp/amc-pkg-<Nom>.o /tmp/amc-pkg-<Nom>.o.c
```
