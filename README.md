# babi

````
cat > /usr/bin/babi <<'BABIEOF'
#!/bin/bash
clear
echo "            ALLXRAY USER LOGIN (KONEKSI ASLI)"
echo "└──────────────────────────────────────────┘"
XRAY_LOG="/var/log/xray/access.log"
NGINX_LOG="/var/log/nginx/xray_real.log"

# === NGITUNG KONEKSI, DUDU IP! ===
# Format log baru: IP - [jam] "GET /path" status CONNECTION_ID waktu
# CONNECTION_ID = 1 HP = 1 ID. 2 HP = 2 ID. IP ganti = ID tetep beda per HP, tapi nek reconnect = ID baru!
# Dadi ngitung: jumlah CONNECTION_ID unik neng 60 detik terakhir = jumlah HP!

MENIT1=$(date +"%d/%b/%Y:%H:%M")
MENIT2=$(date -d "-1 minute" +"%d/%b/%Y:%H:%M")
LOG_ALL=$(grep -F "[$MENIT1" "$NGINX_LOG" 2>/dev/null; grep -F "[$MENIT2" "$NGINX_LOG" 2>/dev/null)
if [ -z "$LOG_ALL" ]; then
  LOG_ALL=$(tail -n 100 "$NGINX_LOG" 2>/dev/null)
fi
LOG_ALL=$(echo "$LOG_ALL" | grep -E "/vless|/vmess|/trojan" | grep -v "205\.210\.")

# Coba ngitung koneksi (kolom ke-6 = $connection, nek format lawas, kolom e beda, fallback ke IP)
# Format baru: IP - [jam] "request" status CONNECTION
JUMLAH_KONEKSI=$(echo "$LOG_ALL" | awk '{print $6}' | grep -E "^[0-9]+$" | sort -u | wc -l)
JUMLAH_IP=$(echo "$LOG_ALL" | awk '{print $1}' | grep -E "^[0-9]+\." | sort -u | wc -l)
IPS=$(echo "$LOG_ALL" | awk '{print $1}' | grep -E "^[0-9]+\." | sort -u)
IP_TERAKHIR=$(echo "$LOG_ALL" | awk '{print $1}' | grep -E "^[0-9]+\." | tail -1)

# Nek $connection durung ono (format lawas), ngitung IP tapi kasih warning!
if [ "$JUMLAH_KONEKSI" -eq 0 ]; then
  JUMLAH_KONEKSI=$JUMLAH_IP
  MODE="FORMAT LAWAS - Ngitung IP (kurang akurat, update log_format disik!)"
else
  MODE="FORMAT BARU - Ngitung KONEKSI (akurat!)"
fi

mapfile -t USERS < <(grep -E "email:" "$XRAY_LOG" 2>/dev/null | tail -n 200 | awk -F'email: ' '{print $2}' | awk '{print $1}' | sort -u)

echo ""
echo "Mode : $MODE"
echo "Total Koneksi (60 detik): $JUMLAH_KONEKSI"
echo "Total IP (60 detik): $JUMLAH_IP (IP ganti-ganti, ojo digawe patokan!)"
echo "IP Saat Iki: $IP_TERAKHIR"
echo "Riwayat IP:"
echo "$IPS" | while read ip; do echo "     [REAL] $ip"; done
echo "------------------------------------------"

for user in "${USERS[@]}"; do
  LAST=$(grep -F "email: $user" "$XRAY_LOG" | tail -1)
  JAM=$(echo "$LAST" | awk '{print $1" "$2}')
  echo "user : $user"
  echo "Online Jam : $JAM wib"
  # Nek 1 user, device = jumlah koneksi! Dudu 1!
  # Nek 2 user, device per user = total koneksi / jumlah user (nek GLOBAL)
  if [ "${#USERS[@]}" -eq 1 ]; then
    echo "Device : $JUMLAH_KONEKSI DEVICE"
    if [ "$JUMLAH_KONEKSI" -eq 1 ]; then
      echo "     => 1 HP, 1 koneksi"
    elif [ "$JUMLAH_KONEKSI" -eq 2 ]; then
      echo "     => 2 HP, 2 koneksi! (Senajan 1 user!)"
    else
      echo "     => $JUMLAH_KONEKSI HP!"
    fi
  else
    echo "Device : $JUMLAH_KONEKSI DEVICE TOTAL (GLOBAL, dibagi $(( ${#USERS[@]} )) user)"
  fi
  echo ""
done
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
