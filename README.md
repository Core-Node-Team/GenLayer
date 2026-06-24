# 🟢 GenLayer Validator Node — Testnet Asimov (Phase 5)

<img width="1937" height="750" alt="genlayer" src="https://github.com/user-attachments/assets/3ceea3c3-303d-40fd-98ae-7c0456f5c6fe" />

> 🇹🇷 Adım adım Türkçe manuel kurulum rehberi.  
> 🌐 Ağ: **Testnet Asimov Phase 5**  
> 📖 Kaynak: [docs.genlayer.com/validators/setup-guide](https://docs.genlayer.com/validators/setup-guide)

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
| 🖥️ **RAM** | 16 GB |
| ⚙️ **CPU** | 8 çekirdek — **AMD64 (x86_64) zorunlu!** ARM çalışmaz. |
| 💾 **Disk** | 128 GB SSD/NVMe |
| 🌐 **Ağ** | 100 Mbps |
| 🐧 **İşletim Sistemi** | 64-bit Linux (Ubuntu 20.04+ önerilir) |
| 🔧 **Yazılım** | Docker, Python 3, Node.js v18+, npm |

> ⚠️ **ÖNEMLİ:** Mutlaka AMD64/x86_64 mimarili sunucu kullan. `uname -m` komutu `x86_64` döndürmelidir.

> 💰 **Token Gereksinimi:** Minimum **42.000 GEN token** gereklidir.

---

## 2. Sunucu Hazırlığı

### 2.1 🔄 Sistem Güncellemesi

```bash
sudo apt update && sudo apt upgrade -y
```

### 2.2 🛠️ Temel Araçları Kur

```bash
sudo apt install -y curl wget git build-essential python3 python3-pip python3-venv ca-certificates gnupg lsb-release
```

### 2.3 🐳 Docker Kurulumu

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

> 💡 **Alternatif — tek komutla Docker kurulumu:**
> ```bash
> wget -q -O docker.sh https://raw.githubusercontent.com/molla202/molla202/refs/heads/main/docker.sh && chmod +x docker.sh && sudo /bin/bash docker.sh
> ```

### 2.4 🟩 Node.js v18+ Kurulumu

```bash
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs

# Doğrula (18.x veya üzeri olmalı)
node --version
npm --version
```

### 2.5 ✅ Mimari Doğrulama

```bash
uname -m
# x86_64 görmelisin. Başka bir şey varsa bu sunucu çalışmaz.
```

---

## 3. GenLayer CLI ve Validator Cüzdanı

> ⚠️ Bu adım **kendi bilgisayarında** (local) yapılmalıdır. Owner adresi soğuk cüzdanda tutulmalıdır.

> 💡 **Bu bölümün amacı nedir?**  
> Validator olabilmek için bir **validator akıllı kontratı** deploy etmen gerekiyor.  
> Bu kontrat `genlayer staking wizard` ile oluşturuluyor.  
> Wizard aynı zamanda node'un blok imzalamak için kullanacağı **operator adresini** de üretiyor.  
> Operator adresi olmadan node validator modunda çalışamaz.

---

### 3.1 📦 GenLayer CLI'yi Kur

```bash
npm install -g genlayer
genlayer --version
```

---

### 3.2 🔑 Mevcut Cüzdanını Import Et (opsiyonel)

Eğer zaten bir owner cüzdanın varsa wizard'ı çalıştırmadan **önce** bunu CLI'ya tanıt.  
Wizard açıldığında mevcut hesaplarını listeler — import ettiğin cüzdanı seçerek devam edebilirsin.

Yeni cüzdan oluşturacaksan bu adımı atlayabilirsin, wizard sana yeni hesap oluşturma seçeneği sunar.

```bash
genlayer account import \
  --password "KAYIT_ŞIFRESI" \
  --private-key "0xOWNER_PRIVATE_KEY" \
  --name "owner"
```

---

### 3.3 🧙 Staking Wizard'ı Çalıştır

```bash
genlayer staking wizard
```

Wizard sırasıyla şunları yapar:

| Adım | Açıklama |
|------|----------|
| 1️⃣ Hesap seçimi | Import ettiğin cüzdanı seç **veya** yeni owner hesabı oluştur |
| 2️⃣ Ağ seçimi | `testnet-asimov` seç |
| 3️⃣ Bakiye kontrolü | En az 42.000 GEN token olduğunu doğrular |
| 4️⃣ Operator oluşturma | Node'un imzalama adresi olan operator hesabını oluşturur |
| 5️⃣ Stake miktarı | Kaç GEN stake edeceğini girersin (min: 42.000) |
| 6️⃣ Kontrat deploy | Validator akıllı kontratını zincire yazar |
| 7️⃣ Kimlik ayarı | Moniker, website gibi bilgileri girersin |

> ✅ Wizard tamamlandığında sende hem bir **validator wallet adresi** (akıllı kontrat)  
> hem de bir **operator adresi** (imzalama hesabı) olacak. İkisi de node config\'inde zorunlu.

---

### 3.4 📝 Bu Bilgileri Not Al!

```
Owner Adresi        : 0x...  ← Stake çekme yetkisi — SOĞUK CÜZDANDA TUT
Operator Adresi     : 0x...  ← Node'un imzalama adresi (config'de lazım)
Operator Private Key: 0x...  ← Node'a import için lazım — GÜVENLİ SAKLA
Validator Wallet    : 0x...  ← Akıllı kontrat adresi (config'de lazım)
```

---

### 3.5 🔍 Validator Durumunu Kontrol Et

```bash
genlayer staking validator-info --validator 0xVALIDATOR_WALLET_ADRESİN
```


---

## 4. Node Yazılımını İndir ve Kur

> 🖥️ Bu adımdan itibaren **sunucunda** çalışıyorsun.

### 4.1 🔍 Mevcut Versiyonları Listele

```bash
curl -s "https://storage.googleapis.com/storage/v1/b/gh-af/o?prefix=genlayer-node/bin/amd64" | \
  grep -o '"name": *"[^"]*"' | \
  sed -n 's/.*\/\(v[^/]*\)\/.*/\1/p' | \
  sort -ru | grep -v "rc" | head -n 5
```

### 4.2 ⬇️ İndir ve Sabit Klasöre Çıkart

Kurulum boyunca `$HOME/.genlayer` sabit klasörünü kullanacağız.
Güncelleme sırasında bu isim değişmez — sadece `bin` ve `third_party` içeriği güncellenir.

```bash
export VERSION=v0.5.12   # Yukarıdaki listeden en yeni versiyonu kullan

wget https://storage.googleapis.com/gh-af/genlayer-node/bin/amd64/${VERSION}/genlayer-node-linux-amd64-${VERSION}.tar.gz

mkdir -p $HOME/.genlayer
tar -xzvf genlayer-node-linux-amd64-${VERSION}.tar.gz
mv $HOME/genlayer-node-linux-amd64/* $HOME/.genlayer/

cd $HOME/.genlayer
```

### 4.3 ⚙️ GenVM Kurulumunu Çalıştır

```bash
python3 $HOME/.genlayer/third_party/genvm/bin/setup.py
```

> ⏳ Birkaç dakika sürebilir, sabırla bekle.

### 4.4 🔗 Sistem Symlink'i Oluştur

`genlayernode` komutunu her yerden çalıştırabilmek için sistem genelinde erişilebilir hale getiriyoruz.

```bash
sudo ln -sf $HOME/.genlayer/bin/genlayernode /usr/local/bin/genlayernode

# Doğrula
which genlayernode
genlayernode --version
```

---

## 5. Konfigürasyon Dosyasını Oluştur

### 5.1 📁 Gerekli Klasörler

```bash
mkdir -p $HOME/.genlayer/configs/node
mkdir -p $HOME/.genlayer/data
mkdir -p $HOME/.genlayer/logs
```

### 5.2 ✏️ Değerleri Önce Tanımla

```bash
# --- Kendi bilgilerinle doldur ---
VALIDATOR_WALLET="0xVALIDATOR_WALLET_ADRESİN"   # Wizard'dan aldın
OPERATOR_ADDRESS="0xOPERATOR_ADRESİN"            # Wizard'dan aldın
# --------------------------------

echo "Validator : $VALIDATOR_WALLET"
echo "Operator  : $OPERATOR_ADDRESS"
```

> 🌐 **Asimov RPC:**  
> HTTP: `https://zksync-os-testnet-genlayer.zksync.dev/http`  
> WS: `wss://zksync-os-testnet-genlayer.zksync.dev/ws`

### 5.3 📄 config.yaml Oluştur

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

> 💡 Wizard sırasında oluşturulan operator hesabının private key'ini node'a import ediyoruz.

```bash
genlayernode account import \
  --password "BURAYA_NODE_ŞİFRENİ_YAZ" \
  --private-key "0xOPERATOR_PRIVATE_KEY" \
  --name "operator"
```

Başarılı çıktı:

```
Account imported:
  Name: operator
  Address: 0x...
```

> ✅ Görünen adresin wizard'dan aldığın **Operator Adresi** ile eşleştiğini doğrula.

### Hesapları Listele

```bash
genlayernode account list -c $HOME/.genlayer/configs/node/config.yaml
```

### 🔒 Key Yedeği Al

```bash
genlayernode account export \
  --account "operator" \
  --password "YEDEK_KEYSTORE_ŞİFREN" \
  --source-password "NODE_ŞİFREN" \
  --output "$HOME/operator-backup.json"
```

> 🔐 Bu dosyayı güvenli bir yerde sakla! Kaybedersen yeni operator kurman gerekir.

---

## 7. Node'u Servis Olarak Çalıştır

LLM API key'ini node binary'si GenVM modülü üzerinden kullanır.
Key, systemd servisine `Environment=` direktifi ile verilir — ayrı bir `.env` dosyasına gerek yok.

### 7.1 🐳 WebDriver'ı Test Et

```bash
cd $HOME/.genlayer
docker compose up -d

# Container çalışıyor mu?
docker ps
# "genlayer-node-webdriver" görünmeli
```

### 7.2 🩺 Konfigürasyonu Doğrula

```bash
cd $HOME/.genlayer
genlayernode doctor
```

> ⚠️ Tüm kontroller geçmeli. Hata görüyorsan servise geçmeden önce mutlaka düzelt.

### 7.3 📝 systemd Servis Dosyasını Oluştur

> 💡 [Heurist](https://www.heurist.ai/) — ücretsiz API kredisi için [dev-api-form.heurist.ai](https://dev-api-form.heurist.ai) (referral kod: `genlayer`)  
> Başka sağlayıcı kullanıyorsan ilgili `Environment=` satırının başındaki `#` işaretini kaldır ve key'ini gir.

```bash
sudo tee /etc/systemd/system/genlayer-node.service > /dev/null << EOF
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

ExecStart=/usr/local/bin/genlayernode run \
  -c $HOME/.genlayer/configs/node/config.yaml \
  --password \${GENLAYERNODE_PASSWORD}

Restart=on-failure
RestartSec=10
LimitNOFILE=65535
StandardOutput=journal
StandardError=journal
SyslogIdentifier=genlayer-node

[Install]
WantedBy=multi-user.target
EOF
```

### 7.4 🚀 Servisi Başlat

```bash
sudo systemctl daemon-reload
sudo systemctl enable genlayer-node
sudo systemctl start genlayer-node

# Durumu kontrol et
sudo systemctl status genlayer-node
```

### 7.5 📊 Logları Takip Et

```bash
# Canlı log
sudo journalctl -u genlayer-node -f

# Sync durumu
sudo journalctl -u genlayer-node | grep "Node is synced"
# "Node is synced!!! blockNumber=XXXXX" görmelisin
```

---

## 8. Faydalı Komutlar

### ⚙️ Servis Yönetimi

```bash
sudo systemctl stop genlayer-node
sudo systemctl restart genlayer-node
sudo systemctl status genlayer-node
sudo journalctl -u genlayer-node -n 100 --no-pager
```

### 📡 Node Durumu

```bash
curl http://localhost:9153/health
curl http://localhost:9153/metrics
curl http://localhost:9153/balance
```

### 👤 Hesap Komutları

```bash
# Kayıtlı hesapları listele
genlayernode account list -c $HOME/.genlayer/configs/node/config.yaml

# Key yedeği al
genlayernode account export \
  --account "operator" \
  --password "YEDEK_KEYSTORE_ŞİFREN" \
  --source-password "NODE_ŞİFREN" \
  --output "$HOME/operator-backup.json"
```

### 🥩 Validator Yönetimi

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

> ♻️ `configs`, `data` ve `logs` klasörleri korunur. Sadece `bin` ve `third_party` güncellenir.

```bash
export NEW_VERSION=v0.5.13

# 1. Yeni versiyonu indir ve çıkart (kendi klasörüyle gelir: genlayer-node-linux-amd64/)
wget https://storage.googleapis.com/gh-af/genlayer-node/bin/amd64/${NEW_VERSION}/genlayer-node-linux-amd64-${NEW_VERSION}.tar.gz
tar -xzvf genlayer-node-linux-amd64-${NEW_VERSION}.tar.gz

# 2. Yeni GenVM kurulumu (çıkan klasör içinden çalıştır)
python3 $HOME/genlayer-node-linux-amd64/third_party/genvm/bin/setup.py

# 3. Servisi durdur
sudo systemctl stop genlayer-node

# 4. Binary ve GenVM'i güncelle — configs/data/logs dokunulmaz
rm -rf $HOME/.genlayer/bin $HOME/.genlayer/third_party
mv $HOME/genlayer-node-linux-amd64/bin $HOME/.genlayer/
mv $HOME/genlayer-node-linux-amd64/third_party $HOME/.genlayer/
mv $HOME/genlayer-node-linux-amd64/docker-compose.yaml $HOME/.genlayer/

# 5. Symlink güncelle ve versiyonu doğrula
sudo ln -sf $HOME/.genlayer/bin/genlayernode /usr/local/bin/genlayernode
genlayernode --version

# 6. WebDriver'ı yeniden başlat
cd $HOME/.genlayer
docker compose down
docker compose up -d

# 7. Servisi başlat ve logları izle
sudo systemctl start genlayer-node
sudo journalctl -u genlayer-node -f

# 8. Temizlik
rm -rf $HOME/genlayer-node-linux-amd64
rm genlayer-node-linux-amd64-*.tar.gz
```

---

## 10. Sık Yapılan Hatalar

### ❌ exec format error

ARM işlemci kullanıyorsun. AMD64 sunucuya geç.
```bash
uname -m   # x86_64 olmalı
```

### ❌ NO_PROVIDER_FOR_PROMPT

Servis dosyasında LLM API key boş bırakılmış.
```bash
sudo systemctl edit genlayer-node
# Environment="HEURISTKEY=..." satırını güncelle
sudo systemctl restart genlayer-node
```

### ❌ Node validator modunda çalışmıyor

`validatorWalletAddress` veya `operatorAddress` config'de boş.
```bash
nano $HOME/.genlayer/configs/node/config.yaml
sudo systemctl restart genlayer-node
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
docker logs genlayer-node-webdriver
```

### ❌ Quarantine durumu

Yeniden başlatma veya güncelleme sonrası geçici olarak normaldir.
Epoch'lar tamamlandıkça otomatik temizlenir.

---

## 📚 Kaynaklar

| 🔗 | Link |
|----|------|
| 📖 Resmi Kurulum Rehberi | [docs.genlayer.com/validators/setup-guide](https://docs.genlayer.com/validators/setup-guide) |
| ⚙️ GenVM Konfigürasyonu | [docs.genlayer.com/validators/genvm-configuration](https://docs.genlayer.com/validators/genvm-configuration) |
| 🔭 Explorer — Asimov | [genlayer-explorer.vercel.app](https://genlayer-explorer.vercel.app/) |
| 💬 Discord | [discord.gg/8Jm4v89VAu](https://discord.gg/8Jm4v89VAu) |
| 📢 Telegram | [t.me/genlayer](https://t.me/genlayer) |

---

> ✅ Sorun yaşarsan GenLayer Discord `#validators` kanalına sorabilirsin.
