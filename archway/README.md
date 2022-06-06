<p style="font-size:14px" align="right">
Join our telegram <a href="https://t.me/sacithbn" target="_blank"><img src="https://user-images.githubusercontent.com/50621007/168689534-796f181e-3e4c-43a5-8183-9888fc92cfa7.png" width="30"/></a>

<p align="center">
  <img width="100" height="auto" src="https://user-images.githubusercontent.com/50621007/164164767-0a9590e5-b018-44de-8a3e-4ebdd905dfbc.png">
</p>

# Archway node setup for Incentivized Testnet — Torii-1

## Archway fullnode'unuzu kurun
### Seçenek 1 (otomatik)
Aşağıdaki otomatik komut dosyasını kullanarak Archway fullnode'unuzu birkaç dakika içinde kurabilirsiniz. Doğrulayıcı düğüm adınızı girmenizi isteyecektir!
```
wget -O archway.sh https://raw.githubusercontent.com/kj89/testnet_manuals/main/archway/archway.sh && chmod +x archway.sh && ./archway.sh
```

### Seçenek 2 (manuel)
[manual kurulum](https://github.com/kj89/testnet_manuals/blob/main/archway/manual_install.md) u tıklarsanız. manuel kurulum sayfasına yönlendirilirsiniz.

### Yükleme sonrası
Kurulum bittiğinde lütfen değişkenleri sisteme yükleyin
```
source $HOME/.bash_profile
```

Ardından, doğrulayıcınızın blokları senkronize ettiğinden emin olmalısınız. Senkronizasyon durumunu kontrol etmek için aşağıdaki komutu kullanabilirsiniz.
```
archwayd status 2>&1 | jq .SyncInfo
```

### Cüzdan oluştur
Yeni cüzdan oluşturmak için aşağıdaki komutu kullanabilirsiniz. memonicleri kaydetmeyi unutmayın!
```
archwayd keys add $WALLET
```

(İSTEĞE BAĞLI) Cüzdanınızı tohum cümlesi kullanarak kurtarmak için
```
archwayd keys add $WALLET --recover
```

Mevcut cüzdan listesini almak için
```
archwayd keys list
```

### Cüzdan bilgilerini kaydet
Cüzdan adresi ekle
```
WALLET_ADDRESS=$(archwayd keys show $WALLET -a)
```

valoper adresi ekle
```
VALOPER_ADDRESS=$(archwayd keys show $WALLET --bech val -a)
```

Değişkenleri sisteme yükle
```
echo 'export WALLET_ADDRESS='${WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export VALOPER_ADDRESS='${VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### Musluğu kullanarak cüzdan bakiyenizi doldurun
Şu anda torii-1 testnet'te jeton almak için iki seçenek var

1. Twitter hesabınızı kullanarak 3 torii alın
* bu adrese gidin https://stakely.io/en/faucet/archway-testnet
* cüzdan adresinizi girin ve doğrula'ya tıklayın
* ardından tweet düğmesine basın ve musluğun tweetinizi işlemesini bekleyin
> Lütfen unutmayın: İstenmeyen hesaplara karşı önlem almak için Stakely musluğu, Twitter hesapları çok az etkinliğe sahip veya çok yeni olan kullanıcılardan gelen istekleri reddedecektir!

2. Twitter hesabınızı kullanarak 3 torii alın
* bu adrese gidin https://faucet.torii-1.archway.tech/ui/
* cüzdan adresinizi girin ve tıklayın `Get Fund`

### Doğrulayıcı oluştur
Doğrulayıcı oluşturmadan önce lütfen en az 1 torii'ye sahip olduğunuzdan (1 torii 1000000 utorii'ye eşittir) ve düğümünüzün senkronize edildiğinden emin olun.

Cüzdan bakiyenizi kontrol etmek için:  $WALLET_ADDRESS ifedesinin yerine kendi cüzdan adresinizi yazın!
```
archwayd query bank balances $WALLET_ADDRESS
```
> Cüzdanınız muhtemelen daha fazla bakiye göstermiyorsa, düğümünüz hala eşitleniyordur. Lütfen senkronizasyonun bitmesini bekleyin ve ardından devam edin 

Aşağıdaki doğrulayıcı çalıştırma komutunuzu oluşturmak için
```
archwayd tx staking create-validator \
  --amount 1000000utorii \
  --from $WALLET \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --commission-rate "0.07" \
  --min-self-delegation "1" \
  --pubkey  $(archwayd tendermint show-validator) \
  --moniker $NODENAME \
  --chain-id $CHAIN_ID
```

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
sudo ufw allow 26656,26660/tcp
sudo ufw enable
```

## Faydalı komutlar
### Servis Yönetimi
Günlükleri kontrol et
```
journalctl -fu archwayd -o cat
```

Hizmeti başlat
```
systemctl start archwayd
```

Hizmeti durdur
```
systemctl stop archwayd
```

Servisi yeniden başlat
```
systemctl restart archwayd
```

### Düğüm bilgisi
Senkronizasyon bilgisi
```
archwayd status 2>&1 | jq .SyncInfo
```

Doğrulayıcı bilgisi
```
archwayd status 2>&1 | jq .ValidatorInfo
```

Düğüm bilgisi
```
archwayd status 2>&1 | jq .NodeInfo
```

Düğüm kimliğini göster
```
archwayd tendermint show-node-id
```

### Cüzdan işlemleri
cüzdan listesi
```
archwayd keys list
```

Cüzdanı kurtar
```
archwayd keys add $WALLET --recover
```

Cüzdanı sil
```
archwayd keys delete $WALLET
```

Cüzdan bakiyesi alın
```
archwayd query bank balances $WALLET_ADDRESS
```

para transferi
```
archwayd tx bank send $WALLET_ADDRESS <TO_WALLET_ADDRESS> 10000000utorii
```

### Stake, Delegasyon ve Ödüller
Delege hissesi
```
archwayd tx staking delegate $VALOPER_ADDRESS 10000000utorii --from=$WALLET --chain-id=$CHAIN_ID --gas=auto
```

Payını doğrulayıcıdan başka bir doğrulayıcıya yeniden devretme
```
archwayd tx staking redelegate <srcValidatorAddress> <destValidatorAddress> 10000000utorii --from=$WALLET --chain-id=$CHAIN_ID --gas=auto
```

Tüm ödülleri geri çek
```
archwayd tx distribution withdraw-all-rewards --from=$WALLET --chain-id=$CHAIN_ID --gas=auto
```

Komisyon ile ödülleri geri çekin
```
archwayd tx distribution withdraw-rewards $VALOPER_ADDRESS --from=$WALLET --commission --chain-id=$CHAIN_ID
```

Hapisten Çıkarma
```
archwayd tx slashing unjail \
  --broadcast-mode=block \
  --from=$WALLET \
  --chain-id=$CHAIN_ID \
  --gas=auto
```
