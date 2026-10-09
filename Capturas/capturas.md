1. Instala o servidor BIND9 no equipo darthvader. Comproba que xa funciona coma servidor DNS caché pegando no documento de entrega a saída deste comando dig @localhost xunta.gal 
![alt text](apartado1.png)
2. Configura o servidor BIND9 no equipo mandalorian para que empregue como reenviador a darthvader pegando no documento de entrega contido do ficheiro /etc/bind/named.conf.options e a saída deste comando: dig @localhost santiagodecompostela.gal.
Contido do ficheiro named.conf.options:


options {
	directory "/var/cache/bind";

	// If there is a firewall between you and nameservers you want
	// to talk to, you may need to fix the firewall to allow multiple
	// ports to talk.  See http://www.kb.cert.org/vuls/id/800113

	// If your ISP provided one or more IP addresses for stable 
	// nameservers, you probably want to use them as forwarders.  
	// Uncomment the following block, and insert the addresses replacing 
	// the all-0s placeholder.

	forwarders {![alt text](image.png)
	    192.168.20.10;
	};

	//========================================================================
	// If BIND logs error messages about the root key being expired,
	// you will need to update your keys.  See https://www.isc.org/bind-keys
	//========================================================================
	dnssec-validation auto;

	auth-nxdomain no;    # conform to RFC1035
	listen-on-v6 { any; };
};

![alt text](apartado2.png)![alt text](image.png)
3.Instala unha zona primaria de resolución directa chamada "starwars.lan" e engade os seguintes rexistros de recursos (a maiores dos rexistros NS e SOA imprescindibles):

    Tipo A: darthvader con IP 192.168.20.10
    Tipo A: skywalker con IP 192.168.20.101
    Tipo A: skywalker con IP 192.168.20.111
    Tipo A: luke con IP 192.168.20.22
    Tipo A: darthsidious con IP 192.168.20.11
    Tipo A: yoda con IP 192.168.20.24 e 192.168.20.25
    Tipo A: c3p0 con IP 192.168.20.26
    Tipo CNAME palpatine a darthsidious
    TIPO MX con prioridade 10 sobre o equipo c3po
    TIPO TXT "lenda" con "Que a forza te acompanhe"
    TIPO NS con darthsidious
    Pega no documento de entrega o contido do arquivo de zona, e do arquivo /etc/bind/named.conf.local
	
	zone "starwars.lan"{
    type primary;
    file "/etc/bind/db.starwars.lan";
};
$TTL    86400
@       IN      SOA     darthvader.starwars.lan. admin.starwars.lan. (
                        2025100701  ; Número de serie
                        3600        ; Actualización (Refresh)
                        1800        ; Reintento (Retry)
                        1209600     ; Caducidade (Expire)
                        86400 )     ; TTL mínimo

; Servidores de nomes
@       IN      NS      darthvader.starwars.lan.
@       IN      NS      darthsidious.starwars.lan.

; Rexistros A dos servidores de nomes
darthvader     IN      A       192.168.20.10
skywalker      IN      A       192.168.20.12
skywalker      IN      A       192.168.20.111
luke           IN      A       192.168.20.22
darthsidious   IN      A       192.168.20.11
yoda           IN      A       192.168.20.24
yoda           IN      A       192.168.20.25
c3p0           IN      A       192.168.20.26 
palpatine      IN      CNAME   darthsidious.starwars.lan.
@              IN   MX  10   c3p0.starwars.lan.
lenda          IN   TXT     "Que a forza te acompañe"

4.Instala unha zona de resolución inversa que teña que ver co enderezo do equipo darthvader, e engade rexistros PTR para os rexistros tipo A do exercicio anterior. Pega no documento de entrega o contido do arquivo de zona, e do arquivo /etc/bind/named.conf.local

zone "db.20.168.192.in-addr.arpa"{
    type primary;
    file "/etc/bind/db.192";  
};

$TTL    86400
@       IN      SOA     darthvader.starwars.lan. admin.starwars.lan. (
                        2025100701  ; Número de serie
                        3600        ; Actualización (Refresh)
                        1800        ; Reintento (Retry)
                        1209600     ; Caducidade (Expire)
                        86400 )     ; TTL mínimo

; Servidores de nomes
@       IN      NS      darthvader.starwars.lan.
; Rexistros A dos servidores de nomes
10      IN      PTR     darthsidious.starwars.lan.
12      IN      PTR     skywalker.starwars.lan.
111      IN      PTR    skywalker.starwars.lan.
22      IN      PTR     luke.starwars.lan.
24      IN      PTR     yoda.starwars.lan.
25      IN      PTR     yoda.starwars.lan.
26      IN      PTR     c3p0.starwars.lan.

5.Comproba que podes resolver os distintos rexistros de recursos. Pega no documento de entrega a saída dos comandos:

nslookup darthvader.starwars.lan localhost

root@darthvader:/var/cache/bind# nslookup darthvader.starwars.lan localhost
Server:		localhost
Address:	127.0.0.1#53

Name:	darthvader.starwars.lan
Address: 192.168.20.10

nslookup skywalker.starwars.lan localhost

root@darthvader:/var/cache/bind# nslookup skywalker.starwars.lan localhost
Server:		localhost
Address:	127.0.0.1#53

Name:	skywalker.starwars.lan
Address: 192.168.20.111
Name:	skywalker.starwars.lan
Address: 192.168.20.12

nslookup starwars.lan localhost
        
nslookup -q=mx starwars.lan localhost

root@darthvader:/var/cache/bind# nslookup -q=mx starwars.lan localhost
Server:		localhost
Address:	127.0.0.1#53

starwars.lan	mail exchanger = 10 c3p0.starwars.lan.

nslookup -q=ns starwars.lan localhost

root@darthvader:/var/cache/bind# nslookup -q=ns starwars.lan localhost
Server:		localhost
Address:	127.0.0.1#53

starwars.lan	nameserver = darthsidious.starwars.lan.
starwars.lan	nameserver = darthvader.starwars.lan.

nslookup -q=soa starwars.lan localhost

root@darthvader:/var/cache/bind# nslookup -q=soa starwars.lan localhost
Server:		localhost
Address:	127.0.0.1#53

starwars.lan
	origin = darthvader.starwars.lan
	mail addr = admin.starwars.lan
	serial = 2025100701
	refresh = 3600
	retry = 1800
	expire = 1209600
	minimum = 86400


nslookup -q=txt lenda.starwars.lan localhost

root@darthvader:/var/cache/bind# nslookup -q=txt lenda.starwars.lan localhost
Server:		localhost
Address:	127.0.0.1#53

lenda.starwars.lan	text = "Que a forza te acompa\195\177e"
   
nslookup 192.168.20.11 localhost

