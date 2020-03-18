# saját PKI  



<br><br>
Ezzel az egész irománnyal ne foglalkozz, csak jegyzetek a téma kapcsán, amit nem akarok elfelejteni.



## Tanácstalanság


Vajon milyen tutorial információit lenne érdemes használni?
Van sok, például:
- https://pki-tutorial.readthedocs.io/en/latest/simple/index.html
- https://roll.urown.net/ca/index.html
- https://jamielinux.com/docs/openssl-certificate-authority/index.html
- https://www.feistyduck.com/library/openssl-cookbook/

Persze ennél jóval több van, de én csak ezeket nézegetem egyelőre.<br>
Külön ciki, hogy sok szakkifejezést nem tudok normálisan magyarra fordítani, szóval...


Olvasgatva e témában merülnek fel olyan kérdések, hogy
- [ ] Merjek-e hinni olyan anyagnak, amelyikben a default_md helyén md5, sha1 szerepel?
- [ ] Mi a helyzet azokkal az oldalakkal, ahol a serial-t egy 'echo 00 >serial' paranccsal intézi el a szerző,
    ahelyett, hogy a sok helyen javasolt 'openssl rand -hex 16 >serial'-t használná legalább?
- [ ] Az `openssl req` parancs elvileg képes self signed certificate előállítására, de... kérdés, hogy root CA esetén jó ötlet-e a használata? Ugyanis nem update-eli az index fájlt, nem nyúl a serial-hoz sem, csak generál egy privát kulcsot, meg egy self signed tanúsítványt, amelynek néhány paraméterét az openssl.cnf-ből veszi. Emiatt például nem lehet visszavonni a root certificate-et, ha kompromittálódna, mert a crl az index fájlból készül. Ugyanakkor... mivel a self signed tanúsítványt használnám a CRL aláírására is... ha a root CA kompromittálódott, akkor a CRL sem lesz hiteles. Akkor ezzel mit is lehet tenni?
- [x] Hogyan lehetne az `openssl verify`-t működésre bírni, tesztkörnyezetben? Nem találja az aláírókat, akkor sem, ha paraméterként megkapja mindkét CA tanúsítványát.
Ezt én rontottam el. Egyelőre nem teljesen tiszta a dolog, de a két CA tanúsítványt be kell "csomagolni" egy fájlba (PEM formátumú mindkettő, ezekből lesz egy PKCS#???)
- [ ] A `man ca` azt írja, hogy az `openssl ca` parancs csak egy sample app... Akkor ne is használjam? 🤔 
Egy opció: [DogTag](https://dogtagpki.org) 
  <br>
- [x] Distinguished Name (DN) használatakor sok helyen felbukkan az E avagy emailAddress RDN (=Relative Distinguished Name), de ez amennyire értem, deprecate státuszú. Helyette 
a SAN (=Subject Alternate Name) használata a javasolt. ([Forrás1](https://docs.oracle.com/cd/E19957-01/816-5531-10/app_dn.htm), [Forrás2](https://www.cryptosys.net/pki/manpki/pki_distnames.html))
- [x] Apropó DN... sokáig kerestem, hogy mit lehet/kell itt megadni. A fenti két forrás alapján
úgy tűnik, vannak értékek, amiket meg kell/illik adni (CN, C, L, ST, O, OU stb.) ugyanakkor
bármely szabványos(???) előre definiált OID ([oid-info.com](oid-info.com)) használható, legalábbis
az openssl toolok elfogadják. (böngészők, egyéb alkalmazások nem tudom, mit szólnak hozzá)
 
  <br>

## <u>root CA</u>
###
#### Létrehozása

- Könyvtár struktúra kialakítása (openssl.cnf [ ca ] section alapján - pl.: certs,newcerts(??),private,requests, index.txt, serial (random értékkel feltöltve)
crlnumber (vagy 0000 vagy random értékkel feltöltve)<br>
<s>Itt a certs jelenléte nem igazán tiszta, sok helyen a newcerts és a certs ugyanarra 
a könyvtárra hivatkozik.</s> A newcerts az, ahová automatikusan bekerülnek az elkészített tanúsítványok, a név a sorozatszám (serial) +".pem" lesz.<br>
A certs-be manuálisan kell az eddigi tapasztalataim szerint letenni az elkészült
tanúsítványokat, tetszőleges névvel, amit könnyebb megjegyezni, mint a serial+".pem"-t.
De ez nem biztos...<br>


- Privát kulcs készítés(min. 4096 bit, jelszóval védett!), self signed cert előállítása<br>
Ennek lépései:
  - Kulcs készítés (`openssl genrsa ...`)<br>
  Ehhez nem kell a config file
  - CSR létrehozása (`openssl req -new ...`) - ehhez szükség lesz egy megfelelően
  kitöltött konfig fájlra. Ehhez mintául szolgálhat a linuxra felrakott default openssl.cnf
  vagy valamelyik tutorial állománya is, ha valaki így akarja felépíteni. (**TODO: összeszedni
  a szükséges paramétereket**)
  - A CSR fájlt aláírni a privát kulccsal. (`openssl ca -selfsign...`) - ehhez szintén kell konfig,
  (**TODO: összeszedni a CA használathoz szükséges paramétereket, köztük kétféle x509v3 extension-nel, ha igaz, mert más kell a selfsignedhez és más az intermediate certificate aláírásához. Előbbinek nem kell a pathlen:0, utóbbihoz feltétlenül szükséges**)<br>
  - De... mindezt sokkal egyszerűbb előállítani egyetlen paranccsal: <br>
  `openssl req -newkey rsa:4096 -keyout ... -x509 -utf8 -days 7330 -out ...`
  Ez egymagában előállítja a privát kulcsot, jelszót kér hozzá, majd ebből generál 
  egy self signed certificate-et. (ami kérdéses: milyen konfig fájl kell neki, kell-e a ca és a req secrion, valamint a különböző extension-ök? - szinte biztosan kell minden)
  Azért ezzel gond is van: a jelek szerint nem vesz figyelembe egy csomó paramétert
  a konfig fájlból, ezeket parancssorból kell átadni: például a létrehozott selfsigned
  tanúsítvány érvényességi idejét biztosan...
  - Nem elfelejteni a crl distribution point paramétert!!
  - CRL-nek is van lejárata, ami nem lehet hosszabb 30 nap/egy hónapnál ([követelmény/ajánlás/szabvány](https://www.cabforum.org/wp-content/uploads/Baseline_Requirements_V1.pdf))



#### Kérdések
- [x] Fontos lenne, de egyelőre nem tudom, hová kell tenni:<br> **basicConstraints=critical,CA:TRUE,pathlen:0**   <br>
<s>Vagy a rootCA openssl.cnf-be, vagy az intermediate-be. Érzésem szerint a rootéba, ott is a ca sectionbe, ezzel meggátolva, hogy az intermediate CA újabb CA tanúsítványt írjon alá.</s> Természetesen a láncban utolsó CA (aki már nem írhat alá újabb CA tanúsítványt) előtti konfigjába
kell, esetemben a rootCA-éba. Hogy ott pontosan hová, az még mindig nem tiszta,
mert csak az általa aláírt tanúsítványba kell bekerülnie, a selfsigned-be nem.  
- [ ] Miért fogadja el az `openssl verify` akkor is a tanúsítványt, ha egy pathlen:0-t tartalmazó
CA tanúsítványával van hitelesítve CA-ként? (már feltéve, hogy a basicConstraints=CA:true ezt
jelenti..
- [x] <s>Vajon mi van, ha a critical itt elmarad?</s>  
    A critical jelentősége annyi, hogy ha az így megjelölt extension-t a tanúsítványt használó applikáció nem ismeri fel, akkor nem fogadja el a tanúsítványt. Szabványos extension-ök esetében nincs jelentősége ([forrás](https://security.stackexchange.com/questions/30974/which-properties-of-a-x-509-certificate-should-be-critical-and-which-not))  
- [ ] Ékezetes karakterekkel lehetnek gondok, ha nem figyelsz: a konfig fájlokban
fogalmam sincs, hogyan lehetne beállítani, hogy következetesen UTF-8 kódolást
használjon. Pár parancsnak van -utf8 kapcsolója és _**elvileg**_ a konfigba írt name_opt = utf8 arra szolgál, hogy ha meg akarod jeleníteni egy UTF8 karaktereket tartalmazó
 tanúsítvány adatait, akkor ezzel fixen be lehet állítani, de nekem egyelőre nem működik
 
- [ ] Vajon mi jelentősége van a titkosításhoz használt algoritmus kiválasztásának? 
Mikor/miért érdemes RSA helyett mást használni?
- [ ] mi mindent érdemes beállítani az extension-ökben? Root CA, intermediate CA esetében,
illetve a végfelhasználói tanúsítvány (szerveré, useré stb.) esetében?
- [ ] CRL disztribúciós címek? 
- [ ] Van a .cnf fájlban egy certs nevű változó/paraméter. Ennek nem látom nyomát
a manualokban (pl. man ca), ellenben a default települő (ubuntu-n) /etc/ssl/openssl.cnf
fájlban szerepel a new_certs_dir mellett, a komment szerint hasonló funkcionalitással.
Tényleg kell?
## intermediate CA
- Könyvtár struktúra létrehozása (mint a root CA)
- Privát kulcs készítése
- CSR létrehozása
- CSR aláírása a root CA-val, aláírt certificate visszamásolása az intermediate könyvtárába.

## server & user certificate

## Források
https://en.m.wikipedia.org/wiki/Public_key_certificate   
https://en.wikipedia.org/wiki/X.509  
https://www.openssl.org/docs/man1.1.1/man5/x509v3_config.html   
  
https://tools.ietf.org/html/rfc4519  

