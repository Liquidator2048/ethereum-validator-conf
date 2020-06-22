# install

### Perchè systemd invece di docker-compose ?

Il motivo è che ho iniziato ad usare docker quando docker-compose ancora non era adatto all'utilizzo in produzione

I motivi per cui non ho mai sostituito l'utilizzo degli unit di systemd con docker-compose sono:

* gestione uniforme dei servizi installati su un server: tutti i servizi possono essere gestiti con i comandi di *systemd*

* i docker vengono aggiornati automaticamente ad ogni avvio

* i file all'interno del docker che non sono su volumi montati vengono eliminati al riavvio del servizio


## 0. docker

* installare docker! :)

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
```

* copiare il file `etc/default/docker-services` -> `/etc/default/docker-services`

per ora non è necessario modificarlo.

Verrà usata la directory `/srv/docker` per la persistenza dei dati dei docker


## 1. beacon node 

* copiare i files:

- `etc/systemd/service/docker-prysm-beacon.service` -> `/etc/systemd/service/docker-prysm-beacon.service`

- `etc/default/docker-prysm-beacon` -> `etc/default/docker-prysm-beacon` : se si vuole settare manulamente l'ip pubblico della macchina

* avviare il nodo

```bash
# reload .service files
systemctl daemon-reload
# start
systemctl start docker-prysm-beacon.service
# start at boot
systemctl enable docker-prysm-beacon.service
```

reference: [Connecting to the testnet: running a beacon node](https://docs.prylabs.network/docs/install/lin/docker#connecting-to-the-testnet-running-a-beacon-node)


## 2. validator

* copiare i files:

- `etc/systemd/service/docker-prysm-validator.service` -> `/etc/systemd/service/docker-prysm-validator.service`

- `etc/default/docker-prysm-validator` -> `/etc/default/docker-prysm-validator` : modificare la password!


* generare le chiavi per il validator

reference: [Generating a validator keypair](https://docs.prylabs.network/docs/install/lin/activating-a-validator#step-3a-generating-a-validator-keypair)

```bash
source /etc/default/docker-prysm-validator

docker run --rm -it \
   -v /srv/docker/prysm-validator/secrets:/secrets \
   gcr.io/prysmaticlabs/prysm/validator:latest \
   accounts create --keystore-path=/secrets --password="${KEYSTORE_PASSPHRASE}" | tee transaction-data.txt
```

* avviare il nodo validatore

```bash
# reload .service files
systemctl daemon-reload
# start
systemctl start docker-prysm-validator.service
# start at boot
systemctl enable docker-prysm-validator.service
```


## 3. attivare il validator

* attende che il beacon node sia sincronizzato prima di effettuare il deposito sullo smart contract

per verificare lo stato del beacon node e del validator è possibile usare i seguenti comandi:

log beacon node:

```
journalctl -fa -u docker-prysm-beacon.service
```

log validator:

```
journalctl -fa -u docker-prysm-validator.service
```


* Ottenere gli ETH sulla testnet ed inviarli allo smart contract allegando i dati presenti nel file `transaction-data.txt`. 
  La procedura guidata è disponibile visitando [https://prylabs.net/participate](https://prylabs.net/participate).


## 4. Slasher

* copiare i files:

- `etc/systemd/service/docker-prysm-slasher.service` -> `/etc/systemd/service/docker-prysm-slasher.service`

[...]


```bash
# reload .service files
systemctl daemon-reload
# start
systemctl start docker-prysm-slasher.service
# start at boot
systemctl enable docker-prysm-slasher.service
```

reference: [Running a slasher](https://docs.prylabs.network/docs/prysm-usage/slasher)

## 5. monitoraggio

[...]

reference: [Monitoring and alerts with Grafana](https://docs.prylabs.network/docs/prysm-usage/monitoring/grafana-dashboard)