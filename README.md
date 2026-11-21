# babi

````
cat > /usr/bin/babi <<'BABIEOF'
#!/bin/bash
clear
echo "            ALLXRAY USER LOGIN (KONEKSI ASLI)"
echo "└──────────────────────────────────────────┘"
XRAY_LOG="/var/log/xray/access.log"
NGINX_LOG="/var/log/nginx/xray_real.log"
[ ! -f "$NGINX_LOG" ] && NGINX_LOG="/var/log/nginx/access.log"

mapfile -t USERS < <(grep -E "email:" "$XRAY_LOG" 2>/dev/null | tail -n 500 | awk -F'email: ' '{print $2}' | awk '{print $1}' | sort -u)

for user in "${USERS[@]}"; do
  JAMS=$(grep -F "email: $user" "$XRAY_LOG" | tail -n 30 | awk '{print $2}' | cut -d. -f1)
  IPS_USER=""
  for JAM in $JAMS; do
    # BUFFER: Golek jam pas + jam +/- 2 detik (ben gak loading 0!)
    for DELTA in 0 1 2 -1 -2; do
      JAM2=$(date -d "today $JAM $DELTA seconds" +%H:%M:%S 2>/dev/null || echo "$JAM")
      IP=$(grep -F ":$JAM2" "$NGINX_LOG" | tail -n 5 | awk '{print $1}' | grep -E "^[0-9]+\." | grep -v "127.0.0.1" | head -1)
      [ -n "$IP" ] && IPS_USER="$IPS_USER\n$IP" && break
    done
  done
  IPS_USER=$(echo -e "$IPS_USER" | grep -v "^$" | sort -u)
  JUMLAH=$(echo "$IPS_USER" | grep -c .)
  [ -z "$IPS_USER" ] && JUMLAH=0
  LAST=$(grep -F "email: $user" "$XRAY_LOG" | tail -1)
  JAM_LAST=$(echo "$LAST" | awk '{print $1" "$2}')
  IP_TERAKHIR=$(echo "$IPS_USER" | tail -1)
  echo "------------------------------------------"
  echo "user : $user"
  echo "Online Jam : $JAM_LAST wib"
  echo "IP Saat Iki: $IP_TERAKHIR"
  echo "Riwayat IP (user iki tok):"
  echo "$IPS_USER" | while read ip; do [ -n "$ip" ] && echo "     [REAL] $ip"; done
  echo "Device : $JUMLAH DEVICE"
  if [ "$JUMLAH" -eq 1 ]; then echo "     => 1 HP!"; else echo "     => $JUMLAH HP!"; fi
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
