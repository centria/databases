---
title: "Efficiency"
permalink: /part7/
nav_order: 8
published: true
---

# Efficiency

Kyselyjen suoritus
SQL-kieli on tietokannan käyttäjälle mukava kyselyjen tekemisessä, koska käyttäjän riittää kuvata, mitä tietoa hän haluaa hakea, ja tietokantajärjestelmä hoitaa loput. Niinpä tietokantajärjestelmän on tärkeää pystyä löytämään jokin tehokas tapa toteuttaa käyttäjän antama kysely ja toimittaa kyselyn tulokset käyttäjälle.

Kyselyn suunnitelma
Monet tietokantajärjestelmät antavat mahdollisuuden pyytää järjestelmää kertomaan suunnitelmansa, miten se aikoo suorittaa annetun kyselyn. Tämän avulla voimme tutkia tietokantajärjestelmän sisäistä toimintaa.

Tarkastellaan esimerkkinä kyselyä, joka hakee retiisin tiedot taulusta Tuotteet:

SELECT * FROM Tuotteet WHERE nimi='retiisi';
Kun laitamme SQLitessä kyselyn eteen sanan EXPLAIN, saamme seuraavan tapaisen selostuksen siitä, miten kysely aiotaan suorittaa:

sqlite> EXPLAIN SELECT * FROM Tuotteet WHERE nimi='retiisi';
addr  opcode         p1    p2    p3    p4             p5  comment      
----  -------------  ----  ----  ----  -------------  --  -------------
0     Init           0     12    0                    00  Start at 12  
1     OpenRead       0     2     0     3              00  root=2 iDb=0; Tuotteet
2     Rewind         0     10    0                    00               
3       Column         0     1     1                    00  r[1]=Tuotteet.nimi
4       Ne             2     9     1     (BINARY)       52  if r[2]!=r[1] goto 9
5       Rowid          0     3     0                    00  r[3]=rowid   
6       Copy           1     4     0                    00  r[4]=r[1]    
7       Column         0     2     5                    00  r[5]=Tuotteet.hinta
8       ResultRow      3     3     0                    00  output=r[3..5]
9     Next           0     3     0                    01               
10    Close          0     0     0                    00               
11    Halt           0     0     0                    00               
12    Transaction    0     0     1     0              01  usesStmtJournal=0
13    TableLock      0     2     0     Tuotteet       00  iDb=0 root=2 write=0
14    String8        0     2     0     retiisi        00  r[2]='retiisi'
15    Goto           0     1     0                    00 
SQLite muuttaa kyselyn tietokannan sisäiseksi ohjelmaksi, joka hakee tietoa tauluista. Tässä tapauksessa ohjelman suoritus alkaa riviltä 12, jossa alkaa transaktio, ja sitten rivillä 14 rekisteriin 2 sijoitetaan hakuehdossa oleva merkkijono "retiisi". Tämän jälkeen suoritus siirtyy riville 1, jossa aloitetaan taulun Tuotteet käsittely, ja rivit 2–9 muodostavat silmukan, joka etsii hakuehtoa vastaavat rivit taulusta.

Voimme myös pyytää tiiviimmän suunnitelman laittamalla kyselyn eteen sanat EXPLAIN QUERY PLAN. Tällöin tulos voi olla seuraava:

sqlite> EXPLAIN QUERY PLAN SELECT * FROM Tuotteet WHERE nimi='retiisi';
0|0|0|SCAN TABLE Tuotteet
Tässä SCAN TABLE Tuotteet tarkoittaa, että kysely käy läpi taulun Tuotteet rivit.

Kyselyn optimointi
Jos kyselyssä haetaan tietoa vain yhdestä taulusta, kysely on yleensä helppo suorittaa, mutta todelliset haasteet tulevat vastaan usean taulun kyselyissä. Tällöin tietokantajärjestelmän tulee osata optimoida kyselyn suorittamista eli muodostaa hyvä suunnitelma, jonka avulla halutut tiedot saadaan kerättyä tehokkaasti tauluista.

Tarkastellaan esimerkkinä seuraavaa kyselyä, joka listaa kurssien ja opettajien nimet:

SELECT K.nimi, O.nimi FROM Kurssit K, Opettajat O WHERE K.opettaja_id = O.id;
Koska kysely kohdistuu kahteen tauluun, olemme ajatelleet kyselyn toiminnan niin, että se muodostaa ensin kaikki rivien yhdistelmät tauluista Kurssit ja Opettajat ja valitsee sitten ne rivit, joilla pätee ehto K.opettaja_id = O.id. Tämä on hyvä ajattelutapa, mutta tämä ei vastaa sitä, miten kunnollinen tietokantajärjestelmä toimii.

Ongelmana on, että tauluissa Kurssit ja Opettajat voi molemmissa olla suuri määrä rivejä. Esimerkiksi jos kummassakin taulussa on miljoona riviä, rivien yhdistelmiä olisi miljoona miljoonaa ja veisi valtavasti aikaa käydä läpi kaikki yhdistelmät.

Tässä tilanteessa tietokantajärjestelmän pitääkin ymmärtää, mitä käyttäjä oikeastaan on hakemassa ja miten kyselyssä annettu ehto rajoittaa tulosrivejä. Käytännössä riittää käydä läpi kaikki taulun Kurssit rivit ja etsiä jokaisen rivin kohdalla jotenkin tehokkaasti yksittäinen haluttu rivi taulusta Opettajat.

Voimme taas pyytää SQLiteä selittämään kyselyn suunnitelman:

sqlite> EXPLAIN QUERY PLAN SELECT K.nimi, O.nimi FROM Kurssit K, Opettajat O WHERE K.opettaja_id = O.id;
0|0|0|SCAN TABLE Kurssit AS K
0|1|1|SEARCH TABLE Opettajat AS O USING INTEGER PRIMARY KEY (rowid=?)
Tämä kysely käy läpi taulun Kurssit rivit (SCAN TABLE Kurssit) ja hakee tietoa taulusta Opettajat pääavaimen avulla (SEARCH TABLE Opettajat). Jälkimmäinen tarkoittaa, että kun käsittelyssä on tietty taulun Kurssit rivi, kysely hakee tehokkaasti taulusta Opettajat rivin, jossa pääavain O.id on sama kuin K.opettaja_id.

Mutta miten käytännössä taulusta Opettajat voi hakea tehokkaasti? Tämä onnistuu käyttämällä indeksiä, joihin tutustumme heti seuraavassa aliluvussa.






Indeksit
Indeksi on tietokannan taulun yhteyteen tallennettu hakemistorakenne, jonka tavoitteena on tehostaa tauluun liittyvien kyselyiden suorittamista. Indeksin avulla tietokantajärjestelmä voi selvittää tehokkaasti, missä päin taulua on rivejä, jotka täsmäävät tiettyyn hakuehtoon.

Indeksiä voi ajatella samalla tavalla kuin kirjan lopussa olevaa hakemistoa, jossa kerrotaan hakusanoista, millä kirjan sivuilla ne esiintyvät. Hakemiston avulla löydämme tietyn sanan sijainnit paljon nopeammin kuin lukemalla koko kirjan läpi.

Pääavaimen indeksi
Kun tietokantaan luodaan taulu, sen pääavain saa automaattisesti indeksin. Tämän seurauksena voimme suorittaa tehokkaasti hakuja, joissa ehto liittyy pääavaimeen.

Esimerkiksi kun luomme SQLitessä taulun

CREATE TABLE Tuotteet (id INTEGER PRIMARY KEY, nimi TEXT, hinta INTEGER);
niin taululle luodaan indeksi sarakkeelle id ja voimme etsiä tehokkaasti tuotteita id-numeron perusteella. Tämän ansiosta esimerkiksi seuraava kysely toimii tehokkaasti:

SELECT hinta FROM Tuotteet WHERE id=3;
Voimme varmistaa tämän kysymällä kyselyn suunnitelman:

sqlite> EXPLAIN QUERY PLAN SELECT hinta FROM Tuotteet WHERE id=3;
selectid    order       from        detail                                                   
----------  ----------  ----------  ---------------------------------------------------------
0           0           0           SEARCH TABLE Tuotteet USING INTEGER PRIMARY KEY (rowid=?)
Suunnitelmassa näkyy SEARCH TABLE, mikä tarkoittaa, että kysely pystyy hakemaan taulusta tietoa tehokkaasti indeksin avulla.

Indeksin luominen
Pääavaimen indeksi on kätevä, mutta voimme haluta myös etsiä tietoa jonkin muun sarakkeen perusteella. Esimerkiksi seuraava kysely hakee rivit sarakkeen hinta perusteella:

SELECT nimi FROM Tuotteet WHERE hinta=4;
Tämä kysely ei ole oletuksena tehokas, koska sarakkeelle hinta ei ole indeksiä. Näemme tämän pyytämällä taas selitystä kyselystä:

sqlite> EXPLAIN QUERY PLAN SELECT nimi FROM Tuotteet WHERE hinta=4;
selectid    order       from        detail             
----------  ----------  ----------  -------------------
0           0           0           SCAN TABLE Tuotteet
Nyt suunnitelmassa näkyy SCAN TABLE, mikä tarkoittaa, että kysely joutuu käymään läpi taulun kaikki rivit. Tämä on hidasta, jos taulussa on paljon rivejä.

Voimme kuitenkin luoda uuden indeksin, joka tehostaa saraketta hinta käyttäviä kyselyitä. Saamme luotua indeksin komennolla CREATE INDEX näin:

CREATE INDEX idx_hinta ON Tuotteet (hinta);
Tässä idx_hinta on indeksin nimi, jolla voimme viitata siihen myöhemmin. Indeksi toimii luonnin jälkeen täysin automaattisesti, eli tietokantajärjestelmä osaa käyttää sitä kyselyissä ja huolehtii sen päivittämisestä.

Miten indeksi toimii?
Indeksi tarvitsee tuekseen hakemistorakenteen, josta voi hakea tehokkaasti rivejä sarakkeen arvon perusteella. Tämä voidaan toteuttaa esimerkiksi puurakenteena, jonka avaimina on sarakkeiden arvoja.

Asiaan liittyvää teoriaa käsitellään tarkemmin kurssilla Tietorakenteet ja algoritmit binäärihakupuiden yhteydessä. Tyypillisiä tietokantojen yhteydessä käytettäviä puurakenteita ovat B-puu ja sen muunnelmat.

Indeksin luomisen jälkeen voimme kysyä uudestaan kyselyn suunnitelmaa:

sqlite> EXPLAIN QUERY PLAN SELECT nimi FROM Tuotteet WHERE hinta=4;
selectid    order       from        detail                                               
----------  ----------  ----------  -----------------------------------------------------
0           0           0           SEARCH TABLE Tuotteet USING INDEX idx_hinta (hinta=?)
Indeksin ansiosta suunnitelmassa ei lue enää SCAN TABLE vaan SEARCH TABLE. Suunnitelmassa näkyy myös, että aikomuksena on hyödyntää indeksiä idx_hinta.

Lisää käyttötapoja
Voimme käyttää indeksiä myös kyselyissä, joissa haemme pienempiä tai suurempia arvoja. Esimerkiksi sarakkeelle hinta luodun indeksin avulla voimme etsiä vaikkapa rivejä, joille pätee ehto hinta<3 tai hinta>=8.

Indeksi on myös mahdollista luoda usean sarakkeen perusteella. Esimerkiksi voisimme luoda indeksin näin:

CREATE INDEX idx_hinta ON Tuotteet (hinta,nimi);
Tässä indeksissä rivit on järjestetty ensisijaisesti hinnan ja toissijaisesti nimen mukaan. Indeksi tehostaa hakuja, joissa hakuperusteena on joko pelkkä hinta tai yhdessä hinta ja nimi. Kuitenkaan indeksi ei tehosta hakuja, joissa hakuperusteena on pelkkä nimi.

Milloin luoda indeksi?
Periaatteessa voisi ajatella, että taulun jokaiselle sarakkeelle kannattaa luoda indeksi, jolloin monenlaiset kyselyt ovat nopeita. Tämä ei ole kuitenkaan käytännössä hyvä idea.

Vaikka indeksit tehostavat kyselyitä, niissä on myös kaksi ongelmaa: indeksin hakemistorakenne vie tilaa ja indeksi myös hidastaa tiedon lisäämistä ja muuttamista. Jälkimmäinen johtuu siitä, että kun taulun sisältö muuttuu, niin muutos täytyy myös päivittää kaikkiin tauluun liittyviin indekseihin. Indeksiä ei siis kannata luoda huvin vuoksi.

Hyvä syy indeksin luontiin on, että haluamme suorittaa usein tietynlaisia kyselyitä ja ne toimivat hitaasti, koska tietokantajärjestelmä joutuu käymään läpi turhaan jonkin taulun kaikki rivit kyselyn aikana. Tällöin voimme lisätä taululle indeksin, jonka avulla tällaiset kyselyt toimivat jatkossa tehokkaasti.

Indekseillä on käytännössä suuri vaikutus tietokantojen tehokkuuteen. Moni tietokanta toimii hitaasti sen takia, että siitä puuttuu oleellisia indeksejä.





Toisteinen tieto
Ihanteena tietokannan suunnittelussa on, että tauluissa ei ole toisteista tietoa, jonka voi päätellä tietokannan muusta sisällöstä. Välillä kuitenkin tästä periaatteesta kannattaa joustaa, jotta saamme tehostettua kyselyitä.

Esimerkki
Tarkastellaan pankin tietokantaa, jossa taulu Tapahtumat sisältää tiedot tilitapahtumista. Jokaiseen tapahtumaan liittyy sarake muutos, joka ilmaisee, paljonko tilin saldo muuttuu (muutos voi olla positiivinen tai negatiivinen).

Seuraava kysely ilmoittaa tilin 123 nykyisen saldon laskemalla yhteen kaikki tiliin liittyvät muutokset:

SELECT SUM(muutos) FROM Tapahtumat WHERE tili_id=123;
Tämä on sinänsä mainio kysely, mutta se joutuu käymään läpi kaikki tiliin 123 liittyvät rivit taulusta Tapahtumat saadakseen selville lopullisen saldon. Tämä on tavallaan turhaa työtä, koska meitä ei kiinnosta tilin historia vaan vain saldo.

Voimmekin tehostaa kyselyä luomalla uuden taulun Saldot, joka sisältää jokaisen tilin saldon. Tämän taulun avulla saamme haettua tilin 123 saldon näin helposti:

SELECT saldo FROM Saldot FROM tili_id=123;
Tässä rikomme periaatetta, jonka mukaan tietokannassa ei saa olla muista tiedoista laskettavissa olevaa tietoa, koska taulun Saldot rivit olisi mahdollista laskea taulun Tapahtumat sisällön perusteella. Saamme kuitenkin näin aikaan tehokkaamman kyselyn.

Mitä haittaa periaatteen rikkomisesta on? Lähinnä se, että meidän on tästä lähtien hankalampaa päivittää tietokantaa. Aina kun lisäämme uuden tapahtuman tauluun Tapahtumat, meidän täytyy lisäksi tehdä muutos tauluun Saldot. Aiemmin saldo laskettiin suoraan tilitapahtumista, minkä ansiosta se oli varmasti aina ajan tasalla.

Denormalisointi
Tietokantojen teoriassa käytetään käsitettä denormalisointi, joka tarkoittaa tietokannan kyselyjen tehostamista lisäämällä toisteista tietoa. Tällöin liikumme tavallaan vastakkaiseen suuntaan kuin normalisoinnissa.

Muutokset vs. kyselyt
Usein esiintyvä ilmiö tietojenkäsittelytieteessä on, että joudumme tasapainoilemaan sen kanssa, haluammeko muuttaa vai hakea tehokkaasti tietoa ja paljonko tilaa voimme käyttää. Tämä tulee tietokantojen lisäksi vastaan esimerkiksi algoritmien suunnittelussa.

Jos tietokannassa ei ole toisteista tietoa, muutokset ovat helppoja, koska jokainen tieto on vain yhdessä paikassa eli riittää muuttaa vain yhden taulun yhtä riviä. Myös hyvänä puolena tietokanta vie vähän tilaa. Toisaalta kyselyt voivat olla monimutkaisia ja hitaita, koska halutut tiedot pitää kerätä kasaan eri puolilta tietokantaa.

Kun sitten lisäämme toisteista tietoa, pystymme nopeuttamaan kyselyjä mutta toisaalta muutokset hidastuvat, koska muutettu tieto pitää päivittää useaan paikkaan. Samaan aikaan myös tietokannan tilankäyttö kasvaa toisteisen tiedon takia.

Ei ole mitään yleistä sääntöä, paljonko toisteista tietoa kannattaa lisätä, vaan tämä riippuu tietokannan sisällöstä ja halutuista kyselyistä. Yksi hyvä tapa on aloittaa tilanteesta, jossa toisteista tietoa ei ole, ja lisätä sitten toisteista tietoa tarvittaessa, jos osoittautuu, että kyselyt eivät muuten ole riittävän tehokkaita.

Huomaa, että indeksit ovat myös yksi esimerkki siitä, miten toisteinen tieto voi tehostaa kyselyjä. Niissä kuitenkaan toisteista tietoa ei tallenneta tauluun vaan taulun ulkopuolelle erilliseen hakemistorakenteeseen.
