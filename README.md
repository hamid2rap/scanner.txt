  GNU nano 8.7                           ./hamid_scanner.sh
#!/data/data/com.termux/files/usr/bin/bash

GREEN="\e[92m"
RED="\e[91m"
BLUE="\e[94m"
YELLOW="\e[93m"
CYAN="\e[96m"
RESET="\e[0m"

clear
echo "Paste your IP list, then press Ctrl + D"
echo "----------------------------------------"

test_port() {
    ip=$1
    port=$2
    curl --connect-timeout 1 -s "http://$ip:$port" >/dev/null
    [ $? -eq 0 ] && echo "open" || echo "closed"
}

http_ping() {
    ip=$1
    t=$(curl -o /dev/null -s -w "%{time_total}" --connect-timeout 1 "http://$ip")
    if [ $? -eq 0 ]; then
        echo "$t"
    else
        echo ""
    fi
}

while read -r ip; do
    ip=$(echo "$ip" | tr -d ' \t\r')
    [ -z "$ip" ] && continue

    if ! echo "$ip" | grep -Eq '^[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+$'; then
        continue
    fi

    # ICMP ping
    out=$(ping -c1 -W1 "$ip" 2>/dev/null)

    if echo "$out" | grep -q "time="; then
        rtt=$(echo "$out" | grep -o "time=[0-9.]*" | cut -d= -f2)
        ping_status="${GREEN}Alive${RESET}"
    else
        # HTTP fallback ping
        hrtt=$(http_ping "$ip")
        if [ -n "$hrtt" ]; then
            rtt=$(printf "%.2f" "$(echo "$hrtt * 1000" | bc)")
            ping_status="${GREEN}Alive (HTTP)${RESET}"
        else
            ping_status="${RED}No Ping${RESET}"
            rtt="---"
        fi
    fi

    # Port tests
    p80=$(test_port "$ip" 80)
    p443=$(test_port "$ip" 443)
    p8080=$(test_port "$ip" 8080)

    [ "$p80" = "open" ] && p80="${GREEN}open${RESET}" || p80="${RED}closed${RESET}"
    [ "$p443" = "open" ] && p443="${GREEN}open${RESET}" || p443="${RED}closed${RESET}"
    [ "$p8080" = "open" ] && p8080="${GREEN}open${RESET}" || p8080="${RED}closed${RESET}"

    echo -e "${BLUE}+--------------------------------------+${RESET}"
    printf "| IP: %-33s |\n" "$ip"
    printf "| Ping: %-30s |\n" "$ping_status ($rtt ms)"
    printf "| Port 80: %-26s |\n" "$p80"
    printf "| Port 443: %-25s |\n" "$p443"
    printf "| Port 8080: %-24s |\n" "$p8080"
    echo -e "${BLUE}+--------------------------------------+${RESET}"
    echo

done 
