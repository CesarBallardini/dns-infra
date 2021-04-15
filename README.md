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

| nombre       | red   | dirección IP   | descripción                                         |
|--------------|-------|----------------|-----------------------------------------------------|
| blindmaster0 | nat   | 10.0.2.15      | NAT VB network: internet access                     |
|              | mgmt  | 192.168.33.10  | management network: provision, ssh                  |
|              | infra | 192.168.44.10  | infrastructure network: service network for clients |
|                                                                                             |
| testinterna0 | mgmt  | 192.168.33.100 | VM con acceso a la ren infra para testing           |
|              | mgmt  | 192.168.44.100 |                                                     |



# Que está funcionando en este momento:

* `blindmaster0` tiene instalados:
  * PowerDNS 
  * nsedit (instalado manualmente)
* `testinterna0` puede consultar registros de DNS en `blindmaster0`

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

Seguir las [instrucciones manuales](docs/manual-install-nsedit.md)  para instalar nsedit en `blindmaster0`.

## Verificar el funcionamiento

* [Verificaciones mínimas para PowerDNS](verificaciones-powerdns.md)

FIXME: TODO


# Referencias

* https://jpmens.net/2010/10/29/alternative-dns-servers-the-book-as-pdf/ Alternative DNS Servers by Jan-Piet Mens
* https://github.com/PowerDNS/pdns-ansible
* https://github.com/tuxis-ie/nsedit

