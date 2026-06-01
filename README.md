# PRÀCTICA: Analitzador de protocols Wireshark

> ⚠️ **ATENCIÓ:** Els noms dels equips hosts al VirtualBox, el nom del sistema quan instal·leu, els noms de l'usuari, equip, etc., han de ser els vostres perquè es puguin veure a les captures.

Teniu l'exercici a fer al final d'aquest enunciat. Llegiu primer tot el document abans de començar la pràctica.

En general, heu de comprovar **SEMPRE** que els canvis de configuració que feu produeixen l'efecte desitjat amb el conjunt de proves necessari.

---

## 🔗 Referències i ajuda

Wireshark és una eina molt potent i amb moltes opcions, aquí teniu un recopilatori de recursos:

* **Configurar Wireshark:** [wireshark.org/docs/wsug_html/](https://www.wireshark.org/docs/wsug_html/)
* **Manuals de Wireshark:**
  * [wireshark.org/#learnWS](https://www.wireshark.org/#learnWS)
  * [wireshark.org/docs/wsug_html/](https://www.wireshark.org/docs/wsug_html/)
  * [wiki.wireshark.org/FrontPage](https://wiki.wireshark.org/FrontPage)
* **Exportar fitxers d'un PCAP amb Wireshark:** [unit42.paloaltonetworks.com/.../exporting-objects-from-a-pcap/](https://unit42.paloaltonetworks.com/using-wireshark-exporting-objects-from-a-pcap/)
* **Display filters:** [unit42.paloaltonetworks.com/.../display-filter-expressions/](https://unit42.paloaltonetworks.com/using-wireshark-display-filter-expressions/)
* **Identificar hosts i usuaris:** [unit42.paloaltonetworks.com/.../identifying-hosts-and-users/](https://unit42.paloaltonetworks.com/using-wireshark-identifying-hosts-and-users/)
* **PCAP per anàlisi de tràfic i malware:** [malware-traffic-analysis.net](https://www.malware-traffic-analysis.net/index.html)
* **Anàlisi de tràfic amb Wireshark (INCIBE):** [incibe.es/.../cert_inf_seguridad_analisis_trafico_wireshark.pdf](https://www.incibe.es/extfrontinteco/img/File/intecocert/EstudiosInformes/cert_inf_seguridad_analisis_trafico_wireshark.pdf)
* **Determinar el fabricant a partir de la MAC:** [maclookup.app](https://maclookup.app/macaddress/8CBEBE)

---

## 🎯 Objectius

L'objectiu d'aquesta activitat és treballar amb **Wireshark**, un *sniffer* o analitzador de protocols, per a resoldre alguns reptes d'obtenció d'informació.

---

## 📖 Introducció (Teoria)

L'eina que farem servir és Wireshark, que ja tenim instal·lat al Kali Linux. 

* **ICMP (Internet Control Message Protocol):** Permet als dispositius de xarxa informar d'una errada. Per exemple, si un router no pot encaminar un paquet, el descarta i torna un error ICMP a l'emissor; o si un paquet acaba el seu TTL (*Time To Live*), el router el descarta i envia un missatge ICMP a l'emissor.
  * **Ping:** És un dels serveis que ofereix ICMP. Serveix per comprovar si es pot accedir a un equip a la xarxa i ens diu el temps que es triga a obtenir resposta.
  * **Traceroute:** Ens diu per quins routers passen els paquets fins a arribar al host destí.
* **ARP (Address Resolution Protocol):** Permet, a partir d'una adreça IP, aconseguir l'adreça MAC corresponent. Això es fa preguntant en *Broadcast* a la xarxa qui té una determinada IP. En teoria, només contestarà un host.
* **DNS (Domain Name Service):** Permet traduir un nom de domini com ara `www.google.es` a una adreça IP. Podem veure aquesta traducció fent servir la comanda `nslookup www.google.es`. També es pot obtenir el nom de domini a partir d'una adreça IP, tot i que no sempre funciona.
* **FTP (File Transfer Protocol):** Permet transferir o enviar fitxers entre hosts. Un servidor FTP obre el port `21` (port ftp) per a poder "conversar" amb ell (enviar ordres i rebre respostes).
  * **Mode Actiu:** El client obre un port i indica al servidor que es connecti a aquest port. Això implicaria al firewall haver d'obrir molts ports i normalment no es permet.
  * **Mode Passiu:** El servidor obre un port per a les dades, indica al client quin és i llavors el client obre una connexió nova per baixar-se les dades (el contingut del directori o el fitxer demanat mitjançant un `get`).
* **HTTP/HTTPS (Hypertext Transfer Protocol):** És el protocol que permet visitar pàgines web.

---

## 🧹 Filtres

El *display filter* de Wireshark usa expressions booleanes. Això permet especificar valors i fer filtres complexos. Els operadors booleans són:

* **Igual:** `==` o `eq`
* **No igual:** `!=`
* **And (i lògic):** `&&` o `and`
* **Or (o lògic):** `||` o `or`

### 💡 Exemples de filtres:

* Paquets que contenen les adreces `192.168.10.195` i `192.168.10.1` (una ha de ser origen i l'altra destí, o al revés):
```text
  ip.addr eq 192.168.10.195 and ip.addr == 192.168.10.1
