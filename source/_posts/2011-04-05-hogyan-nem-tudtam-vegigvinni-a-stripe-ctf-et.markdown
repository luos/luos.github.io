---
layout: post
title: "Hogyan nem tudtam végigvinni a Stripe CTF-et?"
date: 2011-04-05 12:38:00 +0100
comments: true
categories: ctf
---

A Stripe CTF egy webes biztonságtechnikai verseny, ahogy már írtam róla<a title="Stripe CTF – első benyomások" href="http://luoska.wordpress.com/2012/08/23/stripe-ctf-elso-benyomasok/"> egyik előző</a> postomban. Nekem nagyon tetszett, még sosem játszottam ilyenen.

Eljutottam az utolsó előtti pályáig, de az teljesen kifogott rajtam.

Nézegettem a kódot, de sehogy sem sikerült rájönnöm, hogy mi a hiba. Persze arra rájöttem, hogy részben ezt a kódot kell kihasználni:

``` python

def parse_params(raw_params):
   pairs = raw_params.split('&')
   params = {}
   for pair in pairs:
     key, val = pair.split('=')
     key = urllib.unquote_plus(key)
     val = urllib.unquote_plus(val)
     params[key] = val
 return params

```

Elég nyilvánvaló volt, hogy az POST request bodyját úgy kell majd módosítani, hogy a hátul lévő paraméterek felülírják az elől lévőket, így tulajdonképpen valahogy túl kéne járni a hash ellenőrző eszén. Ezt jól is gondoltam, de ezen kívül itt el is akadtam. Fel kellett mennem az IRC-re, hogy megkérdezzem, mégis mi a halált kéne csinálnom (note: itt már kb 2 napja nézegettem a kódot). Annyit mondtak kb, hogy "the hashing has some security vulnerabilities. Nah hát ez sem sokat segített, úgyhogy rákerestem a googleban, hogy mi ennek a feladatnak a megoldása.
Hát nem csodálom, hogy nem tudtam megoldani, de legalább egy olyan dolgot tanultam, amit egy átlagember nem nagyon tud, legalábbis szerintem. Érdekes, hogy egy félév kódolástechnika után semmit sem tudtam arról, hogy a valóságban hogyan működnek a hash függvények.

A hiba ebben a kódrészletben volt:

``` python

secret = str(row['secret'])

h = hashlib.sha1()
 h.update(secret + raw_params)
 print 'computed signature', h.hexdigest(), 'for body', repr(raw_params)
 if h.hexdigest() != sig:
raise BadSignature('signature does not match')
 return True

```

Ezt a fajta ellenőrzést, hogy biztosítsuk azt, hogy attól származik az üzenet, akinek ő mondja magát <a href="http://en.wikipedia.org/wiki/Hash-based_message_authentication_code" target="_blank">Hash-based message authenticationnak (HMAC)</a> hívják. De "sajnos" ebben az appban ez hibásan volt implementálva Ha jól mgnézzük, akkor ezt a stringet hasheli "secret + raw_params" ahol a raw params a postolt változók és értékek &amp; és = összefűzve, a secret meg minden felhasználónak egyedi. Azért hibás, mert a hash függvények az adat blokkjain mennek végig, az itt használt sha1 konkrétan 512 bitenként "forog" egyet. Minden forgás után vannak regiszterek, ahova az eredmény beíródik. Ha ez az utolsó forgás volt, akkor az lesz a hash függvény kimenete. Az utolsó blokkba pedig belekerül az üzenet mérete. az utolsó 8 bájtra, a maradék pedig 0 bitekkel lesz feltöltve.

Ez konkrétan azt jelenti, hogy ha mondjuk van ez a query stringünk:
count=10&amp;lat=37.351&amp;user_id=1&amp;long=-119.827&amp;waffle=eggo

Ha jól számolom ez 52 karakter, ami 416 bit. Ehhez tehát még hozzájön a titkos kulcs (legyen mondjuk most 10 karakter), ahhoz pedig hozzájön még a méret az utolsó 8 bájtra, a maradék 0-kal lesz feltöltve:
TITKOS_KULCS + count=10&amp;lat=37.351&amp;user_id=1&amp;long=-119.827&amp;waffle=eggo (ez eddig 62 karakter), ez már 496 bit, ide már nem fér be a 8 bájtos méret, ezért elkezdjük feltölteni a maradékot. Az sha1 algoritmus 0x80-al kezdi a feltöltést (egy egyes bittel), utána 0-k.

Tehát most van két 512 bites blokkunk, az egyik elején a query string + titkos kulcs  + 0x0 és 0-k, a másikban sok 0 és a végén a méret.
Az előbb írtam, hogy blokkonként történik a feldolgozás, ami annyit jelent, hogy először feldolgozódik az első 512 bit, ez elmentődik a regiszterekbe (ha 1 blokk lenne, akkor itt lenne az eredmény), majd a második blokk feldolgozása kezdődik, aminek a végén megkapjuk a kimenetet.
A lényeg az volt, hogy nekünk van egy ilyen kimenetünk, mert meg tudjuk nézni, hogy mi volt a felhasználók lekérése. Természetesen a titkos kulcsukat nem ismerjük, így nem tudtunk a helyükben üzeneteket küldözgetni. Vagyishogy tudunk:

Ha jól megnézzük, akkor abból kiindulva, hogy blokkonként történik a feldolgozás, ha hozzá tudunk fűzni a lekéréshez még paramétereket (amit tudunk), akkor beállítható a szerver hash funkcióinak regiszterei a kívánt állapotba (tehát, hogy az ismert kimeneti hash legyen a regiszterben), innentől mivel a következő blokk csak az előző blokkon múlik (amit tudunk, hisz vissza tudtuk nézni mások hasheit) és az adott blokkon, az adott blokkot mi adhatjuk meg, és nem kell tudnunk a titkos kulcsot, hogy túljuthassunk a hash ellenőrzésen.

Ehhez le kellett szednem egy sha1 c implementációt, hogy a regisztereket a kívánt kezdeti értékre állíthassam (a hashre, amit ismerek)

<a href="http://luoska.files.wordpress.com/2012/09/c_sha1.png"><img class="aligncenter size-full wp-image-294" title="c_sha1" src="http://luoska.files.wordpress.com/2012/09/c_sha1.png" alt="" width="640" height="307" /></a>

Így kellett volna csinálni elvileg, de nekem mégsem sikerült összehozni, hogy valid hasht generáljak.
Szóval nem sikerült ezt a pályát teljesítenem. :(
Lehet, hogy valamit félreértettem, meg amúgy nem gondolkoztam rajta eleget. Valószínűleg ehhez még kevés voltam most, de majd a következőn. :)

További információ:

<a href="http://www.herongyang.com/Cryptography/SHA1-Message-Digest-Algorithm-Overview.html">http://www.herongyang.com/Cryptography/SHA1-Message-Digest-Algorithm-Overview.html</a>