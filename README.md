# README - Infraestructura para IPAM / DNS / DHCP

En `docs` se encuentra el archivo `THU_D_Bart_Busschots_DHCP_Configuration.pdf`
con el contenido de una presentación: *DHCP, DNS & IP Address Management* 
de Bart Busschots, en la National University of Ireland Maynooth[^1].

Vamos a recrear ese modelo de infraestructura de DNS, mediante Vagrant y Virtualbox.

# Tipos de nodos

* blindmaster: Blind DNS Master
* ipam: php{IPAM}
* localmaster, localslave: Local Auth DNS Cluster 
* globalmaster, globalslave: Global Auth DNS Cluster
* ddiscript: DDI Scripts
* dhcpmaster, dhcpslave: DHCPd cluster
* dnsresolver: DNS resolvers
* mysqlmaster, mysqlslave: cluster de base de datos para almacenar info de la infraestructura

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

[^1]: https://conferences.heanet.ie/assets/files/THU_D_Bart_Busschots_DHCP_Configuration.pdf

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

| nombre       | red   | dirección IP   | descripción                                         |
|--------------|-------|----------------|-----------------------------------------------------|
| blindmaster0 | nat   | 10.0.2.15      | NAT VB network: internet access                     |
|              | mgmt  | 192.168.33.10  | management network: provision, ssh                  |
|              | infra | 192.168.44.10  | infrastructure network: service network for clients |
|              |       |                |                                                     |
| ipam0        | mgmt  | 192.168.33.20  | servidor {php}IPAM                                  |
|              | mgmt  | 192.168.44.20  |                                                     |
|              |       |                |                                                     |
| testinterna0 | mgmt  | 192.168.33.100 | VM con acceso a la red infra para testing           |
|              | mgmt  | 192.168.44.100 |                                                     |
|              |       |                |                                                     |
| mysqlmaster0 | mgmt  | 192.168.33.110 | MySQL master para PowerDNS en blindmaster0          |
|              |       |                |                                                     |



# Que está funcionando en este momento:

* `blindmaster0` tiene instalados:
  * PowerDNS: servidor master de la infraestructura de DNS de la organización; accede a `mysqlmaster0` 
    para almacenar la info de DNS
  * nsedit: la interfaz Web para los técnicos que gestionan la infraestructura de DNS
* `ipam0`: el servidor de {php]IPAM para delegar la gestión de nombres en las áreas usuarias; accede a `mysqlmaster0`
    para almacenar la info de DNS
* `testinterna0` puede consultar registros de DNS en `blindmaster0`
* `mysqlmaster0` es el MySQL server para almacenar los datos de PowerDNS en `blindmaster0`

A medida que vaya avanzando, iré actualizando esta sección.  La meta es replicar la arquitectura
que se describió en la presentación irlandesa.

# Uso de este repositorio

## Instale los requisitos para que funcione ansible

```bash
sudo apt-get install -y python3-pip
sudo -H python3 -m pip install --upgrade pip setuptools wheel
sudo -H python3 -m pip install --upgrade ansible
```

Verificado con Ansible versión 2.8.6, python version 3.6.8

## Clone el repo

```bash
git clone https://github.com/CesarBallardini/dns-infra
cd  dns-infra/
```

## Instalar módulos requeridos para los roles Ansible

```bash
mkdir provision/roles/
ansible-galaxy install -r requirements.yml --roles-path=provision/roles/
```


## Levantar la infraestructura

```bash
time vagrant up
```

Instalar el nodo `ipam0` según [instrucciones manuales](docs/manual-install-phpipam.md).

## Verificar el funcionamiento

* dar de alta en DNS el nodo, por ejemplo:

```bash
echo '192.168.33.10   nsedit.infra.ballardini.com.ar' | sudo tee --append /etc/hosts > /dev/null
echo '192.168.33.20     ipam.infra.ballardini.com.ar' | sudo tee --append /etc/hosts > /dev/null
```


* [Verificaciones mínimas para PowerDNS](docs/verificaciones-powerdns.md)

* http://https://nsedit.infra.ballardini.com.ar/   credenciales: `admin / admin`
* http://https://ipam.infra.ballardini.com.ar/   credenciales: `admin / ipamadmin`

FIXME: TODO


# Referencias

* https://jpmens.net/2010/10/29/alternative-dns-servers-the-book-as-pdf/ Alternative DNS Servers by Jan-Piet Mens
* https://github.com/PowerDNS/pdns-ansible
* https://github.com/tuxis-ie/nsedit
* https://github.com/geerlingguy/ansible-role-nginx
* https://phpipam.net/ phpIPAM – Open source IP address management https://github.com/phpipam https://hub.docker.com/u/phpipam
* https://github.com/mrlesmithjr/ansible-phpipam https://galaxy.ansible.com/CoffeeITWorks/ansible-phpipam
* https://phpipam-ansible-modules.readthedocs.io/en/develop/README.html gestiona datos en phpIPAM
* https://readthedocs.org/projects/phpipam-ansible-modules/ https://github.com/codeaffen/phpipam-ansible-modules This collection provides modules to manage entities in a phpIPAM. This is neighter a collection of roles nor playbooks. It provides modules to wrote your own roles and/or playbooks.

* [Instrucciones manuales](docs/manual-install-nsedit.md)  para instalar nsedit en `blindmaster0`.


# Otras referencias de alternativas no empleadas


* http://spritelink.github.io/NIPAP/ nipap is a sleek, intuitive and powerful IP address management system built to handle large amounts of IP addresses.
* https://github.com/netbox-community/netbox NetBox is an IP address management (IPAM) and data center infrastructure management (DCIM) tool. Initially conceived by the network engineering team at DigitalOcean, NetBox was developed specifically to address the needs of network and infrastructure engineers. It is intended to function as a domain-specific source of truth for network operations.

* https://github.com/nullconfig/phpipam-api Playbooks for interacting with the phpipam api
* https://www.dideo.ir/v/yt/Z6k7jeOQTWs/how-to-integration-powerdns-to-phpipam?list=PLW4lSQGHMQTSowZ4tTSrsWxCfpwBkEpz_





