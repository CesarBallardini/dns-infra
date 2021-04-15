# Verificaciones funcionamiento PowerDNS


## Iniciado correctamente

```bash
sudo systemctl status pdns
```

## Accesible

* no atiende en `localhost`

```bash
dig a www.example.com @127.0.0.1
```

```text
; <<>> DiG 9.16.13-Debian <<>> a www.example.com @127.0.0.1
;; global options: +cmd
;; connection timed out; no servers could be reached
```

* atiende en la interfaz de servicio en la red `infra`

```bash
dig a www.example.com @192.168.44.10
```

```text
; <<>> DiG 9.16.13-Debian <<>> a www.example.com @192.168.44.10
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: REFUSED, id: 16595
;; flags: qr rd; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ADDITIONAL: 1
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1680
;; QUESTION SECTION:
;www.example.com.		IN	A

;; Query time: 0 msec
;; SERVER: 192.168.44.10#53(192.168.44.10)
;; WHEN: Thu Apr 15 15:55:24 UTC 2021
;; MSG SIZE  rcvd: 44
```

* tampoco atiende en la interfaz de `magmt`

```bash
dig a www.example.com @192.168.33.10
```

## Creamos una zona y agregamos registros

*  creamos la zona `example.com` con nameserver `ns1.example.com`
```bash
sudo -u pdns pdnsutil create-zone example.com ns1.example.com
```

```text
Creating empty zone 'example.com'
Also adding one NS record
```

* agregamos registro MX

```bash
sudo -u pdns pdnsutil add-record example.com '' MX '25 mail.example.com'
```

```text
New rrset:
example.com. IN MX 3600 25 mail.example.com
```

* agregamos registro A

```bash
sudo -u pdns pdnsutil add-record example.com. www A 192.168.88.1
```

```text
New rrset:
www.example.com. IN A 3600 192.168.88.1
```

## Consultamos los registros recién agregados


```bash
dig +short www.example.com @192.168.44.10
```

```text
192.168.88.1
```

```bash
dig +short example.com MX @192.168.44.10
```

```text
25 mail.example.com.
```

* si no usamos la opción `+short` queda:

```bash
dig www.example.com @192.168.44.10
```

```text
; <<>> DiG 9.16.13-Debian <<>> www.example.com @192.168.44.10
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 37332
;; flags: qr aa rd; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1680
;; QUESTION SECTION:
;www.example.com.		IN	A

;; ANSWER SECTION:
www.example.com.	3600	IN	A	192.168.88.1

;; Query time: 0 msec
;; SERVER: 192.168.44.10#53(192.168.44.10)
;; WHEN: Thu Apr 15 16:04:12 UTC 2021
;; MSG SIZE  rcvd: 60
```



# Referencias

* https://docs.powerdns.com/authoritative/guides/basic-database.html

