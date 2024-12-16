# DNS_Linux
Résultat du challenge WCS serveur DNS sous Linux


Prérequis, régler l'adresse IP du serveur DNS sur 192.168.0.40/24 et l'adresse du client Linux sur 192.168.1.41/24
Pour répondre au challenge, il suffit de remplacer dans les lignes de commandes ci-dessous "linuxtechi.local" par "wilders.lan"



## 1) Installer le Package Bind 9


  $ sudo apt update
  $ sudo apt install -y bind9 bind9utils bind9-doc dnsutils

Tous les fichiers de configuration de Bind sont dans le repertoire /etc/bind 

## 2) Ajouter les paramètres suivant au fichier de configuration /etc/bind/named.conf.options


  $ sudo vi /etc/bind/named.conf.options

    acl internal-network {
    192.168.0.0/24;
    };
    options {
        directory "/var/cache/bind";
        allow-query { localhost; internal-network; };
        allow-transfer { localhost; };
        forwarders { 8.8.8.8; };
        recursion yes;
        dnssec-validation auto;
        listen-on-v6 { any; };
    };


## 3) Ajouter les paramètres suivant au fichier de configuration /etc/bind/named.conf.local“


  $ cd /etc/bind
  $ sudo vi named.conf.local
    
    zone "linuxtechi.local" IN {
        type master;
        file "/etc/bind/forward.linuxtechi.local";
        allow-update { none; };
    };
    zone "0.168.192.in-addr.arpa" IN {
        type master;
        file "/etc/bind/reverse.linuxtechi.local";
        allow-update { none; };
    };

## 4) Déclaration de la zone de recherche direct

  $ cd /etc/bind
  $ sudo cp db.local forward.linuxtechi.local
  $ sudo vi forward.linuxtechi.local
  
    $TTL 604800
    @ IN SOA primary.linuxtechi.local. root.primary.linuxtechi.local. (
                                   2022072651 ; Serial
                                   3600 ; Refresh
                                   1800 ; Retry
                                   604800 ; Expire
                                   604600 ) ; Negative Cache TTL
    ;Name Server Information
    @       IN  NS    primary.linuxtechi.local.

    ;IP address of Your Domain Name Server(DNS)
    primary IN  A     192.168.0.40

    ;Mail Server MX (Mail exchanger) Record
    linuxtechi.local. IN MX 10   mail.linuxtechi.local.

    ;A Record for Host names
    www     IN  A    192.168.0.50
    mail    IN  A    192.168.0.60

    ;CNAME Record
    ftp     IN CNAME www.linuxtechi.local.



## 5) Déclaration de la zone de recherche inversée



  $ sudo cp db.127 reverse.linuxtechi.local
  $ sudo vi /etc/bind/reverse.linuxtechi.local
    
    $TTL 86400
    @ IN SOA linuxtechi.local. root.linuxtechi.local. (
                           2022072752 ;Serial
                           3600 ;Refresh
                           1800 ;Retry
                           604800 ;Expire
                           86400 ;Minimum TTL
    )
    ;Your Name Server Info
    @ IN NS primary.linuxtechi.local.
    primary   IN  A    192.168.0.40
    ;Reverse Lookup for Your DNS Server
    40        IN PTR   primary.linuxtechi.local.
    ;PTR Record IP address to HostName
    50        IN PTR   www.linuxtechi.local.
    60        IN PTR   mail.linuxtechi.local.




## 5) Modifier les fichier /etc/default/named  afin que le serveur DNS "écoute" les adresses IP V4

    OPTIONS="-u bind -4"




## 6) Appliquer la configuration 


$ sudo systemctl start named
$ sudo systemctl enable named



## 7) Vérifier le status de Bind

$ sudo systemctl status named

(A la ligne Active :   il devrait y avoir "active (running)




## 8) Autoriser le port #53 dans le Firewall (Port pour l'écoute du serveur DNS)


$ sudo ufw allow 53

## 9) sur une machine cliente sous Linux ajouter l'adresse du serveur DNS

$ sudo vi /etc/resolv.conf

    search linuxtechi.local
    nameserver 192.168.0.40


## 9) Vérifier que tout fonctionne avec la commabnde "dig"

$ dig primary.linuxtechi.local




![DNS_Linux](https://github.com/Hebus79/DNS_Linux/blob/main/images/DIG_VERIF_FIN.png)

