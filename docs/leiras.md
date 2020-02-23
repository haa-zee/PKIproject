# Saját PKI

## 1. Mi ez az egész?
_Ez egy leginkább magamnak szóló leírás, hogy hogyan lehet létrehozni saját
CA-t (root, intermediate), elképzelhető, sőt biztos, hogy teli van tárgyi tévedésekkel,
hibásan használt szakkifejezésekkel, hiányos stb. Vélhetőleg folyamatosan változni fog,
ahogy újabb információkra bukkanok_
  
Az iromány célja, hogy összeszedjem az n+1 tutorialban talált infókat a magam számára
emészthető formában. Sok dolog van, amikről az eddig megtalált forrásaim, főként 
a tutorialok, nem igazán írnak, vagy épp ahány oldal annyiféle információt tartalmaznak.

Miért lehet szükség saját (root) CA-ra? Például, ha szeretnél a saját 
hálózatodon https szerver(eke)t üzemeltetni, a wifin tanúsítvány alapú hitelesítést
használni, netán olyan proxy-t, amivel MitM támadást tudsz elkövetni a LAN-on 
lévő gépek ellen, hogy vissza tudd fejteni azok forgalmát stb.    

Ezekhez nem árt ha van lehetőség valahogy összefogni a különböző célokra felhasznált
tanúsítványok készítését, aláírását, hogy ne éktelenkedjen mindenhol self signed
certificate. No és az említett mitm proxy miatt, ismereteim szerint, szükség lehet
hitelesen aláírt tanúsítványokra, amihez nem ismerek más lehetőséget, mint a saját
CA-t. 
## 2. CA-k létrehozása  
  
  
    

## 3. CA-k használata



## n. Felhasznált források
### Tutorialok
- https://pki-tutorial.readthedocs.io/en/latest/simple/index.html
- https://roll.urown.net/ca/index.html
- https://jamielinux.com/docs/openssl-certificate-authority/index.html

### Manualok, wiki oldalak
https://en.m.wikipedia.org/wiki/Public_key_certificate   
https://en.wikipedia.org/wiki/X.509  
https://www.openssl.org/docs/man1.1.1/man5/x509v3_config.html   
  
### RFC-k
https://tools.ietf.org/html/rfc4519 - LDAP - benne a DN alatt felhasználható nevekkel (DC, CN, L, C, O, OU stb.)  

