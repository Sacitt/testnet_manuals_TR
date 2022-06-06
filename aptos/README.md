<p style="font-size:14px" align="right">
Join our telegram <a href="https://t.me/sacithbn" target="_blank"><img src="https://user-images.githubusercontent.com/50621007/168689534-796f181e-3e4c-43a5-8183-9888fc92cfa7.png" width="30"/></a>

<p align="center">
  <img width="100" height="auto" src="https://user-images.githubusercontent.com/50621007/165930080-4f541b46-1ae3-461c-acc9-de72d7ab93b7.png">
</p>

# Devnet için Aptos fullnode kurulumu
Resmi belgeler:
> [tam node kurulumu](https://aptos.dev/tutorials/run-a-fullnode)

Kullanışlı araçlar:
> En son blok yüksekliğini bulmak için [Aptos Network Dashboard](https://status.devnet.aptos.dev/)\
> Düğüm sağlık durumunuzu kontrol etmek için [Aptos Node Informer](http://node-tools.net/aptos/tester/)\
> Tam düğümünüzü başka bir makineye taşımak için  [Migrate your fullnode to another machine](https://github.com/kj89/testnet_manuals/blob/main/aptos/migrate_fullnode.md)

## Donanım Gereksinimleri:
#### Bir üretim sınıfı Fullnode çalıştırmak için aşağıdakileri öneririz:
- CPU: 4 cores (Intel Xeon Skylake or newer)
- Memory: 8GiB RAM

#### Fullnode'u geliştirme veya test amacıyla çalıştırıyorsanız:
- CPU: 2 cores
- Memory: 4GiB RAM

## aptos fullnode'unuzu kurun
### Seçenek 1 (otomatik)
Hızlı kurulum için aşağıdaki komut dosyasını kullanın
```
wget -qO aptos.sh https://raw.githubusercontent.com/kj89/testnet_manuals/main/aptos/aptos.sh && chmod +x aptos.sh && ./aptos.sh
```

### Seçenek 2 (manuel)
Düğümü manuel olarak kurmayı tercih ederseniz , [manual klavuz](https://github.com/kj89/testnet_manuals/blob/main/aptos/manual_install.md) takip edebilirsiniz . 

## Aptos Fullnode sürümünü güncelleyin
```
wget -qO update.sh https://raw.githubusercontent.com/kj89/testnet_manuals/main/aptos/tools/update.sh && chmod +x update.sh && ./update.sh
```

## (İSTEĞE BAĞLI) Yapılandırmaları güncelleyin
```
wget -qO update_configs.sh https://raw.githubusercontent.com/kj89/testnet_manuals/main/aptos/tools/update_configs.sh && chmod +x update_configs.sh && ./update_configs.sh
```

## Düğüm kimliğinizi ve yukarı akış ayrıntılarınızı alın
```
wget -qO get_identity.sh https://raw.githubusercontent.com/kj89/testnet_manuals/main/aptos/tools/get_identity.sh && chmod +x get_identity.sh && ./get_identity.sh
```

## Faydalı komutlar
### Aptos günlüklerini kontrol edin
```
docker logs -f aptos-fullnode-1 --tail 50
```

### Senkronizasyon durumunu kontrol edin
```
curl 127.0.0.1:9101/metrics 2> /dev/null | grep aptos_state_sync_version | grep type
```

### Servisi yeniden başlat
```
docker restart aptos-fullnode-1
```
