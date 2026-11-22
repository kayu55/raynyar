# babi
````
cd /usr/bin
wget -O upraybaru "https://raw.githubusercontent.com/kayu55/raynyar/main/upraybaru.sh"
sed -i 's/\r//g' upraybaru
chmod +x upraybaru
upraybaru
````

````
cat > /usr/bin/babi <<'BABIEOF'
#!/bin/bash
clear
echo  ""
echo "┌───────────────────────────────────────────┐"
echo " ALLXRAY USER LOGIN (KONEKSI ASLI - 1 MENIT)"
echo "└───────────────────────────────────────────┘"
XRAY_LOG="/var/log/xray/access.log"
NGINX_LOG="/var/log/nginx/xray_real.log"
[ ! -f "$NGINX_LOG" ] && NGINX_LOG="/var/log/nginx/access.log"

NOW=$(date +%s)

mapfile -t USERS < <(grep -E "email:" "$XRAY_LOG" 2>/dev/null | tail -n 500 | awk -F'email: ' '{print $2}' | awk '{print $1}' | sort -u)

ADA=0
for user in "${USERS[@]}"; do
  LAST=$(grep -F "email: $user" "$XRAY_LOG" | tail -1)
  [ -z "$LAST" ] && continue
  JAM_LAST=$(echo "$LAST" | awk '{print $1" "$2}')
  # Format Xray: 2026/11/21 11:07:19.096574 -> ambil 2026/11/21 11:07:19
  TGL_JAM=$(echo "$JAM_LAST" | cut -d. -f1)
  LAST_EPOCH=$(date -d "$TGL_JAM" +%s 2>/dev/null || echo 0)
  SELISIH=$((NOW - LAST_EPOCH))

  # === FILTER 1 MENIT: Nek offline > 60 detik, SKIP! Gak ditampilke! ===
  if [ "$SELISIH" -gt 60 ]; then
    continue
  fi

  ADA=1
  JAMS=$(grep -F "email: $user" "$XRAY_LOG" | tail -n 20 | awk '{print $2}' | cut -d. -f1)
  IPS_USER=""
  for JAM in $JAMS; do
    IP=$(grep -F ":$JAM " "$NGINX_LOG" | tail -n 5 | awk '{print $1}' | grep -E "^[0-9]+\." | grep -v "127.0.0.1" | head -1)
    [ -n "$IP" ] && IPS_USER="$IPS_USER\n$IP"
  done
  IPS_USER=$(echo -e "$IPS_USER" | grep -v "^$" | sort -u)
  JUMLAH=$(echo "$IPS_USER" | grep -c .)
  [ -z "$IPS_USER" ] && JUMLAH=0
  IP_TERAKHIR=$(echo "$IPS_USER" | tail -1)

  echo "------------------------------------------"
  echo "user : $user"
  echo "Online Jam : $JAM_LAST wib ($SELISIH detik lalu)"
  echo "IP Saat Iki: $IP_TERAKHIR"
  echo "Riwayat IP (user iki tok):"
  echo "$IPS_USER" | while read ip; do [ -n "$ip" ] && echo "     [REAL] $ip"; done
  echo "Device : $JUMLAH DEVICE"
  if [ "$JUMLAH" -eq 1 ]; then echo "     => 1 HP!"; else echo "     => $JUMLAH HP!"; fi
  echo ""
done

if [ "$ADA" -eq 0 ]; then
  echo "------------------------------------------"
  echo "Gak ono user ONLINE! (Kabeh offline > 1 menit)"
  echo "------------------------------------------"
fi

echo "└──────────────────────────────────────────┘"
echo -e "\033[0;33m┌──────────────────────────────────────────┐\033[0m"
echo -e "      Autoscript By Arya Blitar       "
echo -e "\033[0;33m└──────────────────────────────────────────┘\033[0m"
echo ""
read -n 1 -s -r -p "Press any key to back on menu"
menu
BABIEOF
chmod +x /usr/bin/babi
babi
````
## babi 2
````
cat > /usr/bin/babi <<'BABIEOF'
#!/bin/bash
clear
echo "┌──────────────────────────────────────────┐"
echo " ALLXRAY USER LOGIN (KONEKSI ASLI - 1 MENIT)"
echo "└──────────────────────────────────────────┘"
XRAY_LOG="/var/log/xray/access.log"
NGINX_LOG="/var/log/nginx/xray_real.log"
[ ! -f "$NGINX_LOG" ] && NGINX_LOG="/var/log/nginx/access.log"
NOW=$(date +%s)
mapfile -t USERS < <(grep -E "email:" "$XRAY_LOG" 2>/dev/null | tail -n 500 | awk -F'email: ' '{print $2}' | awk '{print $1}' | sort -u)
TOTAL=0
LIST=""
for u in "${USERS[@]}"; do
  L=$(grep -F "email: $u" "$XRAY_LOG" | tail -1)
  [ -z "$L" ] && continue
  TJ=$(echo "$L" | awk '{print $1" "$2}' | cut -d. -f1)
  EP=$(date -d "$TJ" +%s 2>/dev/null || echo 0)
  D=$((NOW - EP)); [ "$D" -lt 0 ] && D=0
  if [ "$D" -le 60 ]; then TOTAL=$((TOTAL+1)); LIST="$LIST $u"; fi
done
echo ""
echo "TOTAL ONLINE : $TOTAL USER"
echo "USER MASUK   :$LIST"
echo ""
NO=0
for user in "${USERS[@]}"; do
  LAST=$(grep -F "email: $user" "$XRAY_LOG" | tail -1)
  [ -z "$LAST" ] && continue
  JAM_LAST=$(echo "$LAST" | awk '{print $1" "$2}')
  TGL_JAM=$(echo "$JAM_LAST" | cut -d. -f1)
  LAST_EPOCH=$(date -d "$TGL_JAM" +%s 2>/dev/null || echo 0)
  SELISIH=$((NOW - LAST_EPOCH))
  if [ "$SELISIH" -lt 0 ]; then SELISIH=0; fi
  if [ "$SELISIH" -gt 60 ]; then continue; fi
  NO=$((NO+1))
  JAMS=$(grep -F "email: $user" "$XRAY_LOG" | tail -n 20 | awk '{print $2}' | cut -d. -f1)
  IPS_USER=""
  for JAM in $JAMS; do
    IP=$(grep -F ":$JAM " "$NGINX_LOG" | tail -n 5 | awk '{print $1}' | grep -E "^[0-9]+\." | grep -v "127.0.0.1" | head -1)
    [ -n "$IP" ] && IPS_USER="$IPS_USER
$IP"
  done
  IPS_USER=$(printf "%s\n" "$IPS_USER" | grep -v "^$" | sort -u)
  JUMLAH=$(printf "%s\n" "$IPS_USER" | grep -c .)
  [ -z "$IPS_USER" ] && JUMLAH=0
  IP_TERAKHIR=$(printf "%s\n" "$IPS_USER" | tail -1)
  tebak_app() {
    local T="$1"
    case "$T" in
      *google*|*youtube*|*googlevideo*|*gstatic*) echo "GOOGLE/YOUTUBE"; return ;;
      *facebook*|*fbcdn*|*instagram*|*whatsapp*) echo "META (FB/IG/WA)"; return ;;
      *telegram*) echo "TELEGRAM"; return ;;
      *tiktok*|*byte*|*byteoversea*) echo "TIKTOK"; return ;;
      *shopee*) echo "SHOPEE"; return ;;
      *netflix*) echo "NETFLIX"; return ;;
      *spotify*) echo "SPOTIFY"; return ;;
      *vidio*) echo "VIDIO"; return ;;
    esac
    case "$T" in
      142.250.*|172.217.*|172.253.*|216.58.*|74.125.*|173.194.*|64.233.*|142.251.*|108.177.*) echo "GOOGLE/YOUTUBE"; return ;;
      157.*|199.*|57.144.*|57.145.*|31.13.*|179.60.*|185.60.*|129.134.*|199.103.*) echo "META (FB/IG/WA)"; return ;;
      149.154.*|91.108.*|91.105.*) echo "TELEGRAM"; return ;;
      47.89.*|47.251.*|103.152.*|161.117.*|156.59.*|43.240.*|43.241.*) echo "TIKTOK"; return ;;
      35.*|34.*|130.211.*|104.154.*) echo "GOOGLE CLOUD"; return ;;
      15.*|52.*|54.*|3.*|18.*|13.*) echo "AMAZON AWS"; return ;;
      104.*|172.67.*|104.21.*) echo "CLOUDFLARE CDN"; return ;;
      23.*|167.82.*|167.86.*) echo "AKAMAI CDN"; return ;;
      151.101.*|199.232.*) echo "FASTLY CDN"; return ;;
      47.87.*|47.84.*|47.75.*|47.241.*|47.242.*|8.210.*|47.*) echo "ALIBABA CLOUD"; return ;;
      71.18.*|163.181.*|153.43.*|177.0.*|71.*) echo "CDN LUAR"; return ;;
      103.111.*|103.169.*|103.119.*|103.137.*|103.41.*|103.229.*|103.155.*) echo "SERVER INDO - Biznet"; return ;;
      116.68.*|116.90.*|116.66.*|36.*|118.*|114.*|180.*|101.*) echo "SERVER INDO - Telkom"; return ;;
      103.*|116.*|36.*|114.*|180.*|182.*|202.*|61.*) echo "SERVER INDO"; return ;;
      *) echo "LAINNYA ($T)"; return ;;
    esac
  }
  TMPFILE=$(mktemp)
  while IFS= read -r LG; do
    [ -z "$LG" ] && continue
    TUJUAN=$(echo "$LG" | sed -n 's/.*-> //p' | awk '{print $1}' | sed 's/^tcp://;s/^udp://;s/:[0-9]*$//')
    if [ -z "$TUJUAN" ]; then TUJUAN=$(echo "$LG" | sed -n "s/.*email: $user//p" | grep -oE "[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}(:[0-9]+)?|[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+(:[0-9]+)?" | head -1 | sed 's/:[0-9]*$//'); fi
    if [ -z "$TUJUAN" ]; then TUJUAN=$(echo "$LG" | grep -oE "[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+:[0-9]+" | tail -1 | sed 's/:[0-9]*$//'); fi
    [ -z "$TUJUAN" ] && continue
    if [ "$TUJUAN" = "$user" ]; then continue; fi
    if [[ "$TUJUAN" == "127.0.0.1" || "$TUJUAN" == "8.8.8.8" || "$TUJUAN" == "8.8.4.4" || "$TUJUAN" == "1.1.1.1" || "$TUJUAN" == "1.0.0.0" || "$TUJUAN" == "1.0.0.1" || "$TUJUAN" == "146.112.41.2" || "$TUJUAN" == "146.112.41.3" || "$TUJUAN" == "208.67.222.222" || "$TUJUAN" == "208.67.220.220" || "$TUJUAN" == "9.9.9.9" ]]; then continue; fi
    if [[ "$TUJUAN" == 146.112.* || "$TUJUAN" == 208.67.* ]]; then continue; fi
    if [[ "$TUJUAN" == *ntp* || "$TUJUAN" == *canonical* || "$TUJUAN" == *lambda* || "$TUJUAN" == *arpa* ]]; then continue; fi
    if [[ "$TUJUAN" == 185.125.* || "$TUJUAN" == 103.105.* || "$TUJUAN" == 91.189.* || "$TUJUAN" == 162.159.* ]]; then continue; fi
    if [[ "$TUJUAN" == 182.161.* || "$TUJUAN" == 182.4.* || "$TUJUAN" == 140.* || "$TUJUAN" == 203.78.* || "$TUJUAN" == 112.215.* || "$TUJUAN" == 47.241.72.* ]]; then continue; fi
    tebak_app "$TUJUAN" >> "$TMPFILE"
  done < <(grep -F "email: $user" "$XRAY_LOG" | tail -n 30)
  BROWSING_APP=$(sort "$TMPFILE" | uniq -c | sort -nr | awk '{c=$1; $1=""; sub(/^ /,""); print "     - " $0 " (" c "x)"}')
  rm -f "$TMPFILE"
  [ -z "$BROWSING_APP" ] && BROWSING_APP="     - (Gak browsing, mek konek tok)"
  echo "------------------------------------------"
  echo "$NO. user : $user"
  echo "Online Jam : $JAM_LAST wib ($SELISIH detik lalu)"
  echo "IP Saat Iki: $IP_TERAKHIR"
  echo "Riwayat IP (user iki tok):"
  printf "%s\n" "$IPS_USER" | while IFS= read -r ip; do [ -n "$ip" ] && printf "     [REAL] %s\n" "$ip"; done
  echo "Device : $JUMLAH DEVICE (KIRA-KIRA - IP kecampur!)"
  echo "Lagi Buka Opo Ae (KIRA-KIRA teko IP):"
  printf "%s\n" "$BROWSING_APP"
  echo ""
done
if [ "$TOTAL" -eq 0 ]; then
  echo "------------------------------------------"
  echo "Gak ono user ONLINE! (Kabeh offline > 1 menit)"
  echo "------------------------------------------"
fi
echo "------------------------------------------"
echo ""
echo "👉 KAMU NOMER PIRO? KOMEN! 👈"
echo "👉 SENG GAK ONO JENENG E, BERARTI OFFLINE! 👈"
echo ""
echo "      Autoscript By Arya Blitar"
echo ""
read -n 1 -s -r -p "Press any key to back on menu"
menu
BABIEOF
chmod +x /usr/bin/babi
cp /usr/bin/babi /usr/bin/babi-final-bagus
babi
````
## fix
````
# === HAPUS CARA LAWAS IKI ===
# Total IP = cat access.log | grep $user | awk '{print $1}' | sort -u

# === GANTI KARO CARA BARU: CEK SENG JIK KONEK SAAT IKI TOK! ===
# 1. Cek IP seng JIK ESTABLISHED neng port 443 saiki (udu log lawas!)
IP_AKTIF=$(ss -tn state established '( sport = :443 )' | awk '{print $5}' | cut -d: -f1 | sort -u)

# 2. Cek user iki, IP e seng jik konek saiki opo?
# Nek IP lawas wes pedot, otomatis ilang dewe, gak keitung dobel!
echo "$IP_AKTIF" | while read IP; do
  # Cek neng log 5 DETIK terakhir tok, dudu 60 detik!
  grep "$IP" /var/log/nginx/access.log | tail -n 5 | grep -q "$user" && echo "[REAL] $IP"
done | sort -u > /tmp/ip_aktif.txt

DEVICE=$(wc -l < /tmp/ip_aktif.txt)
echo "Device : $DEVICE DEVICE"
````
## upgrade xray
### bckp
````
cp /etc/xray/config.json /root/config.json.BACKUP
cp /etc/xray/config.json /root/config.json.bak-$(date +%F-%H%M)
cat /root/config.json.BACKUP | grep '###' | head -n 20
````
````
systemctl stop xray
systemctl disable xray
rm -rf /usr/local/bin/xray /usr/local/etc/xray /etc/xray/xray
rm -f /etc/systemd/system/xray.service
````
````
apt update -y && apt install -y curl unzip wget
mkdir -p /tmp/xray-baru && cd /tmp/xray-baru
VER=$(curl -s https://api.github.com/repos/XTLS/Xray-core/releases/latest | grep tag_name | cut -d'"' -f4)
echo "Versi terbaru: $VER"
wget https://github.com/XTLS/Xray-core/releases/download/$VER/Xray-linux-64.zip
unzip -o Xray-linux-64.zip
mkdir -p /usr/local/bin /usr/local/etc/xray /var/log/xray
cp xray /usr/local/bin/xray
cp geoip.dat geosite.dat /usr/local/etc/xray/
chmod +x /usr/local/bin/xray
/usr/local/bin/xray version
````
````
mkdir -p /etc/xray
cp /root/config.json.BACKUP /etc/xray/config.json
cat > /etc/systemd/system/xray.service << 'EOF'
[Unit]
Description=Xray Service
After=network.target nss-lookup.target
[Service]
User=root
ExecStart=/usr/local/bin/xray run -config /etc/xray/config.json
Restart=on-failure
RestartSec=3
LimitNOFILE=1048576
[Install]
WantedBy=multi-user.target
EOF
systemctl daemon-reload
systemctl enable xray
systemctl restart xray
systemctl status xray --no-pager -l
````
## cek
````
ss -tlnp | grep -E '443|80|8080'
tail -n 50 /var/log/xray/error.log
babi
````
## fix xray
````
cat > /usr/bin/xray <<'BABIEOF'
#!/bin/bash

NC='\033[0;37m'
green='\033[0;32m' 

clear
clear
source /var/lib/scrz-prem/ipvps.conf
if [[ "$IP" = "" ]]; then
domain=$(cat /etc/xray/domain)
else
domain=$IP
fi
#hpus file lama
systemctl stop xray nginx

rm -fr /etc/nginx/conf.d/xray.conf
rm -fr /etc/systemd/system/xray.service
rm -fr /etc/systemd/system/xray.service.d

echo -e "[ ${green}INFO${NC} ] Checking... "
apt install iptables iptables-persistent -y
sleep 1
echo -e "[ ${green}INFO$NC ] Setting ntpdate"
ntpdate pool.ntp.org 
timedatectl set-ntp true
sleep 1
echo -e "[ ${green}INFO$NC ] Enable chronyd"
systemctl enable chronyd
systemctl restart chronyd
sleep 1
echo -e "[ ${green}INFO$NC ] Enable chrony"
systemctl enable chrony
systemctl restart chrony
timedatectl set-timezone Asia/Jakarta
sleep 1
echo -e "[ ${green}INFO$NC ] Setting chrony tracking"
chronyc sourcestats -v
chronyc tracking -v
echo -e "[ ${green}INFO$NC ] Setting dll"
apt clean all && apt update
apt install curl socat xz-utils wget apt-transport-https gnupg gnupg2 gnupg1 dnsutils lsb-release -y 
apt install socat cron bash-completion ntpdate -y
ntpdate pool.ntp.org
apt -y install chrony
apt install zip -y
apt install curl pwgen openssl netcat cron -y

# install xray
sleep 1
echo -e "[ ${green}INFO$NC ] Downloading & Installing xray core"
domainSock_dir="/run/xray";! [ -d $domainSock_dir ] && mkdir  $domainSock_dir
chown www-data.www-data $domainSock_dir
# Make Folder XRay
mkdir -p /var/log/xray
mkdir -p /etc/xray
chown -R www-data:www-data /var/log/xray
chmod 755 /var/log/xray
touch /var/log/xray/access.log
touch /var/log/xray/error.log
touch /var/log/xray/access2.log
touch /var/log/xray/error2.log
## crt xray
systemctl stop nginx
mkdir /root/.acme.sh
curl https://acme-install.netlify.app/acme.sh -o /root/.acme.sh/acme.sh
chmod +x /root/.acme.sh/acme.sh
/root/.acme.sh/acme.sh --upgrade --auto-upgrade
/root/.acme.sh/acme.sh --set-default-ca --server letsencrypt
/root/.acme.sh/acme.sh --issue -d $domain --standalone -k ec-256
~/.acme.sh/acme.sh --installcert -d $domain --fullchainpath /etc/xray/xray.crt --keypath /etc/xray/xray.key --ecc

# nginx renew ssl
echo -n '#!/bin/bash
/etc/init.d/nginx stop
"/root/.acme.sh"/acme.sh --cron --home "/root/.acme.sh" &> /root/renew_ssl.log
/etc/init.d/nginx start
/etc/init.d/nginx status
' > /usr/local/bin/ssl_renew.sh
chmod +x /usr/local/bin/ssl_renew.sh
if ! grep -q 'ssl_renew.sh' /var/spool/cron/crontabs/root;then (crontab -l;echo "15 03 */3 * * /usr/local/bin/ssl_renew.sh") | crontab;fi

# Make Folder & Log XRay & Log Trojan
rm -fr /var/log/xray
rm -fr /home/vps/public_html
mkdir -p /var/log/xray
mkdir -p /home/vps/public_html
chown www-data.www-data /var/log/xray
chown www-data.www-data /etc/xray
chmod +x /var/log/xray
touch /var/log/xray/access.log
touch /var/log/xray/error.log
touch /var/log/xray/access2.log
touch /var/log/xray/error2.log
chown -R www-data:www-data /var/log/xray
chmod 755 /var/log/xray
chmod 640 /var/log/xray/*.log
# Make Log Autokill & Log Autoreboot
rm -fr /root/log-limit.txt
rm -fr /root/log-reboot.txt
touch /root/log-limit.txt
touch /root/log-reboot.txt
touch /home/limit
echo "" > /root/log-limit.txt
echo "" > /root/log-reboot.txt

# nginx for debian & ubuntu
install_ssl(){
    if [ -f "/usr/bin/apt-get" ];then
            isDebian=`cat /etc/issue|grep Debian`
            if [ "$isDebian" != "" ];then
                    apt-get install -y nginx certbot
                    apt install -y nginx certbot
                    sleep 3s
            else
                    apt-get install -y nginx certbot
                    apt install -y nginx certbot
                    sleep 3s
            fi
    else
        yum install -y nginx certbot
        sleep 3s
    fi
    systemctl stop nginx.service
    if [ -f "/usr/bin/apt-get" ];then
            isDebian=`cat /etc/issue|grep Debian`
            if [ "$isDebian" != "" ];then
                    echo "A" | certbot certonly --renew-by-default --register-unsafely-without-email --standalone -d $domain
                    sleep 3s
            else
                    echo "A" | certbot certonly --renew-by-default --register-unsafely-without-email --standalone -d $domain
                    sleep 3s
            fi
    else
        echo "Y" | certbot certonly --renew-by-default --register-unsafely-without-email --standalone -d $domain
        sleep 3s
    fi
}

# ╔══════════════════════════════════════════════════════════════╗
# ║              📦 INSTALL X-RAY CORE STABLE TERBARU            ║
# ╚══════════════════════════════════════════════════════════════╝
echo -e "\n${green}✦ Mengunduh installer resmi Xray dan memasang versi stabil terbaru...${NC}"

# FIX: GAK USAH TULIS v MANUAL - langsung install NEW, installer resmi yg ambil versi terbaru dari server Xray
if ! bash -c "$(curl -fsSL https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ install; then
    echo -e "\n${NC}❌ Instalasi Xray gagal. Periksa koneksi internet dan log installer."
    exit 1
fi
if [ ! -x /usr/local/bin/xray ]; then
    echo -e "❌ Binary Xray tidak ditemukan di /usr/local/bin/xray. Instalasi dibatalkan.${NC}"
    exit 1
fi
/usr/local/bin/xray version || exit 1
echo -e "[ ${green}INFO${NC} ] Xray NEW berhasil dipasang!"

rm -fr /etc/systemd/system/xray.service.d
rm -fr /etc/systemd/system/xray.service
cat <<EOF> /etc/systemd/system/xray.service
[Unit]
Description=Xray Service
Documentation=https://github.com/XTLS/Xray-core
After=network.target nss-lookup.target

[Service]
User=www-data
CapabilityBoundingSet=CAP_NET_ADMIN CAP_NET_BIND_SERVICE
AmbientCapabilities=CAP_NET_ADMIN CAP_NET_BIND_SERVICE
NoNewPrivileges=true
ExecStart=/usr/local/bin/xray run -config /etc/xray/config.json
Restart=on-failure
RestartPreventExitStatus=23
LimitNPROC=10000
LimitNOFILE=1000000
[Install]
WantedBy=multi-user.target
EOF
echo -e "[ ${GREEN}ok${NC} ] Enable & Start & Restart & Xray"
systemctl daemon-reload >/dev/null 2>&1
systemctl enable xray >/dev/null 2>&1
systemctl start xray >/dev/null 2>&1
systemctl restart xray >/dev/null 2>&1
echo -e "[ ${GREEN}ok${NC} ] Enable & Start & Restart & Nginx"
systemctl daemon-reload >/dev/null 2>&1
systemctl enable nginx >/dev/null 2>&1
systemctl start nginx >/dev/null 2>&1
systemctl restart nginx >/dev/null 2>&1
# Restart All Service
echo -e "$yell[SERVICE]$NC Restart All Service"
chown -R www-data:www-data /home/vps/public_html
# Enable & Restart & Xray & Trojan & Nginx
echo -e "[ ${GREEN}ok${NC} ] Restart & Xray & Nginx"
systemctl daemon-reload >/dev/null 2>&1
systemctl restart xray >/dev/null 2>&1
systemctl restart nginx >/dev/null 2>&1

BABIEOF
chmod +x /usr/bin/xray
xray
````
