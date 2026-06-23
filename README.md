# 🟢 GenLayer Validator Node — Tam Kurulum Rehberi

<img width="1937" height="750" alt="genlayer" src="https://github.com/user-attachments/assets/3ceea3c3-303d-40fd-98ae-7c0456f5c6fe" />

> Adım adım Türkçe manuel kurulum rehberi.  
> Ağ: **Testnet Asimov Phase 5**  
> Kaynak: [docs.genlayer.com/validators/setup-guide](https://docs.genlayer.com/validators/setup-guide)

---

## 📋 İçindekiler

1. [Sistem Gereksinimleri](#1-sistem-gereksinimleri)
2. [Sunucu Hazırlığı](#2-sunucu-hazırlığı)
3. [GenLayer CLI ve Validator Cüzdanı](#3-genlayer-cli-ve-validator-cüzdanı)
4. [Node Yazılımını İndir ve Kur](#4-node-yazılımını-indir-ve-kur)
5. [Konfigürasyon Dosyasını Oluştur](#5-konfigürasyon-dosyasını-oluştur)
6. [Operator Key'i İçe Aktar](#6-operator-keyi-i̇çe-aktar)
7. [Node'u Servis Olarak Çalıştır](#7-nodeu-servis-olarak-çalıştır)
8. [Faydalı Komutlar](#8-faydalı-komutlar)
9. [Güncelleme Rehberi](#9-güncelleme-rehberi)
10. [Sık Yapılan Hatalar](#10-sık-yapılan-hatalar)

---

## 1. Sistem Gereksinimleri

| Kaynak | Gereksinim |
|--------|-----------|
| **RAM** | 16 GB |
| **CPU** | 8 çekirdek — **AMD64 (x86_64) zorunlu!** ARM çalışmaz. |
| **Disk** | 128 GB SSD/NVMe |
| **Ağ** | 100 Mbps |
| **İşletim Sistemi** | 64-bit Linux (Ubuntu 20.04+ önerilir) |
| **Yazılım** | Docker, Python 3, Node.js v18+, npm |

> ⚠️ **ÖNEMLİ:** Mutlaka AMD64/x86_64 mimarili sunucu kullan. `uname -m` komutu `x86_64` döndürmelidir.

> 💰 **Token Gereksinimi:** Minimum **42.000 GEN token** gereklidir.

---

## 2. Sunucu Hazırlığı

### 2.1 Sistem Güncellemesi

```bash
sudo apt update && sudo apt upgrade -y
```

### 2.2 Temel Araçları Kur

```bash
sudo apt install -y curl wget git build-essential python3 python3-pip python3-venv ca-certificates gnupg lsb-release
```

### 2.3 Docker Kurulumu

```bash
# Eski Docker sürümlerini kaldır
sudo apt remove -y docker docker-engine docker.io containerd runc 2>/dev/null || true

# Resmi GPG anahtarını ekle
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
  sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Repoyu ekle
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Docker'ı kur
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

# Kullanıcını docker grubuna ekle (sudo'suz docker kullanımı)
sudo usermod -aG docker $USER
newgrp docker

# Doğrula
docker --version
docker compose version
```
- OR
```
wget -q -O docker.sh https://raw.githubusercontent.com/molla202/molla202/refs/heads/main/docker.sh && chmod +x docker.sh && sudo /bin/bash docker.sh
```
### 2.4 Node.js v18+ Kurulumu

```bash
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs

# Doğrula (18.x veya üzeri olmalı)
node --version
npm --version
```

### 2.5 Mimari Doğrulama

```bash
uname -m
# x86_64 görmelisin. Başka bir şey varsa bu sunucu çalışmaz.
```

---

## 3. GenLayer CLI ve Validator Cüzdanı

> ⚠️ Bu adım **kendi bilgisayarında** (local) yapılmalıdır. Owner adresi soğuk cüzdanda tutulmalıdır.

### 3.1 GenLayer CLI'yi Kur

```bash
npm install -g genlayer
genlayer --version
```

### 3.2 Staking Wizard'ı Çalıştır

```bash
genlayer staking wizard
```

Wizard sırasıyla şunları yapar:
1. Owner hesabı oluşturur veya mevcut cüzdanı bağlar
2. Ağ olarak `testnet-asimov` seçersin
3. En az 42.000 GEN token bakiyeni doğrular
4. Operator keystore dosyasını oluşturur (bir export şifresi belirlersin)
5. Stake miktarını girersin (min: 42.000)
6. Validator akıllı kontratını deploy eder
7. Moniker, website gibi kimlik bilgilerini girersin

### 3.2b Var olan cüzdanı import etme 
```bash
genlayer account import --password pass-write --private-key 0x --name name
```
NOT: daha sonra wizard kodunuzu girince cüzdanınız görünecek onu seçip validator kontratını deploy edıyoruz.

### 3.3 Bu Bilgileri Not Al!

```
Owner Adresi    : 0x...  ← Stake çekme yetkisi — SOĞUK CÜZDANDA TUT
Operator Adresi : 0x...  ← Node'un imzalama adresi (config'de lazım)
Validator Wallet: 0x...  ← Akıllı kontrat adresi (config'de lazım)
```

### 3.4 Keystore Dosyasını Sunucuya Kopyala
NOT: eğer harici bir eyrde yapmadıysanız bu adıma gerek yok.
```bash
# Kendi makinenden çalıştır — IP ve kullanıcı adını değiştir
scp ./operator-keystore.json kullanici@SUNUCU_IP:$HOME/operator-keystore.json
```

### 3.5 Validator Durumunu Kontrol Et

```bash
genlayer staking validator-info --validator 0xVALIDATOR_WALLET_ADRESİN
```

---

## 4. Node Yazılımını İndir ve Kur

> Bu adımdan itibaren sunucunda çalışıyorsun.

### 4.1 Mevcut Versiyonları Listele

```bash
curl -s "https://storage.googleapis.com/storage/v1/b/gh-af/o?prefix=genlayer-node/bin/amd64" | \
  grep -o '"name": *"[^"]*"' | \
  sed -n 's/.*\/\(v[^/]*\)\/.*/\1/p' | \
  sort -ru | grep -v "rc" | head -n 5
```

### 4.2 İndir ve Sabit Klasöre Çıkart

```bash
export VERSION=v0.5.12   # Yukarıdaki listeden en yeni versiyonu kullan

wget https://storage.googleapis.com/gh-af/genlayer-node/bin/amd64/${VERSION}/genlayer-node-linux-amd64-${VERSION}.tar.gz

# Sabit isimli klasöre çıkart — güncelleme sırasında bu isim değişmeyecek
mkdir -p $HOME/.genlayer
tar -xzvf genlayer-node-linux-amd64-${VERSION}.tar.gz
mv $HOME/genlayer-node-linux-amd64/* $HOME/.genlayer/
cd $HOME/.genlayer
```

### 4.3 GenVM Kurulumunu Çalıştır

```bash
python3 ./third_party/genvm/bin/setup.py
```

> Birkaç dakika sürebilir, sabırla bekle.

### 4.4 Sistem Symlink'i Oluştur

```bash
sudo ln -sf $HOME/.genlayer/bin/genlayernode /usr/local/bin/genlayernode

# Doğrula
which genlayernode
```

---

## 5. Konfigürasyon Dosyasını Oluştur

### 5.1 Gerekli Klasörler

```bash
mkdir -p $HOME/.genlayer/configs/node
mkdir -p $HOME/.genlayer/data
mkdir -p $HOME/.genlayer/logs
```

### 5.2 Değerleri Önce Tanımla

```bash
# --- Kendi bilgilerinle doldur ---
VALIDATOR_WALLET="0xVALIDATOR_WALLET_ADRESİN"   # Wizard'dan aldın
OPERATOR_ADDRESS="0xOPERATOR_ADRESİN"            # Wizard'dan aldın
# --------------------------------

echo "Validator : $VALIDATOR_WALLET"
echo "Operator  : $OPERATOR_ADDRESS"
```

> **Asimov RPC:** Caldera tarafından sağlanır.  
> HTTP: `https://zksync-os-testnet-genlayer.zksync.dev/http`  
> WS: `wss://zksync-os-testnet-genlayer.zksync.dev/ws`  
> Birden fazla validator aynı RPC'yi paylaşabilir veya kendi ZKSync full node'unu kurabilirsin.

### 5.3 config.yaml Oluştur

```bash
cat > $HOME/.genlayer/configs/node/config.yaml << EOF
# rollup configuration
rollup:
  genlayerchainrpcurl: "https://zksync-os-testnet-genlayer.zksync.dev/http"
  genlayerchainwebsocketurl: "wss://zksync-os-testnet-genlayer.zksync.dev/ws"
  provider: "caldera"

# Testnet Asimov Phase 5 — consensus configuration
consensus:
  consensusaddress: "0xe66B434bc83805f380509642429eC8e43AE9874a"
  genesis: 17326

# data directory
datadir: "./data/node"

# logging configuration
logging:
  level: "INFO"
  json: false
  file:
    enabled: true
    level: "DEBUG"
    folder: logs
    maxsize: 10
    maxage: 7
    maxbackups: 100
    localtime: false
    compress: true

# node configuration
node:
  mode: "validator"
  validatorWalletAddress: "${VALIDATOR_WALLET}"
  operatorAddress: "${OPERATOR_ADDRESS}"
  admin:
    port: 9155
  rpc:
    port: 9151
    endpoints:
      groups:
        genlayer: true
        genlayer_debug: true
        ethereum: true
        zksync: true
  ops:
    port: 9153
    endpoints:
      metrics: true
      health: true
      balance: true

# genvm configuration
genvm:
  root_dir: ./third_party/genvm
  start_manager: true
  manager_url: http://127.0.0.1:3999
  permits: 8

# advanced configuration
merkleforest:
  maxdepth: 16
  dbpath: "./data/node/merkle/forest/data.db"
  indexdbpath: "./data/node/merkle/index.db"
merkletree:
  maxdepth: 16
  dbpath: "./data/node/merkle/tree/"

# metrics configuration
metrics:
  interval: "15s"
  collectors:
    node:
      enabled: true
    genvm:
      enabled: true
    webdriver:
      enabled: true
EOF
```

Kontrol et:

```bash
cat $HOME/.genlayer/configs/node/config.yaml
```

---

## 6. Operator Key'i İçe Aktar
NOT: eğer daha once var olan cüzdanı import ettiyseniz yada yeni cüzdan olusturduysanız wizard işleminde export ederkene verilen bilgilere dikkat edin. çünkü şuan aşağıda lazım :D
```bash
./bin/genlayernode account import \
  --password "BURAYA_NODE_ŞIFRENI_YAZ" \
  --passphrase "BURAYA_WIZARD_EXPORT_ŞIFRESINI_YAZ" \
  --path "/home/kullanici/operator-keystore.json" \
  -c $(pwd)/configs/node/config.yaml \
  --setup
```

Başarılı çıktı:

```
Account imported:
  Address: 0x...
  Account setup as a validator
```

> Görünen adresin wizard'dan aldığın **Operator Adresi** ile eşleştiğini doğrula.

### Key Yedeği Al

```bash
genlayernode account export \
  --password "NODE_ŞİFREN" \
  --address "0xOPERATOR_ADRESİN" \
  --passphrase "YEDEK_ŞİFREN" \
  --path "$HOME/operator-backup.key" \
  -c $HOME/.genlayer/configs/node/config.yaml
```

> 🔒 Bu dosyayı güvenli bir yerde sakla!

---

## 7. Node'u Servis Olarak Çalıştır

LLM API key'ini **node binary'si** doğrudan kullanır (GenVM modülü üzerinden).
Bu nedenle key, systemd servisine `Environment=` direktifi ile verilir.

### 7.1 WebDriver'ı Test Et

Servis kurmadan önce WebDriver'ı doğrula:

```bash
cd $HOME/.genlayer
docker compose up -d

# Container çalışıyor mu?
docker ps
# ".genlayer-webdriver" görünmeli
```

### 7.2 Konfigürasyonu Doğrula

```bash
cd $HOME/.genlayer
genlayernode doctor
```

Tüm kontroller geçmeli. Hata varsa servise geçmeden önce düzelt.

### 7.3 systemd Servis Dosyasını Oluştur
> [Heurist](https://www.heurist.ai/) — free [API credits](https://dev-api-form.heurist.ai/) with referral code "genlayer".
> Heurist kullanıyorsan `HEURISTKEY`'i doldur. Başka sağlayıcı kullanıyorsan
> ilgili `Environment=` satırını ekle (COMPUT3KEY, IOINTELLIGENCE_API_KEY vb.)

```bash
sudo tee /etc/systemd/system/.genlayer.service > /dev/null << EOF
[Unit]
Description=GenLayer Validator Node — Testnet Asimov
After=network-online.target docker.service
Wants=network-online.target
Requires=docker.service

[Service]
Type=simple
User=$USER
WorkingDirectory=$HOME/.genlayer

# Node şifresi
Environment="GENLAYERNODE_PASSWORD=BURAYA_NODE_ŞİFRENİ_YAZ"

# LLM API Key — kullandığın sağlayıcının satırını doldur
Environment="HEURISTKEY=BURAYA_HEURIST_API_KEY_YAZ"
#Environment="COMPUT3KEY=BURAYA_COMPUT3_API_KEY_YAZ"
#Environment="IOINTELLIGENCE_API_KEY=BURAYA_IONET_API_KEY_YAZ"
#Environment="CHUTES_API_KEY=BURAYA_CHUTES_API_KEY_YAZ"
#Environment="OPENAIKEY=BURAYA_OPENAI_API_KEY_YAZ"

ExecStartPre=/usr/bin/docker compose up -d
ExecStart=/usr/local/bin/genlayernode run \
  -c $HOME/.genlayer/configs/node/config.yaml \
  --password \${GENLAYERNODE_PASSWORD}

Restart=on-failure
RestartSec=10
LimitNOFILE=65535
StandardOutput=journal
StandardError=journal
SyslogIdentifier=.genlayer

[Install]
WantedBy=multi-user.target
EOF
```

### 7.4 Servisi Başlat

```bash
sudo systemctl daemon-reload
sudo systemctl enable .genlayer
sudo systemctl start .genlayer

# Durumu kontrol et
sudo systemctl status .genlayer
```

### 7.5 Logları Takip Et

```bash
# Canlı log
sudo journalctl -u .genlayer -f

# Node sync oldu mu?
sudo journalctl -u .genlayer | grep "Node is synced"
# "Node is synced!!! blockNumber=XXXXX" görmelisin
```

---

## 8. Faydalı Komutlar

### Servis Yönetimi

```bash
sudo systemctl stop .genlayer
sudo systemctl restart .genlayer
sudo systemctl status .genlayer
sudo journalctl -u .genlayer -n 100 --no-pager
```

### Node Durumu

```bash
curl http://localhost:9153/health
curl http://localhost:9153/metrics
curl http://localhost:9153/balance
```

### Hesap Komutları

```bash
genlayernode account list -c $HOME/.genlayer/configs/node/config.yaml
```

### Validator Yönetimi

```bash
# Validator bilgileri
genlayer staking validator-info --validator 0xVALIDATOR_WALLET_ADRESİN

# Aktif validatorlar
genlayer staking active-validators

# Ek stake
genlayer staking validator-deposit --amount 1000gen

# Çıkış (7-epoch unbonding başlar)
genlayer staking validator-exit --shares 100

# Fonları çek (unbonding bittikten sonra)
genlayer staking validator-claim

# Kimlik güncelle
genlayer staking set-identity --validator 0x... --moniker "YeniIsim"
```

---

## 9. Güncelleme Rehberi

`configs`, `data` ve `logs` klasörleri korunur. Sadece `bin` ve `third_party` güncellenir.

```bash
export NEW_VERSION=v0.5.13

# Yeni versiyonu indir
wget https://storage.googleapis.com/gh-af/.genlayer/bin/amd64/${NEW_VERSION}/.genlayer-linux-amd64-${NEW_VERSION}.tar.gz

# Geçici klasöre çıkart
mkdir -p $HOME/.genlayer-new
tar -xzvf .genlayer-linux-amd64-${NEW_VERSION}.tar.gz -C $HOME/.genlayer-new

# Yeni GenVM kurulumu
python3 $HOME/.genlayer-new/third_party/genvm/bin/setup.py

# Servisi durdur
sudo systemctl stop .genlayer
docker compose -f $HOME/.genlayer/docker-compose.yaml down

# Binary ve GenVM'i güncelle (configs/data/logs dokunulmaz)
rm -rf $HOME/.genlayer/bin $HOME/.genlayer/third_party
cp -r $HOME/.genlayer-new/bin $HOME/.genlayer/
cp -r $HOME/.genlayer-new/third_party $HOME/.genlayer/
cp $HOME/.genlayer-new/docker-compose.yaml $HOME/.genlayer/

# Symlink güncelle
sudo ln -sf $HOME/.genlayer/bin/genlayernode /usr/local/bin/genlayernode
genlayernode --version

# Servisi başlat
sudo systemctl start .genlayer
sudo journalctl -u .genlayer -f

# Temizlik
rm -rf $HOME/.genlayer-new
rm .genlayer-linux-amd64-*.tar.gz
```

---

## 10. Sık Yapılan Hatalar

### ❌ exec format error

ARM işlemci kullanıyorsun. AMD64 sunucuya geç.
```bash
uname -m   # x86_64 olmalı
```

### ❌ NO_PROVIDER_FOR_PROMPT

Servis dosyasında LLM API key tanımlı değil veya boş bırakılmış.
```bash
sudo systemctl edit .genlayer
# Environment="HEURISTKEY=..." satırını güncelle
sudo systemctl restart .genlayer
```

### ❌ Node validator modunda çalışmıyor

`configs/node/config.yaml` içinde `validatorWalletAddress` veya `operatorAddress` boş.
```bash
nano $HOME/.genlayer/configs/node/config.yaml
sudo systemctl restart .genlayer
```

### ❌ Docker permission denied

```bash
sudo usermod -aG docker $USER
newgrp docker
```

### ❌ WebDriver başlamıyor

```bash
cd $HOME/.genlayer
docker compose down
docker compose up -d
docker logs .genlayer-webdriver
```

### ❌ Quarantine durumu

Yeniden başlatma veya güncelleme sonrası geçici olarak normal. Epoch'lar tamamlandıkça otomatik temizlenir. Node'un çalışmaya devam ettiğinden ve LLM + WebDriver'ın aktif olduğundan emin ol.

---

## 📚 Kaynaklar

- [Resmi Kurulum Rehberi](https://docs.genlayer.com/validators/setup-guide)
- [GenVM Konfigürasyonu](https://docs.genlayer.com/validators/genvm-configuration)
- [Explorer — Asimov](https://genlayer-explorer.vercel.app/)
- [Discord](https://discord.gg/8Jm4v89VAu)
- [Telegram](https://t.me/genlayer)

---

> ✅ Sorun yaşarsan GenLayer Discord `#validators` kanalına sorabilirsin.
