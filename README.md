# Telegram
- https://t.me/HerculesNode
# Union node kurulumu (testnet-5)
## Sistem gereksinimleri
 - Ubuntu 22.04+
 - 4-8 GB RAM
 - 4 vCPUs
 - 64 GB storage
 - Portlar : 26657 1317 9093
 - [Resmi kaynak](https://union.build/docs/joining-testnet/getting-started/)

## Sistem günceleme ve gerekli paketlerin kurulumu

```
sudo apt update && sudo apt upgrade -y
sudo apt install git make curl screen tar wget jq build-essential -y 
sudo apt install curl tar wget clang pkg-config protobuf-compiler libssl-dev jq build-essential protobuf-compiler bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
rm $HOME/get-docker.sh
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

## Değişkenler

```
echo 'export CHAIN_ID=union-testnet-5' >> ~/.bashrc
echo 'export MONIKER="moniker_adınız"' >> ~/.bashrc
echo 'export KEY_NAME=cuzdan_adi' >> ~/.bashrc
echo 'export GENESIS_URL=union.testnet.4.seed.poisonphang.com:26657/genesis' >> ~/.bashrc
echo 'export UNIOND_VERSION="v0.18.0"' >> ~/.bashrc
echo 'alias uniond='docker run -v ~/.union:/.union --network host -it ghcr.io/unionlabs/uniond:v0.18.0 --home /.union'' >> ~/.bashrc
source ~/.bashrc
```

## Yükleme
```
docker pull ghcr.io/unionlabs/uniond-release:$UNIOND_VERSION
mkdir ~/.union
```
> Aşağıdakiler tek komut komple kopyalayıp yapıştıralım.
```
docker run \
  --user $(id -u):$(id -g) \
  --volume ~/.union:/.union \
  --volume /tmp:/tmp \
  -it ghcr.io/unionlabs/uniond-release:$UNIOND_VERSION init $MONIKER bn254 \
  --home /.union
```
```
curl $GENESIS_URL | jq '.result.genesis' > ~/.union/config/genesis.json
```
```
SEEDS="daa56ac6573c6427ecef438d4c248d72b206597b@union.testnet.4.seed.poisonphang.com:26656"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/" $HOME/.union/config/config.toml
```

## Çalıştırma
```
cd ~/.union
nano compose.yaml
```
> Alttakileri aynen kopyalayıp ctrl+x ile kaydedip çıkın
```
services:
  node:
    image: ghcr.io/unionlabs/uniond-release:${UNIOND_VERSION}
    volumes:
      - ~/.union:/.union
      - /tmp:/tmp
    network_mode: "host"
    restart: unless-stopped
    command: start --home /.union
```
```
docker compose up -d
docker update --restart=unless-stopped union-node
```
```
screen -S uni
docker logs -f root-node-1
#Loglar akıyorsa. ctrl+a+d ile çıkılablir
```

## Cüzdan oluşturma
```
uniond keys add $KEY_NAME
```
> Adres :
```
uniond keys show $KEY_NAME --address
```
> Validator adres:
```
uniond keys show $KEY_NAME --bech=val --address
```
> Test tokeni temin ettiğinizde validator oluşturma
```
uniond tx staking create-validator \
  --amount 1000000muno \
  --pubkey $(uniond tendermint show-validator) \
  --moniker $MONIKER \
  --chain-id $UNIOND_VERSION \
  --from $KEY_NAME \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.20" \
  --commission-rate "0.1" \
  --min-self-delegation "1"
```







