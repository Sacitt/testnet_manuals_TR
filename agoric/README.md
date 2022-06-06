###Sacitt

# devnet için agoric düğüm kurulumu — agoricdev-11

Official documentation:
> [Validator Guide for Devnet](https://github.com/Agoric/agoric-sdk/wiki/Validator-Guide-for-Devnet)

Explorer:
> https://devnet.explorer.agoric.net/

## Kullanışlı araçlar ve referanslar
> To set up monitoring for your validator node navigate to [Set up monitoring and alerting for agoric validator](https://github.com/kj89/testnet_manuals/blob/main/agoric/monitoring/README.md)
>
> To migrate your valitorator to another machine read [Migrate your validator to another machine](https://github.com/kj89/testnet_manuals/blob/main/agoric/migrate_validator.md)

## Agoric tam düğümünüzü kurun
### Seçenek 1 (otomatik)
Aşağıdaki otomatik komut dosyasını kullanarak agoric tam düğümünüzü birkaç dakika içinde kurabilirsiniz. Doğrulayıcı düğüm adınızı girmenizi isteyecektir!
```
wget -O agoric_devnet.sh https://raw.githubusercontent.com/kj89/testnet_manuals/main/agoric/agoric_devnet.sh && chmod +x agoric_devnet.sh && ./agoric_devnet.sh
```

### Seçenek 2 (elle kurulum)
Düğümü manuel olarak kurmayı tercih ederseniz , [manuel kılavuzu](https://github.com/kj89/testnet_manuals/blob/main/agoric/manual_devnet_install.md) takip edebilirsiniz .

### Yükleme sonrası
Kurulum bittiğinde lütfen değişkenleri sisteme yükleyin
```
source $HOME/.bash_profile
```

Ardından, doğrulayıcınızın blokları senkronize ettiğinden emin olmalısınız. Senkronizasyon durumunu kontrol etmek için aşağıdaki komutu kullanabilirsiniz.
```
agd status 2>&1 | jq .SyncInfo
```

### Cüzdan oluştur
Yeni cüzdan oluşturmak için aşağıdaki komutu kullanabilirsiniz. Hatırlatıcı kelimeleri kaydetmeyi unutmayın!
```
agd keys add $WALLET
```

(İSTEĞE BAĞLI) Cüzdanınızı tohum cümlesi kullanarak kurtarmak için
```
agd keys add $WALLET --recover
```

Mevcut cüzdan listesini almak için
```
agd keys list
```

### Cüzdan bilgilerini kaydet
Cüzdan adresi ekle
```
WALLET_ADDRESS=$(agd keys show $WALLET -a)
```

valoper adresi ekle
```
VALOPER_ADDRESS=$(agd keys show $WALLET --bech val -a)
```

Değişkenleri sisteme yükle
```
echo 'export WALLET_ADDRESS='${WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export VALOPER_ADDRESS='${VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### Musluğu kullanarak cüzdan bakiyenizi doldurun
agoricdev-11 testnet'te ücretsiz jeton almak için:
* gidin[Agoric official discord](https://agoric.com/discord)
* DEVELOPMENT kanalını aç kategori lerdeki  #faucet kanalına gir 
* yükleme emri: `!faucet client <WALLET_ADDRESS>`

### Doğrulayıcı oluştur
Doğrulayıcı oluşturmadan önce lütfen en az 1 bld'ye sahip olduğunuzdan (1 bld 1000000 ubld'ye eşittir) ve düğümünüzün senkronize edildiğinden emin olun.

Cüzdan bakiyenizi kontrol etmek için:
```
agd query bank balances $WALLET_ADDRESS
```
> Cüzdanınız muhtemelen daha fazla bakiye göstermiyorsa, düğümünüz hala eşitleniyordur. Lütfen senkronizasyonun bitmesini bekleyin ve ardından devam edin

Aşağıdaki doğrulayıcı çalıştırma komutunuzu oluşturmak için
```
agd tx staking create-validator \
  --amount 1000000ubld \
  --from $WALLET \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --commission-rate "0.07" \
  --min-self-delegation "1" \
  --pubkey  $(agd show-validator) \
  --moniker $NODENAME \
  --chain-id $CHAIN_ID
```

## Güvenlik
Anahtarlarınızı korumak için lütfen temel güvenlik kurallarına uyduğunuzdan emin olun.

### Temel Güvenlik Duvarı güvenliği
ufw'nin durumunu kontrol ederek başlayın.
```
sudo ufw status
```

Varsayılanı, giden bağlantılara izin verecek, ssh ve 26656 hariç tüm gelenleri reddedecek şekilde ayarlar. SSH oturum açma girişimlerini sınırlayın
```
sudo ufw default allow outgoing
sudo ufw default deny incoming
sudo ufw allow ssh/tcp
sudo ufw limit ssh/tcp
sudo ufw allow 26656,26660,26657,1317/tcp
sudo ufw enable
```

## Monitoring
Doğrulayıcı sağlık durumunuzu izlemek ve bu konuda uyarı almak için, [izleme ve agorik doğrulayıcı için uyarı ayarlama konusundaki kılavuzumu kullanabilirsiniz.](https://github.com/kj89/testnet_manuals/blob/main/agoric/monitoring/README.md)

## Senkronizasyon süresini hesaplayın
Bu komut dosyası, düğümünüzü tam olarak senkronize etmenin ne kadar zaman alacağını tahmin etmenize yardımcı olacaktır/
5 dakikalık bir süre boyunca senkronize edilen dakika başına ortalama blokları ölçer ve ardından size sonuçlar verir.
```
wget -O synctime.py https://raw.githubusercontent.com/kj89/testnet_manuals/main/agoric/tools/synctime.py && python3 ./synctime.py
```

## Şu anda bağlı olan eşler listesini kimlikleri ile alın
```
curl -sS http://localhost:26657/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}'
```

## Faydalı komutlar
### Servis Yönetimi
Günlükleri kontrol et
```
journalctl -fu agd -o cat
```

Hizmeti başlat
```
systemctl start agd
```

Hizmeti durdur
```
systemctl stop agd
```

Servisi yeniden başlat
```
systemctl restart agd
```

### Düğüm bilgisi
Senkronizasyon bilgisi
```
agd status 2>&1 | jq .SyncInfo
```

Validator bilgisi
```
agd status 2>&1 | jq .ValidatorInfo
```

düğüm bilgisi
```
agd status 2>&1 | jq .NodeInfo
```

Düğüm kimliğini göster
```
agd show-node-id
```

### Cüzdan işlemleri
cüzdan listesi
```
agd keys list
```

Cüzdanı kurtar ( mevcut bir cüzdanınız var ise bu kod ile yülkeyebilirsiniz )
```
agd keys add $WALLET --recover
```

Bu kod ile mevcut cüzdanları silebilirsiniz.
```
agd keys delete $WALLET
```

cüzdan bakiyesini sorgulama $wallet_ address yazan yere kendi cüzdan adresinizi yazın
```
agd query bank balances $WALLET_ADDRESS
```

para transferi <to_wallet_address> yazan yere cüzdan adresinizi yazınız
```
agd tx bank send $WALLET_ADDRESS <TO_WALLET_ADDRESS> 10000000ubld
```

### oylama
```
agd tx gov vote 1 yes --from $WALLET --chain-id=$CHAIN_ID
```

### Stake, Delegasyon ve Ödüller
Delege hissesi
```
agd tx staking delegate $VALOPER_ADDRESS 10000000ubld --from=$WALLET --chain-id=$CHAIN_ID --gas=auto
```

Payını doğrulayıcıdan başka bir doğrulayıcıya yeniden devretme
```
agd tx staking redelegate <srcValidatorAddress> <destValidatorAddress> 10000000ubld --from=$WALLET --chain-id=$CHAIN_ID --gas=auto
```

Tüm ödülleri geri çek
```
agd tx distribution withdraw-all-rewards --from=$WALLET --chain-id=$CHAIN_ID --gas=auto
```

Komisyon ile ödülleri geri çekin
```
agd tx distribution withdraw-rewards $VALOPER_ADDRESS --from=$WALLET --commission --chain-id=$CHAIN_ID
```

### Doğrulayıcı yönetimi
Doğrulayıcıyı düzenle
```
agd tx staking edit-validator \
--moniker=$NODENAME \
--identity=1C5ACD2EEF363C3A \
--website="http://kjnodes.com" \
--details="Providing professional staking services with high performance and availability. Find me at Discord: kjnodes#8455 and Telegram: @kjnodes" \
--chain-id=$CHAIN_ID \
--from=$WALLET
```

Hapisten çıkma doğrulayıcısı
```
agd tx slashing unjail \
  --broadcast-mode=block \
  --from=$WALLET \
  --chain-id=$CHAIN_ID \
  --gas=auto
```
