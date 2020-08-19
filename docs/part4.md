---
title: "More techniques"
permalink: /part4/
nav_order: 5
published: true
---

# More techniques

SQL:n ominaisuuksia
SQL-kielessä esiintyy monia samoja elementtejä kuin ohjelmoinnissa: tyyppejä, lausekkeita ja funktioita. Olemme jo nähneet monia esimerkkejä SQL-komennoista, mutta nyt on hyvä hetki tutustua kieleen hieman syvällisemmin.

Tyypit
SQL:n saatavilla olevat tyypit ja niiden ominaisuudet riippuvat käytetystä tietokantajärjestelmästä. Usein pelkästään kokonaislukuja ja merkkijonoja varten on saatavilla suuri määrä vaihtoehtoja. Käytännössä kuitenkin tyypit INTEGER (kokonaisluku) ja TEXT (merkkijono) riittävät pitkälle.

TEXT vs. VARCHAR
Perinteikäs SQL-tyyppi merkkijonon tallentamiseen on VARCHAR, jossa annetaan merkkijonon maksimipituus. Esimerkiksi VARCHAR(100) tarkoittaa merkkijonoa, jossa saa olla enintään 100 merkkiä.

Tämä on jälleen yksi muistuma vanhan ajan ohjelmoinnista: ennen vanhaan merkkijono tallennettiin usein taulukkona, jossa on kiinteä määrä merkkejä. Käytännössä kuitenkin TEXT on kätevämpi, koska meidän ei tarvitse keksiä, montako merkkiä sarakkeessa olevassa merkkijonossa saa olla enintään.

Kokonaisluvun ja merkkijonon lisäksi kolmas usein hyödyllinen tyyppi on aikatyyppi, johon voi tallentaa päivämäärän ja kellonajan. Tämän tyypin nimi ja käyttötavat vaihtelevat kuitenkin käytetyn tietokantajärjestelmän mukaan, joten yksityiskohdat on tarkastettava tietokannan dokumentaatiosta.

Lausekkeet
Lauseke on SQL-komennon osa, jolla on tietty arvo. Esimerkiksi kyselyssä

SELECT hinta FROM Tuotteet WHERE nimi='retiisi';
on kaksi lauseketta: hinta ja nimi='retiisi'. Tässä lauseke hinta määrittää tulostaulun rivien sisällön, kun taas lauseke nimi='retiisi' on ehtolauseke, joka rajoittaa kyselyä.

Voimme käyttää lausekkeissa laskutoimituksia ja muita operaattoreita samaan tapaan kuin ohjelmoinnissa. Esimerkiksi kysely

SELECT hinta*2 FROM Tuotteet WHERE nimi='retiisi';
antaa retiisin hinnan kaksinkertaisena.

Hyvä tapa testata SQL:n lausekkeiden toimintaa on keskustella tietokannan kanssa tekemällä kyselyitä, jotka eivät hae tietoa mistään taulusta vaan laskevat vain tietyn lausekkeen arvon. Keskustelu voi näyttää vaikkapa seuraavalta:

sqlite> SELECT 2*(1+3);
8
sqlite> SELECT 'tes' || 'ti';
testi
sqlite> SELECT 3 < 5;
1
Ensimmäinen kysely laskee lausekkeen 2*(1+3) arvon. Toinen kysely yhdistää ||-operaattorilla merkkijonot 'tes' ja 'ti' merkkijonoksi 'testi'. Kolmas kysely puolestaan määrittää ehtolausekkeen 3 < 5 arvon. Käytäntönä SQL:ssä on, että kokonaisluku ilmaisee totuusarvon: 1 on tosi ja 0 on epätosi.

Monet SQL:n lausekkeisiin liittyvät asiat ovat tuttuja ohjelmoinnista:

laskutoimitukset: +, -, *, /, %
vertaileminen: =, <>, <, <=, >, >=
ehtojen yhdistys: AND, OR, NOT
Näiden lisäksi SQL:ssä on kuitenkin myös erikoisempia ominaisuuksia, joiden tuntemisesta on välillä hyötyä. Seuraavassa on joitakin niistä:

BETWEEN
Lauseke x BETWEEN a AND b on tosi, jos x on vähintään a ja enintään b. Esimerkiksi kysely

SELECT * FROM Tuotteet WHERE hinta BETWEEN 4 AND 6;
hakee tuotteet, joiden hinta on vähintään 4 ja korkeintaan 6. Voimme toki kirjoittaa samalla tavalla toimivan kyselyn myös näin:

SELECT * FROM Tuotteet WHERE hinta >= 4 AND hinta <= 6;
CASE
Rakenne CASE mahdollistaa ehtolausekkeen tekemisen. Siinä voi olla yksi tai useampi WHEN-osa sekä mahdollinen ELSE-osa. Esimerkiksi kysely

SELECT nimi, CASE WHEN hinta>5 THEN 'kallis' ELSE 'halpa' END FROM Tuotteet;
hakee kunkin tuotteen nimen, sekä tiedon siitä, onko tuote kallis vai halpa. Tässä tuote on kallis, jos sen hinta on yli 5, ja muuten halpa.

IN
Lauseke x IN (...) on tosi, jos x on jokin annetuista arvoista. Esimerkiksi kysely

SELECT * FROM Tuotteet WHERE nimi IN ('lanttu','nauris','selleri');
hakee tuotteet, joiden nimi on lanttu, nauris tai selleri.

LIKE
Lauseke s LIKE p on tosi, jos merkkijono s vastaa kuvausta p. Kuvauksessa voi käyttää erikoismerkkejä _ (mikä tahansa yksittäinen merkki) sekä % (mikä tahansa määrä mitä tahansa merkkejä). Esimerkiksi kysely

SELECT * FROM Tuotteet WHERE nimi LIKE '%ri%';
hakee tuotteet, joiden nimen osana esiintyy merkkijono "ri" (kuten nauris ja selleri).

NULL
NULL-arvojen käsittelyä varten on erillinen syntaksi. Esimerkiksi kysely

SELECT * FROM Tuotteet WHERE hinta IS NULL;
hakee tuotteet, joille ei ole annettu hintaa, ja vastaavasti kysely

SELECT * FROM Tuotteet WHERE hinta IS NOT NULL;
hakee tuotteet, joille on annettu hinta.

Funktiot
Lausekkeiden osana voi esiintyä myös funktioita samaan tapaan kuin ohjelmoinnissa. Kuten tyyppien yhteydessä, saatavilla olevat funktiot ja niiden käyttötavat riippuvat käytetystä tietokantajärjestelmästä ja lisätiedot on tarkastettava tietokannan dokumentaatiosta.

Tässä on esimerkkinä joitakin hyödyllisiä SQLite-tietokannan funktioita:

funktio	toiminta
ABS(x)	antaa luvun x itseisarvon
COALESCE(...)	antaa listan ensimmäisen arvon, joka ei ole NULL (tai NULL, jos arvoa ei ole)
LENGTH(s)	antaa merkkijonon s pituuden
LOWER(s)	muuttaa merkkijonon s kirjaimet pieniksi
MAX(x,y)	antaa suuremman luvuista x ja y
MIN(x,y)	antaa pienemmän luvuista x ja y
RANDOM()	antaa satunnaisen luvun
ROUND(x,d)	antaa luvun x pyöristettynä d desimaalin tarkkuudelle
UPPER(s)	muuttaa merkkijonon s kirjaimet suuriksi
Esimerkiksi kysely

SELECT * FROM Tuotteet WHERE LENGTH(nimi)=6;
hakee tuotteet, joiden nimessä on kuusi kirjainta (kuten lanttu ja nauris). Kysely

SELECT * FROM Tuotteet ORDER BY RANDOM();
puolestaan antaa rivit satunnaisessa järjestyksessä, koska järjestys ei perustu minkään sarakkeen sisältöön vaan satunnaiseen arvoon.






Alikyselyt
Alikysely on SQL-komennon osana oleva lauseke, jonka arvo syntyy jonkin kyselyn perusteella. Voimme rakentaa alikyselyjä samaan tapaan kuin varsinaisia kyselyjä ja toteuttaa niiden avulla hakuja, joita olisi vaikea saada aikaan muuten.

Esimerkki
Tarkastellaan esimerkkinä tilannetta, jossa tietokannassa on pelaajien tuloksia taulussa Tulokset. Oletamme, että taulun sisältö on seuraava:

id          nimi        tulos     
----------  ----------  ----------
1           Uolevi      120       
2           Maija       80        
3           Liisa       120       
4           Aapeli      45        
5           Kaaleppi    115    
Haluamme nyt selvittää ne pelaajat, jotka ovat saavuttaneet korkeimman tuloksen, eli kyselyn tulisi palauttaa Uolevi ja Liisa. Saamme tämän aikaan alikyselyllä seuraavasti:

SELECT nimi, tulos FROM Tulokset WHERE tulos = (SELECT MAX(tulos) FROM Tulokset);
Kyselyn tuloksena on:

nimi        tulos     
----------  ----------
Uolevi      120       
Liisa       120       
Tässä tapauksessa alikysely on SELECT MAX(tulos) FROM Tulokset, joka antaa suurimman taulussa olevan tuloksen eli tässä tapauksessa arvon 120. Huomaa, että alikysely tulee kirjoittaa sulkujen sisään, jotta se ei sekoitu pääkyselyyn.

Tässä on vielä vähän mutkikkaampi kysely:

SELECT nimi, tulos FROM Tulokset WHERE tulos >= 0.9*(SELECT MAX(tulos) FROM Tulokset);
Tämä kysely näyttää, että voimme käyttää alikyselyn antamaa arvoa lausekkeen osana samaan tapaan kuin mitä tahansa muuta arvoa. Kysely hakee pelaajat, joiden tulos on enintään 10 prosenttia huonompi kuin paras tulos:

nimi        tulos     
----------  ----------
Uolevi      120       
Liisa       120       
Kaaleppi    115     
Riippuva alikysely
Alikysely on mahdollista toteuttaa myös niin, että sen sisältö riippuu pääkyselyssä käsiteltävästä rivistä. Näin on seuraavassa kyselyssä:

SELECT nimi, tulos, (SELECT COUNT(*) FROM Tulokset WHERE tulos > T.tulos) paremmat FROM Tulokset T;
Tämän kyselyn ideana on laskea jokaiselle pelaajalle, monenko pelaajan tulos on parempi kuin pelaajan oma tulos. Esimerkiksi Maijalle vastaus on 3, koska Uolevin, Liisan ja Kaalepin tulos on parempi. Kysely antaa seuraavan tuloksen:

nimi        tulos       paremmat
----------  ----------  ----------
Uolevi      120         0                                                    
Maija       80          3                                                    
Liisa       120         0                                                    
Aapeli      45          4                                                    
Kaaleppi    115         2                                                   
Koska taulu Tulokset esiintyy kahdessa roolissa alikyselyssä, pääkyselyn taululle on annettu vaihtoehtoinen nimi T. Tämän ansiosta alikyselyssä on selvää, että halutaan laskea rivejä, joiden tulos on parempi kuin pääkyselyssä käsiteltävän rivin tulos.

Monta arvoa alikyselystä
Alikysely voi palauttaa myös useita arvoja, kunhan alikyselyn tulosta käytetään kohdassa, jossa tämä on sallittua. Näin on seuraavassa kyselyssä:

SELECT nimi FROM Tuotteet
WHERE id IN (SELECT tuote_id FROM Ostokset WHERE asiakas_id = 1);
Tämä kysely hakee kaikkien tuotteiden nimet asiakkaan 1 ostoskorissa. Alikysely palauttaa tuotteiden id-numerot, minkä pystyy yhdistämään IN-syntaksiin.

Huomaa, että olisimme voineet rakentaa vastaavan kyselyn myös näin:

SELECT T.nimi
FROM Tuotteet T, Ostokset O
WHERE T.id = O.tuote_id AND O.asiakas_id = 1;
Usein alikysely onkin vaihtoehtoinen tapa toteuttaa jokin kysely, jonka voisi tehdä yhtä hyvin myös esimerkiksi sopivasti laaditulla monen taulun kyselyllä.





Tulosrivien rajaus
SQL-kysely palauttaa oletuksena kaikki ehtoihin täsmäävät rivit, mutta voimme tarvittaessa pyytää vain osan tulostaulun riveistä. Tämä on hyödyllistä esimerkiksi sovelluksissa, joissa halutaan näyttää yhdellä sivulla vain osa haun tuloksista.

Rajaustavat
Kun lisäämme kyselyn loppuun LIMIT x, kysely antaa vain x ensimmäistä tulosriviä. Esimerkiksi LIMIT 3 tarkoittaa, että kysely antaa kolme ensimmäistä tulosriviä.

Yleisempi muoto on LIMIT x OFFSET y, mikä tarkoittaa, että haluamme x riviä kohdasta y alkaen (0-indeksoituna). Esimerkiksi LIMIT 3 OFFSET 1 tarkoittaa, että kysely antaa toisen, kolmannen ja neljännen tulosrivin.

Esimerkki
Tarkastellaan esimerkkinä kyselyä, joka hakee tuotteita halvimmasta kalleimpaan:

SELECT * FROM Tuotteet ORDER BY hinta;
Kyselyn tuloksena on seuraava tulostaulu:

id          nimi        hinta     
----------  ----------  ----------
3           nauris      2
5           selleri     4         
2           porkkana    5         
1           retiisi     7         
4           lanttu      8         
Saamme haettua kolme halvinta tuotetta seuraavasti:

SELECT * FROM Tuotteet ORDER BY hinta LIMIT 3;
Kyselyn tulos on seuraava:

id          nimi        hinta     
----------  ----------  ----------
3           nauris      2         
5           selleri     4         
2           porkkana    5      
Seuraava kysely puolestaan hakee kolme halvinta tuotetta toiseksi halvimmasta tuotteesta alkaen:

SELECT * FROM Tuotteet ORDER BY hinta LIMIT 3 OFFSET 1;
Tämän kyselyn tulos on seuraava:

id          nimi        hinta     
----------  ----------  ----------
5           selleri     4         
2           porkkana    5         
1           retiisi     7      
Rajaus alikyselyssä
Tarkastellaan vielä tilannetta, jossa haluamme laskea kolmen halvimman tuotteen yhteishinnan. Seuraava kysely ei toimi halutulla tavalla:

SELECT SUM(hinta) FROM Tuotteet ORDER BY hinta LIMIT 3;
Kysely antaa kaikkien taulun tuotteiden hintojen summan:

SUM(hinta)
----------
26
Ongelmana on, että kysely muodostaa tulostaulun, jossa on vain yhdellä rivillä luku 26 (kaikkien tuotteiden summa), minkä jälkeen tästä tulostaulusta haetaan kolme ensimmäistä riviä (eli käytännössä vain yksi rivi).

Ratkaisu ongelmaan on luoda alikysely, jossa haetaan kolme halvinta hintaa, ja laskea sitten summa näistä hinnoista:

SELECT SUM(hinta) FROM (SELECT hinta FROM Tuotteet ORDER BY hinta LIMIT 3);
Nyt kysely antaa halutun tuloksen:

SUM(hinta)
----------
11
