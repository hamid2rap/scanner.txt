#!/data/data/com.termux/files/usr/bin/bash

echo "Paste your IP list, then press Ctrl + D to start scanning"
echo "---------------------------------------------------------"

while read -r ip; do
    ip=$(echo "$ip" | tr -d ' \t\r')

    [ -z "$ip" ] && continue

    if ! echo "$ip" | grep -Eq '^[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+$'; then
        continue
    fi

    out=$(ping -c1 -W1 "$ip" 2>/dev/null | grep "time=")

    if [ -n "$out" ]; then
        rtt=$(echo "$out" | sed -n 's/.*time=\([0-9.]*\).*/\1/p')

        echo "+----------------------+"
        printf "| IP: %-16s |\n" "$ip"
        printf "| Ping: %-13s |\n" "$rtt ms"
        echo "+----------------------+"
        echo
    fi
done
