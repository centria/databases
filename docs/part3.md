---
title: "Multiple tables"
permalink: /part3/
nav_order: 4
published: true
---

# Multiple tables


Viittaukset ja kyselyt
Keskeinen idea relaatiotietokannoissa on, että taulun rivi voi viitata toisen taulun riviin. Tällöin voimme muodostaa kyselyn, joka kerää tietoa useista tauluista viittausten perusteella. Käytännössä viittauksena on yleensä toisessa taulussa olevan rivin id-numero.

Esimerkki
Tarkastellaan esimerkkinä tilannetta, jossa tietokannassa on tietoa kursseista ja niiden opettajista. Oletamme, että jokaisella kurssilla on yksi opettaja ja sama opettaja voi opettaa monta kurssia.

Tallennamme tauluun Opettajat tietoa opettajista. Jokaisella opettajalla on id-numero, jolla voimme viitata siihen.

id          nimi      
----------  ----------
1           Kaila     
2           Luukkainen
3           Kivinen   
4           Laaksonen 
Taulussa Kurssit on puolestaan tietoa kursseista ja jokaisen kurssin kohdalla viittaus kurssin opettajaan.

id          nimi              opettaja_id
----------  ----------------  -----------
1           Laskennan mallit  3          
2           Ohjelmoinnin per  1          
3           Ohjelmoinnin jat  1          
4           Tietokantojen pe  4          
5           Tietorakenteet j  3        
Voimme nyt hakea kurssit opettajineen seuraavalla kyselyllä, joka hakee tietoa samaan aikaan tauluista Kurssit ja Opettajat:

SELECT Kurssit.nimi, Opettajat.nimi
FROM Kurssit, Opettajat
WHERE Kurssit.opettaja_id = Opettajat.id;
Koska kyselyssä on monta taulua, ilmoitamme sarakkeiden taulut. Esimerkiksi Kurssit.nimi viittaa taulun Kurssit sarakkeeseen nimi.

Kysely antaa seuraavan tuloksen:

nimi              nimi      
----------------  ----------
Laskennan mallit  Kivinen   
Ohjelmoinnin per  Kaila     
Ohjelmoinnin jat  Kaila     
Tietokantojen pe  Laaksonen 
Tietorakenteet j  Kivinen 
Mitä tässä tapahtui?
Yllä olevassa kyselyssä uutena asiana on, että kysely koskee useaa taulua (FROM Kurssit, Opettajat), mutta mitä tämä tarkoittaa oikeastaan?

Ideana on, että kun kyselyssä on monta taulua, kyselyn tulosrivien lähtökohtana ovat kaikki tavat valita rivien yhdistelmiä tauluista. Tämän jälkeen WHERE-osan ehdoilla voi määrittää, mitkä yhdistelmät ovat kiinnostuksen kohteena.

Hyvä tapa saada ymmärrystä monen taulun kyselyn toiminnasta on tarkastella ensin kyselyä, joka hakee kaikki sarakkeet ja jossa ei ole WHERE-osaa. Yllä olevassa esimerkkitilanteessa tällainen kysely on seuraava:

SELECT * FROM Kurssit, Opettajat;
Koska taulussa Kurssit on 5 riviä ja taulussa Opettajat on 4 riviä, kyselyn tulostaulussa on 5 * 4 = 20 riviä. Tulostaulu sisältää kaikki mahdolliset tavat valita ensin jokin rivi taulusta Kurssit ja sitten jokin rivi taulusta Opettajat:

id          nimi              opettaja_id  id          nimi      
----------  ----------------  -----------  ----------  ----------
1           Laskennan mallit  3            1           Kaila     
1           Laskennan mallit  3            2           Luukkainen
1           Laskennan mallit  3            3           Kivinen   
1           Laskennan mallit  3            4           Laaksonen 
2           Ohjelmoinnin per  1            1           Kaila     
2           Ohjelmoinnin per  1            2           Luukkainen
2           Ohjelmoinnin per  1            3           Kivinen   
2           Ohjelmoinnin per  1            4           Laaksonen 
3           Ohjelmoinnin jat  1            1           Kaila     
3           Ohjelmoinnin jat  1            2           Luukkainen
3           Ohjelmoinnin jat  1            3           Kivinen   
3           Ohjelmoinnin jat  1            4           Laaksonen 
4           Tietokantojen pe  4            1           Kaila     
4           Tietokantojen pe  4            2           Luukkainen
4           Tietokantojen pe  4            3           Kivinen   
4           Tietokantojen pe  4            4           Laaksonen 
5           Tietorakenteet j  3            1           Kaila     
5           Tietorakenteet j  3            2           Luukkainen
5           Tietorakenteet j  3            3           Kivinen   
5           Tietorakenteet j  3            4           Laaksonen 
Suurin osa tulosriveistä ei ole kuitenkaan kiinnostavia, koska ne eivät liity toisiinsa mitenkään. Esimerkiksi ensimmäinen tulosrivi kertoo vain, että on olemassa kurssi Laskennan mallit ja toisaalta on olemassa opettaja Kaila. Tämän vuoksi rajaamme hakua niin, että opettajan id-numeron tulee olla sama kummankin taulun riveissä:

SELECT * FROM Kurssit, Opettajat
WHERE Kurssit.opettaja_id = Opettajat.id;
Tämän seurauksena kysely alkaa antaa mielekkäitä tuloksia:

id          nimi              opettaja_id  id          nimi      
----------  ----------------  -----------  ----------  ----------
1           Laskennan mallit  3            3           Kivinen   
2           Ohjelmoinnin per  1            1           Kaila     
3           Ohjelmoinnin jat  1            1           Kaila     
4           Tietokantojen pe  4            4           Laaksonen 
5           Tietorakenteet j  3            3           Kivinen   
Tämän jälkeen voimme vielä parantaa kyselyä ilmoittamalla meitä kiinnostavat sarakkeet:

SELECT Kurssit.nimi, Opettajat.nimi
FROM Kurssit, Opettajat
WHERE Kurssit.opettaja_id = Opettajat.id;
Näin päädymme samaan tulokseen kuin aiemmin:

nimi              nimi      
----------------  ----------
Laskennan mallit  Kivinen   
Ohjelmoinnin per  Kaila     
Ohjelmoinnin jat  Kaila     
Tietokantojen pe  Laaksonen 
Tietorakenteet j  Kivinen 
Lisää ehtoja kyselyssä
Monen taulun kyselyissä WHERE-osa kytkee toisiinsa meitä kiinnostavat taulujen rivit, mutta lisäksi voimme laittaa WHERE-osaan muita ehtoja samaan tapaan kuin ennenkin. Esimerkiksi voimme suorittaa seuraavan kyselyn:

SELECT Kurssit.nimi, Opettajat.nimi
FROM Kurssit, Opettajat
WHERE Kurssit.opettaja_id = Opettajat.id AND Opettajat.nimi = 'Kivinen';
Näin saamme haettua kurssit, joiden opettajana on Kivinen:

nimi              nimi      
----------------  ----------
Laskennan mallit  Kivinen   
Tietorakenteet j  Kivinen 
Taulujen lyhyet nimet
Voimme tiivistää monen taulun kyselyä antamalla tauluille vaihtoehtoiset lyhyet nimet, joiden avulla voimme viitata niihin kyselyssä. Esimerkiksi kysely

SELECT Kurssit.nimi, Opettajat.nimi
FROM Kurssit, Opettajat
WHERE Kurssit.opettaja_id = Opettajat.id;
voidaan esittää lyhemmin näin:

SELECT K.nimi, O.nimi
FROM Kurssit AS K, Opettajat AS O
WHERE K.opettaja_id = O.id;
Itse asiassa sana AS ei ole pakollinen, eli voimme lyhentää kyselyä lisää:

SELECT K.nimi, O.nimi
FROM Kurssit K, Opettajat O
WHERE K.opettaja_id = O.id;
Saman taulun toistaminen
Monen taulun kyselyssä voi esiintyä myös monta kertaa sama taulu, kunhan niille annetaan eri nimet. Esimerkiksi seuraava kysely hakee kaikki tavat valita kahden opettajan pari:

SELECT A.nimi, B.nimi FROM Opettajat A, Opettajat B;
Kyselyn tulos on seuraava:

nimi        nimi      
----------  ----------
Kaila       Kaila     
Kaila       Luukkainen
Kaila       Kivinen   
Kaila       Laaksonen 
Luukkainen  Kaila     
Luukkainen  Luukkainen
Luukkainen  Kivinen   
Luukkainen  Laaksonen 
Kivinen     Kaila     
Kivinen     Luukkainen
Kivinen     Kivinen   
Kivinen     Laaksonen 
Laaksonen   Kaila     
Laaksonen   Luukkainen
Laaksonen   Kivinen   
Laaksonen   Laaksonen 







Liitostaulu
Taulujen välillä esiintyy yleensä kahdenlaisia suhteita:

Yksi moneen -suhde: Taulun A rivi liittyy enintään yhteen taulun B riviin. Taulun B rivi voi liittyä useaan taulun A riviin.
Monta moneen -suhde: Taulun A rivi voi liittyä useaan taulun B riviin. Taulun B rivi voi liittyä useaan taulun A riviin.
Tapauksessa 1 voimme lisätä tauluun A sarakkeen, joka viittaa tauluun B, kuten teimme edellisen aliluvun esimerkissä. Tapauksessa 2 tilanne on kuitenkin hankalampi, koska yksittäinen viittaus kummankaan taulun rivissä ei riittäisi. Ratkaisuna on luoda kolmas liitostaulu, joka sisältää tiedot viittauksista.

Esimerkki
Tarkastellaan esimerkkinä tilannetta, jossa verkkokaupassa on tuotteita ja asiakkaita ja jokainen asiakas on valinnut tiettyjä tuotteita ostoskoriin. Tietyn asiakkaan korissa voi olla useita tuotteita, ja toisaalta tietty tuote voi olla usean asiakkaan korissa.

Rakennamme tietokannan niin, että siinä on kolme taulua: Tuotteet, Asiakkaat ja Ostokset. Liitostaulu Ostokset ilmaisee, mitä tuotteita on kunkin asiakkaan ostoskorissa. Sen jokainen rivi esittää yhden parin muotoa "asiakkaan x korissa on tuote y".

Oletamme, että taulujen sisällöt ovat seuraavat:


Nyt voimme hakea asiakkaat ja tuotteet seuraavasti:

SELECT A.nimi, T.nimi
FROM Asiakkaat A, Tuotteet T, Ostokset O
WHERE A.id = O.asiakas_id AND T.id = O.tuote_id;
Kyselyn ideana on hakea tauluista Asiakkaat ja Tuotteet taulun Ostokset rivejä vastaavat tiedot. Jotta saamme mielekkäitä tuloksia, kytkemme rivit yhteen kahden ehdon avulla. Kysely tuottaa seuraavan tulostaulun:

nimi        nimi      
----------  ----------
Uolevi      porkkana  
Uolevi      selleri   
Maija       retiisi   
Maija       lanttu    
Maija       selleri   
Samalla idealla voimme vaikkapa selvittää, mitä tuotteita on tietyn asiakkaan ostoskorissa. Esimerkiksi seuraava kysely hakee Maijan korissa olevat tuotteet:

SELECT T.nimi
FROM Asiakkaat A, Tuotteet T, Ostokset O
WHERE A.id = O.asiakas_id AND T.id = O.tuote_id AND A.nimi = 'Maija';
Kysely tulos on seuraava:

nimi      
----------
retiisi   
lanttu    
selleri   
Yhteenveto tauluista
Voimme käyttää koostefunktioita ja ryhmittelyä myös usean taulun kyselyissä. Esimerkiksi voimme luoda yhteenvedon, joka näyttää jokaisesta asiakkaasta, montako tuotetta hänen ostoskorissaan on ja mikä on tuotteiden yhteishinta. Tämä onnistuu seuraavasti:

SELECT A.nimi, COUNT(T.id), SUM(T.hinta)
FROM Asiakkaat A, Tuotteet T, Ostokset O
WHERE A.id = O.asiakas_id AND T.id = O.tuote_id
GROUP BY A.id;
Miten ryhmitellään?
Tässä kyselyssä ryhmittely tapahtuu sarakkeen A.id mukaan, mutta kyselyssä haetaan sarake A.nimi. Tämä on sinänsä järkevää, koska sarake A.id määrää sarakkeen A.nimi, ja kysely toimii mainiosti SQLitessä.

Muissa tietokannoissa (kuten PostgreSQL:ssä) vaatimuksena voi kuitenkin olla, että sellaisenaan haettavan sarakkeen tulee aina esiintyä myös ryhmittelyssä. Tällöin ryhmittelyn tulee olla GROUP BY A.id, A.nimi.

Kyselyn ideana on ryhmitellä rivit asiakkaan id-numeron mukaan, jolloin funktio COUNT(T.id) antaa tuotteiden määrän asiakkaan korissa ja funktio SUM(T.hinta) antaa tuotteiden yhteishinnan. Kyselyn tulos on seuraava:

nimi        COUNT(T.id)  SUM(T.hinta)
----------  -----------  ------------
Uolevi      2            9           
Maija       3            19          
Tämä tarkoittaa, että Uolevin ostoskorissa on 2 tuotetta, joiden yhteishinta on 9, ja Maijan ostoskorissa on puolestaan 3 tuotetta, joiden yhteishinta on 19. Kaikki näyttää hyvältä – vai näyttääkö sittenkään?

Yhteenvedon puutteena on vielä, että siinä ei ole lainkaan kolmatta tietokannassa olevaa asiakasta eli Aapelia. Olemme törmänneet ongelmaan, mutta onneksi löydämme siihen ratkaisun seuraavan aliluvun lopussa.






JOIN-syntaksi
Tähän mennessä olemme hakeneet tietoa tauluista listaamalla taulut kyselyn FROM-osassa, mikä toimii yleensä hyvin. Kuitenkin joskus on tarpeen vaihtoehtoinen JOIN-syntaksi. Siitä on hyötyä silloin, kun kyselyn tuloksesta näyttää "puuttuvan" tietoa.

Kyselytavat
Seuraavassa on kaksi tapaa toteuttaa sama kysely, ensin käyttäen ennestään tuttua tapaa ja sitten käyttäen JOIN-syntaksia.

SELECT Kurssit.nimi, Opettajat.nimi
FROM Kurssit, Opettajat
WHERE Kurssit.opettaja_id = Opettajat.id;
SELECT Kurssit.nimi, Opettajat.nimi
FROM Kurssit JOIN Opettajat ON Kurssit.opettaja_id = Opettajat.id;
JOIN-syntaksissa taulujen nimien välissä esiintyy sana JOIN ja lisäksi taulujen rivit toisiinsa kytkevä ehto annetaan erillisessä ON-osassa. Tämän jälkeen olisi mahdollista laittaa muita ehtoja WHERE-osaan, kuten ennenkin.

Tässä tapauksessa JOIN-syntaksi on vain vaihtoehtoinen tapa toteuttaa kysely eikä se tuo mitään uutta. Kuitenkin näemme seuraavaksi, miten voimme laajentaa syntaksia niin, että se antaa meille uusia mahdollisuuksia kyselyissä.

Puuttuvan tiedon ongelma
Tarkastellaan esimerkkinä tilannetta, jossa tietokannassa on tutut taulut Kurssit ja Opettajat, mutta taulussa Kurssit yhdeltä kurssilta puuttuu opettaja:

id          nimi              opettaja_id
----------  ----------------  -----------
1           Laskennan mallit  3          
2           Ohjelmoinnin per  1          
3           Ohjelmoinnin jat  1          
4           Tietokantojen pe             
5           Tietorakenteet j  3          
Rivin 4 sarakkeessa opettaja_id on arvo NULL, joten jos suoritamme jommankumman äskeisen kyselyn, ongelmaksi tulee, että rivi 4 ei täsmää mihinkään taulun Opettajat riviin. Tämän seurauksena tulostauluun ei tule riviä kurssista Tietokantojen perusteet:

nimi              nimi      
----------------  ----------
Laskennan mallit  Kivinen   
Ohjelmoinnin per  Kaila     
Ohjelmoinnin jat  Kaila     
Tietorakenteet j  Kivinen 
Ratkaisu ongelmaan on käyttää LEFT JOIN -syntaksia, joka tarkoittaa, että mikäli jokin vasemman taulun rivi ei yhdisty mihinkään oikean taulun riviin, kyseinen vasemman taulun rivi pääsee silti mukaan yhdeksi riviksi tulostauluun. Kyseisellä rivillä jokaisen oikeaan tauluun perustuvan sarakkeen arvona on NULL.

Tässä tapauksessa voimme toteuttaa kyselyn näin:

SELECT Kurssit.nimi, Opettajat.nimi
FROM Kurssit LEFT JOIN Opettajat ON Kurssit.opettaja_id = Opettajat.id;
Nyt tulostauluun ilmestyy myös kurssi Tietokantojen perusteet ilman opettajaa:

nimi              nimi      
----------------  ----------
Laskennan mallit  Kivinen   
Ohjelmoinnin per  Kaila     
Ohjelmoinnin jat  Kaila     
Tietokantojen pe            
Tietorakenteet j  Kivinen   
JOIN-kyselyperhe
Itse asiassa JOIN-kyselystä on olemassa peräti neljä eri muunnelmaa:

JOIN: toimii kuten tavallinen kahden taulun kysely
LEFT JOIN: jos vasemman taulun rivi ei yhdisty mihinkään oikean taulun riviin, se valitaan kuitenkin mukaan erikseen
RIGHT JOIN: jos oikean taulun rivi ei yhdisty mihinkään vasemman taulun riviin, se valitaan kuitenkin mukaan erikseen
FULL JOIN: sekä vasemmasta että oikeasta taulusta valitaan erikseen mukaan rivit, jotka eivät yhdisty toisen taulun riviin
SQLiten rajoituksena on kuitenkin, että vain kaksi ensimmäistä kyselytapaa ovat mahdollisia. Onneksi LEFT JOIN on yleensä se, mitä haluamme.

Puuttuva tieto yhteenvedossa
Nyt voimme pureutua edellisen aliluvun ongelmaan, jossa yhteenvetokyselystä puuttui tietoa. Tietokannassamme on seuraavat taulut:


Muodostimme yhteenvedon ostoskoreista seuraavalla kyselyllä:

SELECT A.nimi, COUNT(T.id), SUM(T.hinta)
FROM Asiakkaat A, Tuotteet T, Ostokset O
WHERE A.id = O.asiakas_id AND T.id = O.tuote_id
GROUP BY A.id;
Kuitenkin ongelmaksi tuli, että Aapeli puuttuu yhteenvedosta:

nimi        COUNT(T.id)  SUM(T.hinta)
----------  -----------  ------------
Uolevi      2            9
Maija       3            19
Ongelman syynä on, että Aapelin ostoskori on tyhjä eli kun kysely valitsee yhdistelmiä taulujen riveistä, ei ole mitään sellaista riviä, jolla esiintyisi Aapeli. Ratkaisu ongelmaan on käyttää LEFT JOIN -syntaksia näin:

SELECT A.nimi, COUNT(T.id), SUM(T.hinta)
FROM Asiakkaat A LEFT JOIN Ostokset O ON A.id = O.asiakas_id
                 LEFT JOIN Tuotteet T ON T.id = O.tuote_id
GROUP BY A.id;
Nyt myös Aapeli ilmestyy mukaan yhteenvetoon:

nimi        COUNT(T.id)  SUM(T.hinta)
----------  -----------  ------------
Uolevi      2            9           
Maija       3            19          
Aapeli      0                     
Koska Aapelin ostoskorissa ei ole tuotteita, hintojen summaksi tulee NULL.