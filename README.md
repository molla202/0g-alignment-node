# 0G AI Alignment Node - Servis Kurulum Rehberi

## Sistem Gereksinimleri

Öncelikle sisteminizin bu minimum gereksinimleri karşıladığından emin olun:

| Bileşen | Minimum Gereksinim |
|---------|-------------------|
| RAM | 64 MB |
| CPU | 1 x86 Core @ 2.1GHz |
| Disk Alanı | 10 GB |
| İnternet | 10 Mbps bağlantı |
| Network | Port dışarıdan erişilebilir olmalı (firewall'da yapılandır) |

NOT: private key işlemelride baştaki 0X kullanmıyoruz.

# LOGLARINIZI KİMSEYLE PAYLAŞMAYINIZ... KİMSEYLE

# CoreNode Operator Adres : 0xb984B1f158963417467900b4Be868f83deA007fC

## Adım 1: Dizin Yapısı ve Binary Kurulumu

### 1. Ana dizini ve alt klasörleri oluşturun:
```bash
# Ana 0G dizinini ev dizininde oluştur
mkdir -p $HOME/0g-ai/{bin,config,logs}
```

### 2. Node binary'sini indirin ve yerleştirin:
```bash
# 0g-ai dizinine geçin
cd $HOME/0g-ai

# En son 0G alignment node binary'sini indirin
wget https://github.com/0gfoundation/alignment-node-release/releases/download/v1.0.0/alignment-node.tar.gz
tar -xzf alignment-node.tar.gz

# Çalıştırma izni verin
mv alignment-node/0g-alignment-node $HOME/0g-ai/bin/0g-alignment-node
chmod +x $HOME/0g-ai/bin/0g-alignment-node
rm -rf alignment-node alignment-node.tar.gz
```

### 3. Sistem genelinde erişim için symbolic link oluşturun:
```bash
# /usr/local/bin'e symbolic link oluştur (sistem geneli erişim)
sudo ln -s $HOME/0g-ai/bin/0g-alignment-node /usr/local/bin/0g-alignment-node -f

# Link'in çalıştığını test edin
0g-alignment-node --help
```

## Adım 2: Yapılandırma Dosyasını Oluşturun

### 1. Yapılandırma dosyasını oluşturun:
```bash
# Config dizinine .env dosyası oluştur
nano $HOME/0g-ai/config/.env
```

### 2. .env Dosyası İçeriği:
```bash
ZG_ALIGNMENT_NODE_LOG_LEVEL="info"
ZG_ALIGNMENT_NODE_SERVICE_IP="http://0.0.0.0:8080"
ZG_ALIGNMENT_NODE_SERVICE_PRIVATEKEY="your_private_key_here"
```
```bash
# Config dizinine config.toml dosyası oluştur
nano $HOME/0g-ai/config/config.toml
```
```bash
ZG_ALIGNMENT_NODE_LOG_LEVEL="info"
ZG_ALIGNMENT_NODE_SERVICE_IP="http://0.0.0.0:8080"
ZG_ALIGNMENT_NODE_SERVICE_PRIVATEKEY="your_private_key_here"
```

**Önemli Notlar:**
- `LOG_LEVEL`: Sorun giderme için `debug`, normal çalışma için `info`
- `SERVICE_PORT`: Varsayılan 8080. Bu portun dışarıdan erişilebilir olduğundan emin olun
- `PRIVATEKEY`: Alignment node lisanslarınızı tutan cüzdanınızın private key'i

## Adım 3: Network Yapılandırması

Yapılandırmanızda belirtilen port (varsayılan 8080) consensus iletişimi için dışarıdan erişilebilir olmalıdır.

Bu portun açık olduğundan emin olun:
- Cloud security groups/firewall'larda (AWS, Azure, GCP vb.)
- VPS sağlayıcı firewall'larında
- Yerel sunucu firewall kurallarında

## Register.
```
cd
cd $HOME/0g-ai/config
```
```
0g-alignment-node approve --mainnet --key privatey-key-yaz --chain-id 42161 --rpc https://arb1.arbitrum.io/rpc --contract 0xdD158B8A76566bC0c342893568e8fd3F08A9dAac --destNode cüzdan-adresi-yaz --tokenIds YOUR_NFT_TOKEN_ID
```
```
0g-alignment-node registerOperator --key privatey-key-yaz --token-id YOUR_NFT_TOKEN_ID --commission 10 --chain-id 42161 --rpc https://arb1.arbitrum.io/rpc --contract 0xdD158B8A76566bC0c342893568e8fd3F08A9dAac --mainnet
```

NOT: daha sonraki işlemleri https://claim.0gfoundation.ai/ yapıcaksınız.
## Adım 4: Systemd Servisi Oluşturma

Node'unuzu systemd servisi olarak çalıştırmak için:

### 1. Servis dosyasını tee ile oluşturun:
```bash
sudo tee /etc/systemd/system/0g-alignment-node.service > /dev/null <<EOF
[Unit]
Description=0G AI Alignment Node
After=network.target
Wants=network.target

[Service]
Type=simple
User=$USER
WorkingDirectory=$HOME/0g-ai
EnvironmentFile=$HOME/0g-ai/config/.env
ExecStart=$(which 0g-alignment-node) start --mainnet
Restart=always
RestartSec=10
StandardOutput=journal
StandardError=journal
SyslogIdentifier=0g-alignment-node

# Log dosyasını da tutmak isterseniz (opsiyonel)
# StandardOutput=append:$HOME/0g-ai/logs/node.log
# StandardError=append:$HOME/0g-ai/logs/node-error.log

[Install]
WantedBy=multi-user.target
EOF
```

**Not:** 
- `User=$USER` mevcut kullanıcıyı kullanır
- Sistem servisi olduğu için tam path kullanıyoruz
- `$(which 0g-alignment-node)` dinamik olarak binary'nin tam yolunu bulur

### 2. Servisi etkinleştirme ve başlatma:
```bash
# Systemd daemon'unu yeniden yükle
sudo systemctl daemon-reload

# Servisi etkinleştir (boot'ta otomatik başlasın)
sudo systemctl enable 0g-alignment-node

# Servisi başlat
sudo systemctl start 0g-alignment-node
```

## Servis Yönetimi Komutları

```bash
# Servisi başlat
sudo systemctl start 0g-alignment-node

# Servisi durdur
sudo systemctl stop 0g-alignment-node

# Servisi yeniden başlat
sudo systemctl restart 0g-alignment-node

# Servis durumunu kontrol et
sudo systemctl status 0g-alignment-node

# Servisi devre dışı bırak
sudo systemctl disable 0g-alignment-node

# Logları görüntüle
sudo journalctl -u 0g-alignment-node -f

# Son 100 log girişini görüntüle
sudo journalctl -u 0g-alignment-node -n 100
```

# LOGLARINIZI KİMSEYLE PAYLAŞMAYINIZ... KİMSEYLE

## Node İzleme

### Node durumunu kontrol etme:
```bash
# Sistemde herhangi bir yerden çalıştırabilirsiniz
0g-alignment-node status

# which komutu ile tam yolu görebilirsiniz
which 0g-alignment-node
```

### Logları görüntüleme:
```bash
# Systemd logları (önerilen)
sudo journalctl -u 0g-alignment-node -f

# Eğer dosyaya da log yazdırıyorsanız
tail -f $HOME/0g-ai/logs/node.log
```

### Sağlıklı çalışma kontrolleri:
- Durum raporları hata içermemeli
- Loglar düzenli aktivite göstermeli, sürekli çökmeler olmamalı

## Sorun Giderme

### Node bağlanamıyor:
- Portun açık ve dışarıdan erişilebilir olduğunu doğrulayın
- Firewall/security group ayarlarınızı kontrol edin
- Private key'in ilişkili lisanslara sahip olduğundan emin olun

### Node çöküyor:
- Hatalara ilişkin logları kontrol edin
- Sistem gereksinimlerinin karşılandığını doğrulayın
- İnternet bağlantısının kararlı olduğundan emin olun

## En İyi Uygulamalar

### Kendi Barındırılan Node'lar İçin:
- **Düzenli Güncellemeler:** Node binary'sini güncel tutun
- **İzleme:** Kesinti durumu için uyarılar ayarlayın
- **Yedekleme:** Private key'lerin güvenli yedeğini saklayın
- **Güvenlik:** Node işlemi için özel cüzdan kullanın
- **Ağ:** Kararlı internet bağlantısı sağlayın

## Güvenlik Notları

1. **Private Key Güvenliği:** Private key'inizi güvenli bir şekilde saklayın
2. **Firewall Yapılandırması:** Sadece gerekli portları açın
3. **Düzenli Güncelleme:** Node yazılımını düzenli olarak güncelleyin
4. **Log İzleme:** Anormal aktiviteleri takip edin

## Güncelleme İşlemi

Yeni sürüm geldiğinde güncelleme yapmak çok kolay:

### 1. Servisi durdurun:
```bash
sudo systemctl stop 0g-alignment-node
```

### 2. Mevcut binary'yi yedekleyin (opsiyonel):
```bash
cp $HOME/0g-ai/bin/0g-alignment-node $HOME/0g-ai/bin/0g-alignment-node.backup
```

### 3. Yeni binary'yi indirip değiştirin:
```bash
...

# Symbolic link otomatik olarak yeni binary'yi işaret edecek
```

### 4. Servisi yeniden başlatın:
```bash
sudo systemctl start 0g-alignment-node
```

### 5. Güncellemeyi doğrulayın:
```bash
# Versiyon kontrolü
0g-alignment-node version

# Servis durumu kontrolü
sudo systemctl status 0g-alignment-node
```

## Dizin Yapısı Özeti

Kurulum sonunda dizin yapınız şöyle olacak:

```
$HOME/0g-ai/
├── bin/
│   └── 0g-alignment-node          # Binary dosyası
├── config/
│   └── .env                       # Yapılandırma dosyası
│   └── config.toml
└── logs/                          # Log dosyaları (opsiyonel)
    ├── node.log
    └── node-error.log

/usr/local/bin/
└── 0g-alignment-node -> $HOME/0g-ai/bin/0g-alignment-node  # Symbolic link

/etc/systemd/system/
└── 0g-alignment-node.service      # Systemd servis dosyası
```

## Ek Komutlar

```bash
# Node versiyonunu kontrol et
0g-alignment-node version

# Binary'nin nerede olduğunu göster
which 0g-alignment-node

# Yapılandırma dosyasını test et
0g-alignment-node validate-config

# Manuel çalıştırmak isterseniz (test için)
cd $HOME/0g-ai && source config/.env && 0g-alignment-node start

# Symbolic link'i yeniden oluşturmak için
sudo ln -s $HOME/0g-ai/bin/0g-alignment-node /usr/local/bin/0g-alignment-node -f
```

Bu rehber ile 0G AI Alignment Node'unuzu başarıyla systemd servisi olarak kurabilir ve çalıştırabilirsiniz. Herhangi bir sorunla karşılaşırsanız, logları kontrol edin ve sorun giderme bölümündeki adımları izleyin.
