# Drosera Operator Node Türkçe Kurulum Rehberi 🇹🇷

Bu rehber, Drosera Operator Node'unu sıfırdan yeni bir sunucuya kurmak isteyenler için hazırlanmıştır.

---

## 1️⃣ Gerekli Bağımlılıklar

```bash
sudo apt-get update && sudo apt-get upgrade -y
sudo apt install curl ufw iptables build-essential git wget lz4 jq make gcc nano automake autoconf tmux htop nvme-cli libgbm1 pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip -y
```

## 2️⃣ Docker Kurulumu

```bash
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg -y
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu \
$(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update -y && sudo apt upgrade -y
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
sudo docker run hello-world
```

## 3️⃣ Drosera CLI ve Foundry Kurulumu

```bash
curl -L https://app.drosera.io/install | bash
source /root/.bashrc
droseraup

curl -L https://foundry.paradigm.xyz | bash
source /root/.bashrc
foundryup

curl -fsSL https://bun.sh/install | bash
```

## 4️⃣ Trap Deploy Etme

```bash
mkdir my-drosera-trap
cd my-drosera-trap
git config --global user.email "GitHub_Emailin"
git config --global user.name "GitHub_KullanıcıAdın"
forge init -t drosera-network/trap-foundry-template
bun install
forge build
DROSERA_PRIVATE_KEY=0xyourprivatekey drosera apply
```

Trap'i whitelistan sonra:

```bash
cd my-drosera-trap
nano drosera.toml
# Dosyaya şunu ekle:
# private_trap = true
# whitelist = ["0xSENIN_OPERATOR_ADRESIN"]

DROSERA_PRIVATE_KEY=0xyourprivatekey drosera apply
```

## 5️⃣ Operator CLI Kurulumu

```bash
cd ~
curl -LO https://github.com/drosera-network/releases/releases/download/v1.16.2/drosera-operator-v1.16.2-x86_64-unknown-linux-gnu.tar.gz
tar -xvf drosera-operator-v1.16.2-x86_64-unknown-linux-gnu.tar.gz
sudo cp drosera-operator /usr/bin
drosera-operator --version
```

🔁 systemd ile Drosera Node’u Arka Planda Çalıştırma (Detaylı)
Aşağıdaki komutu çalıştırmadan önce şu değişkenleri kendi bilgilerinle doldur:

PV_KEY → EVM cüzdan private key’in

VPS_IP → Sunucunun IP adresi (ya da localhost)

```bash
sudo tee /etc/systemd/system/drosera.service > /dev/null <<EOF
[Unit]
Description=drosera node service
After=network-online.target

[Service]
User=$USER
Restart=always
RestartSec=15
LimitNOFILE=65535
ExecStart=$(which drosera-operator) node --db-file-path $HOME/.drosera.db --network-p2p-port 31313 --server-port 31314 \
    --eth-rpc-url https://ethereum-holesky-rpc.publicnode.com \
    --eth-backup-rpc-url https://1rpc.io/holesky \
    --drosera-address 0xea08f7d533C2b9A62F40D5326214f39a8E3A32F8 \
    --eth-private-key PV_KEY \
    --listen-address 0.0.0.0 \
    --network-external-p2p-address VPS_IP \
    --disable-dnr-confirmation true

[Install]
WantedBy=multi-user.target
EOF
```

![fe4a5f15986d9999cde644f16814154b](https://github.com/user-attachments/assets/4001826f-918c-40a5-8cc2-f3b4a2b0e5e2)

