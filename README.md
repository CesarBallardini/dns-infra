# README - Infraestructura para IPAM / DNS / DHCP

En `docs` se encuentra el archivo `THU_D_Bart_Busschots_DHCP_Configuration.pdf`
con el contenido de una presentación: *DHCP, DNS & IP Address Management* 
de Bart Busschots, en la National University of Ireland Maynooth.

Vamos a recrear ese modelo de infraestructura de DNS, mediante Vagrant y Virtualbox.

# Tipos de nodos

* blindmaster: Blind DNS Master
* ipam: php{IPAM}
* localmaster, localslave: Local Auth DNS Cluster 
* globalmaster, globalslave: Global Auth DNS Cluster
* ddiscript: DDI Scripts
* dhcpmaster, dhcpslave: DHCPd cluster
* dnsresolver: DNS resolvers

y para simular las solicitudes de DNS, tendremos dos nodos adicionales,
uno en la red interna y otro en la red externa, para que sus solicitudes 
sean atendidas por los dnsresolver y por el globalcluster, respectivamente.

* testinterna
* testexterna

El nombre del nodo se construye con el tipo como prefijo y un dígito, el primer nodo se asocia con el 0.  
Ejemplo:

* blindmaster0
* localmaster0, localmaster1, etc.
* localslave0, localslave1, etc.
* dnsresolver0, dnsresolver1, etc.


# Redes

* mgmt: red de gestión de los nodos, usada para el aprovisionamiento, respaldos, etc.
* infra: red de servicios de infraestructura, una de muchas redes donde
   residen las VMs que necesitan servicios de infraestructura
* externa: la red que simula los accesos desde la Internet

| red     |            CIDR |
|---------|-----------------|
| mgmt    | 192.168.33.0/24 |
| infra   | 192.168.44.0/24 |
| externa | 192.168.55.0/24 |

# Catálogo de nodos

| nombre       | red   | dirección IP  |
|--------------|-------|---------------|
| blindmaster0 | mgmt  | 192.168.33.10 |
|              | infra | 192.168.44.10 |



# Referencias

* https://jpmens.net/2010/10/29/alternative-dns-servers-the-book-as-pdf/ Alternative DNS Servers by Jan-Piet Mens

