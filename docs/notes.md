# saját CA  

## Tanácstalanság
Vajon milyen tutorial információit lenne érdemes használni?
Van sok, például:
- https://pki-tutorial.readthedocs.io/en/latest/simple/index.html
- https://roll.urown.net/ca/index.html
- https://jamielinux.com/docs/openssl-certificate-authority/index.html  
Persze ennél jóval több van, de én csak ezeket nézegetem egyelőre.

Olvasgatva e témában merülnek fel olyan kérdések, hogy
- [ ] Merjek-e hinni olyan anyagnak, amelyikben a default_md helyén md5, sha1 szerepel?
- [ ] Mi a helyzet azokkal az oldalakkal, ahol a serial-t egy 'echo 00 >serial' paranccsal intézi el a szerző,
    ahelyett, hogy a sok helyen javasolt 'openssl rand -hex 16 >serial'-t használná legalább?



## root CA
- Könyvtár struktúra kialakítása (openssl.cnf alapján - certs,newcerts(??),private,requests, index.txt, serial (random értékkel feltöltve)
- Privát kulcs készítés(4096 bit, jelszóval védett!)
- Self-signed cert előállítása

- [ ] Fontos lenne, de egyelőre nem tudom, hová kell tenni: **basicConstraints=critical,CA:TRUE,pathlen:0**  
Vagy a rootCA ooenssl.cnf-be, vagy az intermediate-be. Érzésem szerint a rootéba, ott is a ca sectionbe, ezzel meggátolva, hogy az intermediate CA újabb CA tanúsítványt írjon alá.  
- [x] Vajon mi van, ha a critical itt elmarad?  
    A critical jelentősége annyi, hogy ha az így megjelölt extension-t a tanúsítványt használó applikáció nem ismeri fel, akkor nem fogadja el a tanúsítványt. Szabványos extension-ök esetében nincs jelentősége ([forrás](https://security.stackexchange.com/questions/30974/which-properties-of-a-x-509-certificate-should-be-critical-and-which-not))  
  


## intermediate CA
- Könyvtár struktúra létrehozása (mint a root CA)
- Privát kulcs készítése
- CSR létrehozása
- CSR aláírása a root CA-val, aláírt certificate visszamásolása az intermediate könyvtárába.

## server & user certificate

## Források
https://en.m.wikipedia.org/wiki/Public_key_certificate   
https://www.openssl.org/docs/man1.1.1/man5/x509v3_config.html   
