### Escuela Colombiana de Ingeniería
### Arquitecturas de Software - ARSW

## Escalamiento en Azure con Maquinas Virtuales, Sacale Sets y Service Plans

### Dependencias
* Cree una cuenta gratuita dentro de Azure. Para hacerlo puede guiarse de esta [documentación](https://azure.microsoft.com/en-us/free/search/?&ef_id=Cj0KCQiA2ITuBRDkARIsAMK9Q7MuvuTqIfK15LWfaM7bLL_QsBbC5XhJJezUbcfx-qAnfPjH568chTMaAkAsEALw_wcB:G:s&OCID=AID2000068_SEM_alOkB9ZE&MarinID=alOkB9ZE_368060503322_%2Bazure_b_c__79187603991_kwd-23159435208&lnkd=Google_Azure_Brand&dclid=CjgKEAiA2ITuBRDchty8lqPlzS4SJAC3x4k1mAxU7XNhWdOSESfffUnMNjLWcAIuikQnj3C4U8xRG_D_BwE). Al hacerlo usted contará con $200 USD para gastar durante 1 mes.

### Parte 0 - Entendiendo el escenario de calidad

Adjunto a este laboratorio usted podrá encontrar una aplicación totalmente desarrollada que tiene como objetivo calcular el enésimo valor de la secuencia de Fibonnaci.

**Escalabilidad**
Cuando un conjunto de usuarios consulta un enésimo número (superior a 1000000) de la secuencia de Fibonacci de forma concurrente y el sistema se encuentra bajo condiciones normales de operación, todas las peticiones deben ser respondidas y el consumo de CPU del sistema no puede superar el 70%.

### Parte 1 - Escalabilidad vertical

1. Diríjase a el [Portal de Azure](https://portal.azure.com/) y a continuación cree una maquina virtual con las características básicas descritas en la imágen 1 y que corresponden a las siguientes:
    * Resource Group = SCALABILITY_LAB
    * Virtual machine name = VERTICAL-SCALABILITY
    * Image = Ubuntu Server 
    * Size = Standard B1ls
    * Username = scalability_lab
    * SSH publi key = Su llave ssh publica

![Imágen 1](images/part1/part1-vm-basic-config.png)

2. Para conectarse a la VM use el siguiente comando, donde las `x` las debe remplazar por la IP de su propia VM.

    `ssh scalability_lab@xxx.xxx.xxx.xxx`

3. Instale node, para ello siga la sección *Installing Node.js and npm using NVM* que encontrará en este [enlace](https://linuxize.com/post/how-to-install-node-js-on-ubuntu-18.04/).
4. Para instalar la aplicación adjunta al Laboratorio, suba la carpeta `FibonacciApp` a un repositorio al cual tenga acceso y ejecute estos comandos dentro de la VM:

    `git clone <your_repo>`

    `cd <your_repo>/FibonacciApp`

    `npm install`

5. Para ejecutar la aplicación puede usar el comando `npm FibinacciApp.js`, sin embargo una vez pierda la conexión ssh la aplicación dejará de funcionar. Para evitar ese compartamiento usaremos *forever*. Ejecute los siguientes comando dentro de la VM.

    `npm install forever -g`

    `forever start FibinacciApp.js`

6. Antes de verificar si el endpoint funciona, en Azure vaya a la sección de *Networking* y cree una *Inbound port rule* tal como se muestra en la imágen. Para verificar que la aplicación funciona, use un browser y user el endpoint `http://xxx.xxx.xxx.xxx:3000/fibonacci/6`. La respuesta debe ser `The answer is 8`.

![](images/part1/part1-vm-3000InboudRule.png)

7. La función que calcula en enésimo número de la secuencia de Fibonacci está muy mal construido y consume bastante CPU para obtener la respuesta. Usando la consola del Browser documente los tiempos de respuesta para dicho endpoint usando los siguintes valores:
    * 1000000
    * 1010000
    * 1020000
    * 1030000
    * 1040000
    * 1050000
    * 1060000
    * 1070000
    * 1080000
    * 1090000    

8. Dírijase ahora a Azure y verifique el consumo de CPU para la VM. (Los resultados pueden tardar 5 minutos en aparecer).

![Imágen 2](images/part1/part1-vm-cpu.png)

9. Ahora usaremos Postman para simular una carga concurrente a nuestro sistema. Siga estos pasos.
    * Instale newman con el comando `npm install newman -g`. Para conocer más de Newman consulte el siguiente [enlace](https://learning.getpostman.com/docs/postman/collection-runs/command-line-integration-with-newman/).
    * Diríjase hasta la ruta `FibonacciApp/postman` en una maquina diferente a la VM.
    * Para el archivo `[ARSW_LOAD-BALANCING_AZURE].postman_environment.json` cambie el valor del parámetro `VM1` para que coincida con la IP de su VM.
    * Ejecute el siguiente comando.

    ```
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
    ```

10. La cantidad de CPU consumida es bastante grande y un conjunto considerable de peticiones concurrentes pueden hacer fallar nuestro servicio. Para solucionarlo usaremos una estrategia de Escalamiento Vertical. En Azure diríjase a la sección *size* y a continuación seleccione el tamaño `B2ms`.

![Imágen 3](images/part1/part1-vm-resize.png)

11. Una vez el cambio se vea reflejado, repita el paso 7, 8 y 9.
12. Evalue el escenario de calidad asociado al requerimiento no funcional de escalabilidad y concluya si usando este modelo de escalabilidad logramos cumplirlo.
13. Vuelva a dejar la VM en el tamaño inicial para evitar cobros adicionales.

**Preguntas**

1. ¿Cuántos y cuáles recursos crea Azure junto con la VM?
   
      Azure crea 6 recursos junto con la VM
      1. **Virtual Network**
      2. **Storage Account**
      3. **Public IP Address**
      4. **Network Security Group**
      5. **Network Interface**
      6. **Disk**
      
2. ¿Brevemente describa para qué sirve cada recurso?
   

   1. **Virtual Network**: Permite que muchos tipos de recursos de Azure, como las Máquinas Virtuales Azure (VM), se comuniquen de forma segura entre sí,con el Internet y con redes locales on-premise.
   2. **Storage Account**: Contiene todos los objetos de datos de Azure Storage: bloques, archivos, colas, tablas y discos.
   3. **Public IP Address**: Direcciones IP públicas para los recursos de Azure para comunicarse con otros recursos, con el Internet y con redes locales on-premise.
   4. **Network Security Group**: Para filtrar el tráfico de red hacia y desde los recursos Azure en una red virtual Azure con un grupo de seguridad de red.
   5. **Network Interface**: Una interfaz de red permite que una máquina virtual Azure se comunique con el Internet y con recursos locales.
   6. **Disk**: Almacenamiento de la máquina virtual Azure.
3. ¿Al cerrar la conexión ssh con la VM, por qué se cae la aplicación que ejecutamos con el comando `npm FibonacciApp.js`? ¿Por qué debemos crear un *Inbound port rule* antes de acceder al servicio?

* Al acceder a la máquina virtual mediante SSH, se inicia un proceso para este servicio y todos los comandos realizados por SSH se ejecutarán como hijos de dicho proceso, es por eso por lo que al salir de la sesión SSH, este proceso termina y por ende también acaba los procesos hijos.

Se debe crear un “Inbound port rule” para permitir el tráfico de la red a un número de puerto específico TCP o UDP perteneciente a la máquina virtual, en este caso el puerto 3000.
4. Adjunte tabla de tiempos e interprete por qué la función tarda tando tiempo.

      Tiempo -- Disco `A0`
    |    N     | Fibonacci(N)|
    |:--------:|:-----------:|
    |1000000   |    163.31s  |
    |1010000   |    166.71s  |
    |1020000   |    145.56s  |
    |1030000   |    151.14s  |
    |1040000   |    152.83s  |
    |1050000   |    159.29s  |
    |1060000   |    158.13s  |
    |1070000   |    158.06s  |
    |1080000   |    162.49s  |
    |1090000   |    162.80s  |
    
      Tiempo -- Disco `A6`
    |    N     | Fibonacci(N)|
    |:--------:|:-----------:|
    |1000000   |    70.63s   |
    |1010000   |    71.52s   |
    |1020000   |    72.62s   |
    |1030000   |    74.91s   |
    |1040000   |    75.12s   |
    |1050000   |    79.40s   |
    |1060000   |    79.39s   |
    |1070000   |    79.70s   |
    |1080000   |    82.37s   |
    |1090000   |    82.78s   |
5. Adjunte imágen del consumo de CPU de la VM e interprete por qué la función consume esa cantidad de CPU.
AO
![CPUA0](https://user-images.githubusercontent.com/48091585/78277007-6fc30b00-74d9-11ea-82e6-c0c21db9c95e.png)

A6
![CPUA6](https://user-images.githubusercontent.com/48091585/78276913-51f5a600-74d9-11ea-8d20-146afa102bd4.png)

Fibonacci es un algoritmo que es exhaustivo para el procesador, al implementar la función sin ningún tipo memorización o cache y sobre un procesador de bajas especificaciones, el consumo de este se vuelve evidente, como se muestra en las imagenes anteriores.

6. Adjunte la imagen del resumen de la ejecución de Postman. Interprete:
    * Tiempos de ejecución de cada petición.
    * Si hubo fallos documentelos y explique.
    ![postman environment](https://user-images.githubusercontent.com/48091585/78277755-9e8db100-74da-11ea-9337-245bffe8449b.png)
7. ¿Cuál es la diferencia entre los tamaños `B2ms` y `B1ls` (no solo busque especificaciones de infraestructura)?

El disco B1ls esta solamente disponible en Linux y es más económico que el disco B2ms.

    |Name | vCPU | Memory (GiB) | Temp Storage (SSD) GiB | Base CPU Perf of VM | Max CPU Perf of VM | Max NICs|
    |:---:|:----:|:------------:|:----------------------:|:-------------------:|:------------------:|:-------:|
    |B1ls |  1   |     0.5      |            4           |          5%         |         100%       |    2    |
    |B2ms |  2   |     8        |            16          |          60%        |         200%       |    3    |
    
8. ¿Aumentar el tamaño de la VM es una buena solución en este escenario?, ¿Qué pasa con la FibonacciApp cuando cambiamos el tamaño de la VM?

   Para este escenario puede ser una buena solución si se desea aumentar la cantidad de peticiones que se pueden realizar concurrentemente al servidor, sin embargo, si se desea aumentar el tiempo de respuesta, se debe buscar una mejor manera de implementar la aplicación FibonacciApp.js.
   
9. ¿Qué pasa con la infraestructura cuando cambia el tamaño de la VM? ¿Qué efectos negativos implica?

   Cuando se cambia el tamaño de la máquina virtual implica que esta se tenga que reiniciar, por lo que la aplicación se detiene y la disponibilidad disminuye. Al iniciar la máquina se debe volver a empezar el servicio de FibonacciApp. 
   
10. ¿Hubo mejora en el consumo de CPU o en los tiempos de respuesta? Si/No ¿Por qué?

   Hubo mejora en el consumo de CPU, en los tiempos prácticamente se mantuvo igual con ambos discos. La mejora se debe a que el disco tiene mayor capacidad de procesamiento, una vCPU más,  que permite que las conexiones no se cierren y la aplicación soporte más cálculos.
   
11. Aumente la cantidad de ejecuciones paralelas del comando de postman a `4`. ¿El comportamiento del sistema es porcentualmente mejor?

![paralelo](https://user-images.githubusercontent.com/48091585/78624349-03386b00-784f-11ea-8ac7-2f165d7ed5fd.png)

![cpu paralelo](https://user-images.githubusercontent.com/48091585/78624548-835ed080-784f-11ea-8ae5-31e7a18a1937.png)



### Parte 2 - Escalabilidad horizontal

#### Crear el Balanceador de Carga

Antes de continuar puede eliminar el grupo de recursos anterior para evitar gastos adicionales y realizar la actividad en un grupo de recursos totalmente limpio.

1. El Balanceador de Carga es un recurso fundamental para habilitar la escalabilidad horizontal de nuestro sistema, por eso en este paso cree un balanceador de carga dentro de Azure tal cual como se muestra en la imágen adjunta.

![](images/part2/part2-lb-create.png)

2. A continuación cree un *Backend Pool*, guiese con la siguiente imágen.

![](images/part2/part2-lb-bp-create.png)

3. A continuación cree un *Health Probe*, guiese con la siguiente imágen.

![](images/part2/part2-lb-hp-create.png)

4. A continuación cree un *Load Balancing Rule*, guiese con la siguiente imágen.

![](images/part2/part2-lb-lbr-create.png)

5. Cree una *Virtual Network* dentro del grupo de recursos, guiese con la siguiente imágen.

![](images/part2/part2-vn-create.png)

#### Crear las maquinas virtuales (Nodos)

Ahora vamos a crear 3 VMs (VM1, VM2 y VM3) con direcciones IP públicas standar en 3 diferentes zonas de disponibilidad. Después las agregaremos al balanceador de carga.

1. En la configuración básica de la VM guíese por la siguiente imágen. Es importante que se fije en la "Avaiability Zone", donde la VM1 será 1, la VM2 será 2 y la VM3 será 3.

![](images/part2/part2-vm-create1.png)

2. En la configuración de networking, verifique que se ha seleccionado la *Virtual Network*  y la *Subnet* creadas anteriormente. Adicionalmente asigne una IP pública y no olvide habilitar la redundancia de zona.

![](images/part2/part2-vm-create2.png)

3. Para el Network Security Group seleccione "avanzado" y realice la siguiente configuración. No olvide crear un *Inbound Rule*, en el cual habilite el tráfico por el puerto 3000. Cuando cree la VM2 y la VM3, no necesita volver a crear el *Network Security Group*, sino que puede seleccionar el anteriormente creado.

![](images/part2/part2-vm-create3.png)

4. Ahora asignaremos esta VM a nuestro balanceador de carga, para ello siga la configuración de la siguiente imágen.

![](images/part2/part2-vm-create4.png)

5. Finalmente debemos instalar la aplicación de Fibonacci en la VM. para ello puede ejecutar el conjunto de los siguientes comandos, cambiando el nombre de la VM por el correcto

```
git clone https://github.com/daprieto1/ARSW_LOAD-BALANCING_AZURE.git

curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.34.0/install.sh | bash
source /home/vm1/.bashrc
nvm install node

cd ARSW_LOAD-BALANCING_AZURE/FibonacciApp
npm install

npm install forever -g
forever start FibonacciApp.js
```

Realice este proceso para las 3 VMs, por ahora lo haremos a mano una por una, sin embargo es importante que usted sepa que existen herramientas para aumatizar este proceso, entre ellas encontramos Azure Resource Manager, OsDisk Images, Terraform con Vagrant y Paker, Puppet, Ansible entre otras.

#### Probar el resultado final de nuestra infraestructura

1. Porsupuesto el endpoint de acceso a nuestro sistema será la IP pública del balanceador de carga, primero verifiquemos que los servicios básicos están funcionando, consuma los siguientes recursos:

```
http://52.155.223.248/
http://52.155.223.248/fibonacci/1
```

2. Realice las pruebas de carga con `newman` que se realizaron en la parte 1 y haga un informe comparativo donde contraste: tiempos de respuesta, cantidad de peticiones respondidas con éxito, costos de las 2 infraestrucruras, es decir, la que desarrollamos con balanceo de carga horizontal y la que se hizo con una maquina virtual escalada.

3. Agregue una 4 maquina virtual y realice las pruebas de newman, pero esta vez no lance 2 peticiones en paralelo, sino que incrementelo a 4. Haga un informe donde presente el comportamiento de la CPU de las 4 VM y explique porque la tasa de éxito de las peticiones aumento con este estilo de escalabilidad.

```
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
```

**Preguntas**

* ¿Cuáles son los tipos de balanceadores de carga en Azure y en qué se diferencian?, ¿Qué es SKU, qué tipos hay y en qué se diferencian?, ¿Por qué el balanceador de carga necesita una IP pública?

   * Existen dos tipos de balanceadores, Public Load Balance e Internal Load Balancer, Public Load Balancer asigna la dirección IP pública y el puerto del tráfico entrante a la dirección IP privada y al puerto de la máquina virtual, mientras que un Internal Load Balancer dirige el tráfico sólo a los recursos que están contenidos en la red virtual.

   * SKU representa una unidad de mantenimiento de existencias (Stock Keeping Unit) comprable bajo un producto. Los tipos que existen son el básico y el estandar. El SKU estándar tiene más características que el básico, unas cuantas diferencias son el soporte de mayor cantidad de instancias, soporte del protocolo HTTPS en sus Health Probes, la mayoría de las operaciones se realiza en menos de 30 segundos, pero no es gratis como el SKU básico.

   * Un balanceador de carga necesita una IP pública por que actúa como el único punto con el que los clientes interactúan con la aplicación, es el encargado de distribuir el tráfico entre varios nodos disponibles

* ¿Cuál es el propósito del *Backend Pool*?

   * El backend pool se refiere a un conjunto de backends que reciben un tráfico similar para su aplicación y responden con el comportamiento esperado, su propósito es definir como los diferentes backends se deben evaluar mediante los health probes.

* ¿Cuál es el propósito del *Health Probe*?

   *Se envían periódicamente peticiones de sonde HTTP/HTTPS a cada backend configurado, con el propósito de determinar proximidad y salud de los backends para equilibrar las cargas de las solicitudes de los usuarios.

* ¿Cuál es el propósito de la *Load Balancing Rule*? ¿Qué tipos de sesión persistente existen, por qué esto es importante y cómo puede afectar la escalabilidad del sistema?.

   * El propósito de la Load Balancing Rule es distribuir el tráfico que llega al front hacia los backend pools.

   * En Azure existen tres tipos de sesión de persistencia:
      1. None (hash-based): Especifica que las solicitudes sucesivas del mismo cliente pueden ser manejadas por cualquier máquina virtual.
      2. Client IP (source IP affinity 2-tuple): Especifica que las peticiones sucesivas de la misma dirección IP del cliente serán gestionadas por la misma máquina virtual.
      3. Client IP and Protocol (Source IP affinity 3-tuple): Especifica que las solicitudes sucesivas de la misma combinación de dirección IP de cliente y protocolo serán tratadas por la misma máquina virtual.

   * Esto es importante porque indica cómo las peticiones deben comportarse al momento de llegar a el balanceador de carga, dependiendo de su configuración decide si esa petición se va a una máquina especifica o a una aleatoria. Afecta la escalabilidad en el sentido de que la configuración debe estar dada por el tipo de aplicación y servicio que se está ofreciendo, por ejemplo, en la aplicación de FibonacciApp.js no es necesario que una máquina mantenga una sesión con una IP específica, cualquier máquina puede atender la petición sin inconvenientes, por lo cual su escalabilidad es excelente. Para aplicaciones en las que se mantiene una sesión con la IP del cliente, se genera cierta dependencia y la escalabilidad se ve afectada.

* ¿Qué es una *Virtual Network*? ¿Qué es una *Subnet*? ¿Para qué sirven los *address space* y *address range*?

   * Una Virtual Network es una representación de una red propia en la nube. Es un aislamiento lógico de la nube de Azure dedicada a la suscripción del usuario. Puede utilizar VNets para aprovisionar y gestionar redes virtuales privadas (VPNs) en Azure y, opcionalmente, enlazar las VNets con otras VNets en Azure, o con una infraestructura de TI local.

   * Una Subnet es un rango de direcciones lógicas. Al mantener una red de gran tamaño, es una buena estrategia dividirla en subredes para reducir el tamaño de dominios de broadcast y hacerla más fácil de administrar. En Azure la Subnet permite segmentar la red virtual en una o más subredes y asignar una parte del espacio de direcciones de la Virtual Network a cada subred.

   * Un address space especifica un espacio de direcciones IP privado personalizado utilizando direcciones públicas y privadas. Azure asigna a los recursos en una red virtual una dirección IP privada desde el espacio de direcciones que se asigne.

   * Un address range es el rango de direcciones IP que se define a partir del address space.

* ¿Qué son las *Availability Zone* y por qué seleccionamos 3 diferentes zonas?. ¿Qué significa que una IP sea *zone-redundant*?

   *Las Availability Zone son promesas de disponibilidad que protegen las aplicaciones y datos de los fallos del centro de datos. Las Avalilability Zones son lugares físicos únicos dentro de una región de Azure.

   * Se seleccionaron tres zonas de diferentes para no centralizar la aplicación en caso tal de que una de las zonas caiga debido a cualquier tipo de imprevisto.

   * Que una IP sea zone redundant significa que todos los flujos entrantes o salientes son atendidos por múltiples zonas de disponibilidad en una región simultáneamente utilizando una esa sola dirección de IP.

* ¿Cuál es el propósito del *Network Security Group*?

   * Su propósito es filtrar el tráfico de red hacia y desde los recursos Azure en una red virtual Azure con un grupo de seguridad de red. Continene reglas de seguridad que permiten o niegan el tráfico entrante a la red.

* Pruebas de servicios basicos (Punto 1)

   ![prueba load balancer hola](https://user-images.githubusercontent.com/48091585/78839080-444e8d80-79bd-11ea-8543-0deb7f07cffa.png)
   ![prueba load balancer](https://user-images.githubusercontent.com/48091585/78839079-444e8d80-79bd-11ea-88f1-f28d683e9b30.png)

* Informe de newman 1 (Punto 2)
   ![informe newman](https://user-images.githubusercontent.com/48091585/78839081-444e8d80-79bd-11ea-9d52-a93e812e8313.png)

    |Escalamiento|  SO  | Tipo |  Nivel | Name |vCPU|RAM (GiB)|Discos|Costo XMes|Tiempo promedio de respuesta|Peticiones Exitosas|
    |:----------:|:----:|:----:|:------:|:----:|:--:|:-------:|:----:|:--------:|:--------------------------:|:-----------------:|
    |  Vertical  |Linux |Ubuntu|Estandar|  A6  | 4  |    28   |  8   | $321.20  |             1m7s           |         20        |
    | Horizontal |Linux |Ubuntu|Estandar|F2s_v2| 2  |     4   |  4   |  $61.76  |             16.8s          |         19        |


* Informe de newman 1 (Punto 3)

![vm1newman](https://user-images.githubusercontent.com/48091585/78839083-44e72400-79bd-11ea-9707-2e657b63213a.png)
![vm2newman](https://user-images.githubusercontent.com/48091585/78839084-44e72400-79bd-11ea-9be4-45b774d6589f.png)
![vm3newman](https://user-images.githubusercontent.com/48091585/78839086-44e72400-79bd-11ea-99a1-c6ceca1931ea.png)
![vm4newman](https://user-images.githubusercontent.com/48091585/78839089-457fba80-79bd-11ea-8299-601df72a4450.png)
![cpuvm1newman](https://user-images.githubusercontent.com/48091585/78839090-457fba80-79bd-11ea-8675-defaf7b4eae7.png)
![cpuvm2newman](https://user-images.githubusercontent.com/48091585/78839091-457fba80-79bd-11ea-8b47-96d38c0d4b2b.png)
![cpuvm3newman](https://user-images.githubusercontent.com/48091585/78839076-43b5f700-79bd-11ea-9a29-77a49c8ae356.png)
![cpuvm4newman](https://user-images.githubusercontent.com/48091585/78839077-43b5f700-79bd-11ea-90f2-07a59e4a34af.png)

   * Con este estilo de escalabilidad, fallaron por proceso newman dos o tres peticiones. El éxito de estas peticiones es principalmente dado a que el balanceador de carga permite equilibrar las carga sobre las aplicaciones en las cuatro máquinas virtuales desplegadas, como se aprecia en las imágenes, dejando que estas calculen los resultados de las peticiones sin sobrecargarse.


* Presente el Diagrama de Despliegue de la solución.
![Diagrama de despliegue](https://user-images.githubusercontent.com/48091585/78839059-3567db00-79bd-11ea-9d94-0aca28f61024.png)




