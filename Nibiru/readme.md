# Nibiru (nibiru-itn-1) Node Kurulum

![ghgf](https://user-images.githubusercontent.com/107190154/221386540-dab23489-6e59-4df3-b818-75e1c16130d1.png)

### Sistem Gereksinimleri 

|CPU | RAM  | Disk  | 
|----|------|----------|
|   4| 16GB  | 1000GB    |

## [Resmi Doküman](https://nibiru.fi/docs/run-nodes/testnet/)

## Tarayıcı: https://explorer.ppnv.space/nibiru

## Rehberde hata ya da eksik varsa PR atın.

## Güncelleme ve kütüphane kurulumunu yapıyoruz.
```
sudo apt update && sudo apt upgrade -y && sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```
## Go kurulumu
```
ver="1.19.2"
cd $HOME
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile
source ~/.bash_profile
go version
```
## Nibid Kurulumu
```
curl -s https://get.nibiru.fi/@v0.19.2! | bash
```
## Versiyon kontrol edelim
> Versiyon 0.19.2 olmalı
```
nibid version
```
## Initialize işlemi
> NODEADINIZ yazan yere isminizi yazın.
```
nibid init NODEADINIZ --chain-id=nibiru-itn-1 --home $HOME/.nibid
```
## Genesis dosyası
```
NETWORK=nibiru-itn-1
curl -s https://networks.itn.nibiru.fi/$NETWORK/genesis > $HOME/.nibid/config/genesis.json
```
```
curl -s https://rpc.itn-1.nibiru.fi/genesis | jq -r .result.genesis > $HOME/.nibid/config/genesis.json
```
```
NETWORK=nibiru-itn-1
curl -s https://networks.itn.nibiru.fi/$NETWORK/genesis > $HOME/.nibid/config/genesis.json
```
```
curl -s https://rpc.itn-1.nibiru.fi/genesis | jq -r .result.genesis > $HOME/.nibid/config/genesis.json
```
## Seed ayarı
```
NETWORK=nibiru-itn-1
sed -i 's|seeds =.*|seeds = "'$(curl -s https://networks.itn.nibiru.fi/$NETWORK/seeds)'"|g' $HOME/.nibid/config/config.toml
```
# Minimum gas price ayarı
```
sed -i 's/minimum-gas-prices =.*/minimum-gas-prices = "0.025unibi"/g' $HOME/.nibid/config/app.toml
```

## Ağ ile daha hızlı senkron olmak için State Sync parametrelerini ayarlayın (isteğe bağlıdır ancak önerilir)
```
NETWORK=nibiru-itn-1
sed -i 's|enable =.*|enable = true|g' $HOME/.nibid/config/config.toml
sed -i 's|rpc_servers =.*|rpc_servers = "'$(curl -s https://networks.itn.nibiru.fi/$NETWORK/rpc_servers)'"|g' $HOME/.nibid/config/config.toml
sed -i 's|trust_height =.*|trust_height = "'$(curl -s https://networks.itn.nibiru.fi/$NETWORK/trust_height)'"|g' $HOME/.nibid/config/config.toml
sed -i 's|trust_hash =.*|trust_hash = "'$(curl -s https://networks.itn.nibiru.fi/$NETWORK/trust_hash)'"|g' $HOME/.nibid/config/config.toml
```
## Servis dosyamızı oluşturup başlatalım.
```
sudo tee /etc/systemd/system/nibid.service > /dev/null <<EOF
[Unit]
Description=Nibiru
After=network-online.target

[Service]
User=$USER
ExecStart=$(which nibid) start
Restart=always
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable nibid
sudo systemctl restart nibid && journalctl -fu nibid -o cat
```
## Log Kontrol
```
sudo journalctl -u nibid -f -o cat
```
## Sync Durumu
```
nibid status 2>&1 | jq .SyncInfo
```
## Cüzdan Oluşturma

## KEPLR'a yeni test ağını eklemek için önce eski nibiru-testnet-2 ağını kaldırın ve https://app.nibiru.fi/ buraya bağlanarak yeni test ağını ekleyin. Siteye bağlandığınızda oto olarak ekleniyor.

### Zaten daha önce kaydolduysanız burada kayıt olurken kullandığınız cüzdanı import edin kelimelerle.
```
nibid keys add CÜZDAN --recover
```
## Daha kayıt olmadıysanız kurulumu yaptıktan sonra oluşturduğunuz cüzdan adresi ile kaydolun.
> Kayıt :https://gleam.io/yW6Ho/nibiru-incentivized-testnet-registration
```
nibid keys add CÜZDAN
```

## Faucet ---> https://discord.gg/nibiru

<img width="840" alt="image" src="https://user-images.githubusercontent.com/107190154/221564563-8ed5bc8f-2dcc-467c-946f-495ef31c4d36.png">

## Validator Oluşturma
```
nibid tx staking create-validator \
--amount 10000000unibi \
--commission-max-change-rate "0.1" \
--commission-max-rate "0.20" \
--commission-rate "0.1" \
--min-self-delegation "1" \
--pubkey=$(nibid tendermint show-validator) \
--moniker NODEADINIZ \
--chain-id nibiru-itn-1 \
--gas-prices 0.025unibi \
--from CÜZDANADINIZ
```

# ADIM-2

## Pricefeeder ayarlaması
```
curl -s https://get.nibiru.fi/pricefeeder! | bash
```
## Değişiklik yapmadan direkt yapıştırın terminale.
```
export CHAIN_ID="nibiru-itn-1"
```
## Bu komutu da direkt girin.
```
nibid keys add pricefeeder
```
## Cüzdan çıktısındaki kelimeleri tek tırnak arasına kopyalayın, tırnakları silmeyin ve ardından terminale yapıştırın.
```
export FEEDER_MNEMONIC='BURAYAKELİMELERİKOPYALAYIN'
```
## Servis dosyamızı oluşturalım.
> Tek seferde girin komutu
```
sudo tee /etc/systemd/system/pricefeeder.service > /dev/null <<EOF
[Unit]
Description=pricefeeder
Requires=network-online.target
After=network-online.target

[Service]
Type=exec
User=$USER
ExecStart=/usr/local/bin/pricefeeder start
Restart=on-failure
ExecReload=/bin/kill -HUP $MAINPID
KillSignal=SIGTERM
PermissionsStartOnly=true
LimitNOFILE=65535
Environment=CHAIN_ID='$CHAIN_ID'
Environment=GRPC_ENDOPINT='$GRPC_ENDOPINT'
Environment=WEB_ENDPOINT='$WEB_ENDPOINT'
Environment=EXCHANGE_SYMBOLS_MAP='$EXCHANGE_SYMBOLS_MAP'
Environment=FEEDER_MNEMONIC='$FEEDER_MNEMONIC'
Environment=VALIDATOR_ADDRESS='$VALIDATOR_ADDRESS'

[Install]
WantedBy=multi-user.target
EOF
```
## Başlatalım
```
sudo systemctl daemon-reload && /
sudo systemctl enable pricefeeder && /
sudo systemctl start pricefeeder
```
## Kontrol edelim
```
journalctl -fu pricefeeder
```
## Validator Düzenleme
```
nibid tx staking edit-validator \
--moniker=NODEADINIZ \
--identity=<KEYBASE ID> \
--website="<WEBSİTENİZ>" \
--details="<AÇIKLAMA>" \
--chain-id nibiru-itn-1 \
--from=CÜZDANADINIZ
```

## Node Silme Komutları
```
sudo systemctl stop nibid
sudo systemctl disable nibid
sudo rm /etc/systemd/system/nibi* -rf
sudo rm $(which nibid) -rf
sudo rm $HOME/.nibid* -rf
sudo rm $HOME/nibiru -rf
sed -i '/NIBIRU_/d' ~/.bash_profile
```
