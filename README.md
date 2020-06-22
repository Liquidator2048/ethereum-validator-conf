# install

### Perchè systemd invece di docker-compose ?

Il motivo è che ho iniziato ad usare docker quando docker-compose ancora non era adatto all'utilizzo in produzione

I motivi per cui non ho mai sostituito l'utilizzo degli unit di systemd con docker-compose sono:

* gestione uniforme dei servizi installati su un server: tutti i servizi possono essere gestiti con systemctl sia che girano in un docker sia che siano installati

* i docker vengono aggiornati automaticamente ad ogni avvio

* i file all'interno del docker che non sono su volumi montati vengono eliminati al riavvio del servizio


## 0. docker

* installare docker! :)

* copiare il file `etc/default/docker-services`

per ora non è necessario modificarlo.

Verrà usata la directory `/srv/docker` per la persistenza dei dati dei docker


## 1. beacon node 

* copiare i files:

- `etc/systemd/service/docker-prysm-beacon.service` -> `/etc/systemd/service/docker-prysm-beacon.service`

- `etc/default/docker-prysm-beacon` -> `etc/default/docker-prysm-beacon` : se si vuole settare manulamente l'ip pubblico della macchina

* avviare il nodo

```bash
systemctl daemon-reload
systemctl start docker-prysm-beacon.service
```

reference:

[Connecting to the testnet: running a beacon node](https://docs.prylabs.network/docs/install/lin/docker#connecting-to-the-testnet-running-a-beacon-node)


## 2. validator

* copiare i files:

- `etc/systemd/service/docker-prysm-validator.service` -> `/etc/systemd/service/docker-prysm-validator.service`

- `etc/default/docker-prysm-validator` -> `/etc/default/docker-prysm-validator` : modificare la password!


* generare le chiavi per il validator

reference: [Generating a validator keypair](https://docs.prylabs.network/docs/install/lin/activating-a-validator#step-3a-generating-a-validator-keypair)

```bash
source /etc/default/docker-prysm-validator

docker run --rm -it \
   -v /srv/docker/prysm-validator/data:/data \
   gcr.io/prysmaticlabs/prysm/validator:latest \
   accounts create --keystore-path=/data --password="${KEYSTORE_PASSPHRASE}" | tee transaction-data.txt
```

* avviare il nodo validatore

```bash
systemctl daemon-reload
systemctl start docker-prysm-validator.service
```

## 3. attivare il validator

### importante: attende che il beacon sia sincronizzato prima di effettuare il deposito sullo smart contract

per verificare lo stato del beacon e del validator è possibile usare i seguenti comandi:

beacon node:
```
journalctl -fa -u docker-prysm-beacon.service


validator:
```
journalctl -fa -u docker-prysm-validator.service
```


* Ottenere gli ETH sulla testnet ed inviarli allo smart contract allegando i dati presenti nel file `transaction-data.txt`. 
  La procedura guidata è disponibile [qui](https://prylabs.net/participate).


## 4. monitoraggio

[...]