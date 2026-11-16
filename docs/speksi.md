# ATSEEE-koulubussikartta — tekninen määrittely

Versio 0.1 · 16.11.2026 · Eurooppa-koulu I, Kirchberg

Tämä dokumentti toimii toteutuksen lähtökohtana ja arviointipyyntönä. Suositeltu toteutus on pieni selainkäyttöinen kartta, jonka datalähteenä on olemassa oleva Excel. Kävelyreititin liitetään erillisen rajapinnan kautta. Sama aikataululaskenta palvelee myöhemmin isokronikarttaa ja asuntohakua.

## 1. Käyttötapaukset ja rajaus

| Vaihe | Käyttäjän toiminto | Tulos |
|---|---|---|
| A. Linjakartta | Valitse linja ja vuoro tai hae pysäkkiä nimellä | Kartta näyttää pysäkit ja linjan. Aikataulurivi keskittää kartan pysäkille ja karttamerkki avaa aikataulun. |
| B. Reittihaku | Valitse kodin sijainti kartalta ja koulumatkan suunta sekä kellonaika | Kävely, odotus, suora ATSEEE-vuoro ja kokonaisaika. Vaihtoehtoiset pysäkit ja vuorot näkyvät tuloksessa. |
| C. Saavutettavuus | Aseta aikaikkuna ja halutessa enimmäismatka-aika | Kartta näyttää arvioidut alueet, joilta kouluun ehtii tai joille koululta pääsee. |
| D. Asuntohaku | Anna karttapiste, myöhemmin osoite, aikarajat ja suunta | Sama reittihaku palauttaa koneellisesti luettavan tuloksen osoitekohtaiseen vertailuun. |

Ensimmäinen käyttökelpoinen versio sisältää A:n ja B:n. Toteuta A ensin ja testaa B aluksi linjoilla 53 ja 59. Laajenna sama tuonti ja laskenta kaikkiin 19 aineistossa olevaan linjaan ennen ensimmäisen version luovutusta. Käytä yhtä piste paikkaa kohti. Ensimmäisen version kävelymatka yhdistää kodin ja asuinalueen pysäkin, koulun päätepiste on EE1. Käyttäjä voi valita oletetun aikataulusarakkeen ilman päivämäärää. Päivämääräpohjainen haku edellyttää vahvistettua liikennöintikalenteria. Vaihdot, muu joukkoliikenne, geokoodaus ja reaaliaikatieto ovat mahdollisia jatkotöitä.

## 2. Lähtöaineisto ja tuonti

**Aineisto.** Täydennetty työkirja on repossa: `data/source/atseee-aikataulut.xlsx`. Välilehti on `aikataulut`. Linjaotsikot ovat A:ssa, pysäkkinimet B:ssä, lat/lon C–D:ssä, pysäkkivastaavuuden tiedot E–J:ssä ja Morning/Lunch/Evening K–M:ssä. Karttakuvat ovat visuaalista taustamateriaalia. Aikataulujen nykyinen versio sopii prototyypin lähtöaineistoksi. Näytä aineiston versio käyttöliittymässä.

**Rakenteen käsittely.** Tunnista linjalohkot sarakkeesta A ja tarkista jokaisen lohkon otsikkorivi. Lue kellonajat Excelin kellonaikatyypistä tai validoidusta kellonaikamerkkijonosta. Muunna sekunneiksi paikallisesta keskiyöstä. Erottele `x` eli vuoro ohittaa pysäkin ja tyhjä eli tietoa puuttuu. Tyhjä tai virheellinen arvo raportoidaan ja käsitellään avoimena tietona. Rakenteeton lohko raportoidaan rivinumeroineen. N CPE KI on otsikko ilman pysäkkirivejä.

**Järjestys vuoroittain.** Kaikissa 19 lohkossa viimeinen rivi on Kirchberg EE1 ja aamun kellonajat etenevät rivijärjestyksessä. Paluuvuorojen järjestys muodostetaan kellonajan mukaan. Linjoilla 59, 60 ja 63 on poikkeamia pelkästä käänteisestä rivijärjestyksestä. Esimerkiksi linjalla 59 iltapäivällä Rue des Promenades klo 16.45 on ennen Hubertuspadia klo 16.46. Tasakellonajoissa käytä paluun käänteistä lähderivijärjestystä ja varmista koulupisteen rooli lähtöpisteenä. Jos tarkempi järjestys tarvitaan myöhemmin, tue vuorokohtaista käsin annettua järjestystä. Suora haku kouluun tai koululta toimii myös silloin, kun kahdella välipysäkillä on sama minuutti.

**Varmennus tuotannossa.** Pidä Excel syötteenä ja generoitu JSON rakennustuotteena. Kirjaa tuontiraporttiin linjat, pysäkkirivit, pois jääneet aikataulusolut, koordinaattipuutteet ja järjestyspoikkeamat. Tarkista, että jokaisella reititettävällä vuorolla on kelvollinen koulun kellonaika ja vähintään yksi asuinalueen pysäkki. Varmista kellonaikojen ja koordinaattien rajat sekä ID-viittaukset. Säilytä GTFS-tunnukset merkkijonoina etunollineen. Raportoi puutteet ennen julkaisua. Aja tuonti uudelleen samalla komennolla aina, kun työkirja päivittyy.

### Tietomalli

| Tietue | Kentät ja säännöt |
|---|---|
| Dataset | schemaVersion, dataVersion, sourceWorkbook, timeZone = Europe/Luxembourg, generatedAt, schoolStopId |
| Stop | id, name, lat, lon. Valinnaiset osmUrl, gtfsId, positionSource ja matchStatus säilyttävät lähdetiedon. EE1:lle käytetään yhtä yhteistä tunnusta. Yhdistä muiden linjojen pysäkit vain vahvistetulla yhteisellä tunnuksella tai täsmäävällä normalisoidulla nimellä ja pisteellä. |
| Line | id merkkijonona, label, sourceUrl, displayStopIds |
| Trip | id, lineId, direction (to_school/from_school), slot (morning/lunch/evening), serviceId tai null |
| StopTime | tripId, stopId, sequence, timeSeconds ja sourceRow. Koululla aikaan sovelletaan vuoron suunnan mukaista lähtö- tai saapumismerkitystä. Muissa pisteissä aamun aika on nousuaika ja paluun aika poistumisaika. |
| Service | weekdays, validFrom, validTo ja dateExceptions. Tarvitaan päiväkohtaiselle haulle. Epävarma kalenteri voidaan jättää tyhjäksi ja käyttää nimenomaista vuorovalintaa. |

Data voidaan toimittaa selaimelle yhtenä versiona `data.json`. Muunna pisteet kartan rajalla GeoJSONin [lon, lat] -järjestykseen. Ehdotettu rakenne on `scripts/import_xlsx.py`, `web/`, `web/data/data.json`, `tests/` ja `README.md`. Tutki yhteistä aineistoa selaimessa ilman tietokantaa. Tuloksista voidaan tarvittaessa muodostaa myöhemmin GTFS, jolloin tuotetaan myös vuorojen liikennöintikalenteri.

## 3. Kartta ja käyttöliittymä

- Avaa kartta EE1:n ja linjaverkon yleiskuvaan. Linjavalinnan jälkeen sovita näkymä linjan pisteisiin. Valinta näyttää kyseisen linjan aikataulun ja korostaa pysäkit.
- Pysäkkihaku käyttää olemassa olevia nimiä, hyväksyy aksentittoman kirjoitusasun ja palauttaa samannimisille pisteille linjat tai paikkakunnan. Klikkaus toimii molempiin suuntiin kartan ja aikataulun välillä.
- Aikataulussa näkyvät Morning/Lunch/Evening tai niiden suomenkieliset vastineet. Käyttäjä saa valita vuoron, jolloin karttaviiva seuraa kyseisen vuoron järjestystä ja ohitetut pysäkit merkitään.
- Piirrä ensin viiva välietapeista. Jos tieverkkoa seuraava viiva halutaan, laske se vuorojärjestyksittäin etukäteen autoprofiililla ja kuvaa se arvioiduksi ajolinjaksi. Säilytä bussin ajoaika Excelistä. Piirrä laskettu kävelyreitti omalla viivatyylillään.
- Mobiilissa kartta ja aikataulu toimivat samassa näkymässä esimerkiksi avattavalla alapaneelilla. Tarjoa myös tekstimuotoinen reittitulos, kartalta valitun kotipisteen vaihto ja selkeä tila, kun reittiä ei löydy.
- Näytä pohjakartan attribuutio, dataversio ja varsinainen aikataululähde. Reitityspalvelun poikkeuksessa linjakartta ja aikataulut toimivat edelleen.

## 4. Reittihaku ja toteutusrajapinta

**Syöte.** Kotipiste, suunta, vuorovalinta tai myöhemmin palvelupäivä, sekä ajan tulkinta `depart_at` tai `arrive_by`. Käytä paikallisaikaa. Enimmäiskävelyaika ja nousuvara ovat muuttujia. Ehdotettu oletus nousuvaralle prototyypissä on nolla, jolloin tulokset voi verrata suoraan taulukkoon. Arvo voi myöhemmin olla käyttäjän valittavissa.

**Haku.** Rajaa ensin kävelyetäisyyden perusteella mahdolliset pysäkit. Laske kävelyaika verkkoa pitkin matriisipyynnöllä tai pienellä määrällä tavallisia reittipyyntöjä. Pyydä kävelyviiva vain näytettäville vaihtoehdoille. Aamulla kävelyn suunta on kodista pysäkille ja paluussa pysäkiltä kotiin. Käytä vain vuoroja, joilla pisteellä on kellonaika. Haku voi laajentaa ehdokasjoukkoa, jos ensimmäinen erä ei tuota reittiä. Tuloksen mahdollinen ehdokasraja kannattaa tuoda näkyviin, sillä lähin pysäkki ei välttämättä tuota parasta koulumatkaa.

| Haku | Kelpoisuusehto | Ensisijainen järjestys |
|---|---|---|
| Aamu, lähtö aikaisintaan | Kotilähtö + kävely + nousuvara ≤ bussin lähtö pysäkiltä | Aikaisin saapuminen kouluun |
| Aamu, perillä viimeistään | Kouluun saapuminen ≤ määräaika. Laske viimeinen mahdollinen kotilähtö pysäkin lähtöajasta miinus kävely ja nousuvara. | Myöhäisin mahdollinen kotilähtö |
| Paluu, lähtö aikaisintaan | Koululta lähtö ≥ käyttäjän aika. Kotisaapuminen on pysäkin jättöaika + kävely. | Aikaisin saapuminen kotiin |
| Paluu, perillä viimeistään | Kotisaapuminen ≤ määräaika | Myöhäisin mahdollinen lähtö koululta |

Tasatilanteessa käytä lyhyempää kävelyaikaa ja vakaata ID-järjestystä. Palauta linja, vuoro, pysäkki, lähtö- ja saapumisajat, kävelysekunnit, bussisekunnit, odotusaika ja kokonaisaika. Pidä käyttäjän valitsemasta kellonajasta laskettu odotus mukana, kun haku alkaa silloin. Esitä arvioitu viimeinen lähtö erillisenä, jos tulokseen kuuluu myös odottelun välttäminen kotona.

**Rajapinta.** Pidä kävelypalvelu vaihdettavana kahdella operaatiolla `matrix(origins, destinations)` ja `route(origin, destination)`. Tulokset ovat sekunteja, metrejä ja GeoJSON-viiva. Vastaus erottaa reitittömän pisteparin palveluvirheestä. Selain voi tehdä aikataululaskennan paikallisesti. Avainta vaativan kävelypalvelun eteen sopii pieni saman sivuston palvelinfunktio, joka rajaa koordinaatit, pyyntökoon ja aikakatkaisun. Tee käyttöliittymän testiversio kiinteillä testivastauksilla, ja varmista oikean palvelun toiminta erillisellä integraatiotestillä. Pidä kotipiste selaimessa siihen asti, kun se tarvitaan kävelypyynnössä. Julkaistussa versiossa kerro, mille reitityspalvelulle koordinaatit välitetään.

## 5. Teknologiavalinnat ja hostaus

| Osa | Vaihtoehto | Arvio |
|---|---|---|
| Kartta ja tiilet | MapLibre GL JS + OpenFreeMap | Suositus. Vektoripohjakartta ja omat GeoJSON-tasot. OpenFreeMapilla on julkinen avaimeton palvelu ja itse hostattava vaihtoehto. |
| Kartta ja tiilet | Leaflet + OSM-pohjaiset rasteritiilet | Yksinkertainen selainkartta. Tiililähde valitaan erikseen, OSM:n julkisilla tiilillä on oma käyttöpolitiikka. |
| Kartta ja tiilet | OpenLayers | Sopii hyvin, jos käytetään suoraan WMS- tai muita paikkatietotasoja. |
| Kävelyreititys | openrouteservice | Suositus hostatulle kävelyhaulle. Directions, Matrix ja Isochrones tukevat jatkokehitystä. Palveluavaimen ja käyttörajojen käsittely tarvitaan. Saatavilla myös itse hostattavana. |
| Kävelyreititys | GraphHopper | Avoin moottori ja erillinen hostattu palvelu. Tarkista halutun profiilin, matriisin ja isokronien saatavuus valitussa tilauksessa. |
| Kävelyreititys | Valhalla | Avoin moottori, kävely ja isokronit. Julkinen demo kokeiluun. Itse hostaus sopii suurempaan laskentaan. |
| Kävelyreititys | OSRM | Avoin moottori, jossa kävely vaatii kävelyprofiililla rakennetun palvelun. FOSSGISin kävelydemo sopii pieneen kokeiluun. |
| Aikataulureititys | Selainlaskenta / OpenTripPlanner | Selainlaskenta riittää suoriin ATSEEE-matkoihin. OTP on jatkopolku, kun tarvitaan vaihtoja, kansallinen GTFS ja OSM samassa reitityspalvelussa. |

**Toteutus tässä Work-ympäristössä.** Pystyn kirjoittamaan Python-tuonnin, JavaScript-kartan, reittilaskennan ja testit sekä paketoimaan staattisen sivun. Yksityinen Sites-julkaisu on vaihtoehto staattiselle kartalle ja aikatauluille. Testaisin karttatiilien latauksen ja mobiilikäytön julkaisuympäristössä. Kävelyhaun ulkoisesta palvelusta testaisin yhden Luxembourg–pysäkki-pyynnön sekä selaimen cross-origin-käyttäytymisen tai saman originin palvelinreitin. Ulkoisen palvelun toimivuus testataan käyttöönotossa.

**Toteutus kehittäjälle.** Avaimen tarvitseva versio voidaan hostata esimerkiksi Cloudflare Pagesissa, missä Pages Function säilyttää avaimen salaisuutena. Sama rakenne toimii myös staattisella frontend-hostilla ja erillisellä pienellä backendillä. Jos avainta tai hostausta ei ole vielä käytössä, kartta ja aikatauluhaku voidaan arvioida paikallisella HTTP-palvelimella, kävelyintegraatio testivastauksilla ja kävely kokeellisesti julkisella demolla sen käyttöehtojen mukaan. Omalle palvelimelle voi asentaa reititysmoottorin ja kattaa Luxemburgin sekä käytetyt Saksan rajaseudut OSM-otteella. Ulkoinen palvelutili, API-avain, konttipalvelin tai käyttöoikeus tarvitaan vasta kyseistä vaihtoehtoa käytettäessä.

## 6. Isokroni ja asuntokytkentä

Aamun haussa annetaan aikaisin kotilähtö E ja viimeinen sallittu koulusaapuminen A. Karsi vuorot, joiden koulusaapuminen C on A:n jälkeen. Jokaiselle kelvolliselle nousupysäkille kävelybudjetti on pysäkin lähtö miinus E miinus nousuvara. Jos halutaan lisäksi enintään T minuutin ovelta koululle -matka, käytä aikaisimpana kotilähtönä suurempaa arvoista E ja C − T. Laske kävelyalue takaperin kohti pysäkkiä, jos reititin tukee sitä. Mikäli käytetään pysäkiltä alkavaa kävelyaluetta aamun arviona, ilmaise oletus.

Paluussa valitaan ensimmäinen mahdollinen lähtö koululta E, jonka täyttävät vuorot otetaan mukaan. Pysäkiltä kodin suuntaan lasketaan kävelyalue, jonka aikabudjetti on kokonaistakaraja miinus vuoron jättöaika. Jos käyttäjä antaa enimmäismatka-ajan E:stä, takaraja on E + T. Vähennä näin myös koululla syntyvä odotus. Hylkää negatiiviset kävelybudjetit. Yhdistä polygonit suunnittain ja tarvittaessa 30/45/60 minuutin tasoiksi. Toistuville vakiohauille sopivat ennakkoon lasketut GeoJSON-tasot. Rajoita palvelupyyntöjä ryhmittelemällä pysäkit ja välimuistittamalla toistuvat isokronit.

Asuntohaussa kutsutaan pistekohtaista reitinhakua samoilla aikaehdoilla. Polygonin sisällä olo toimii nopeaan esivalintaan. Yksittäinen kohde arvioidaan kävelyverkolla ja aikataululla, jotta tulos sisältää myös ehdotetun vuoron.

## 7. Hyväksymistestit ja luovutus

| Testi | Odotettu tulos |
|---|---|
| Excel-tuonti | 19 aineistollista linjaa, 225 pysäkkiriviä ja yksi yhteinen EE1-karttapiste. N CPE KI tunnistetaan tyhjäksi lohkoksi. |
| Linja 53, Mambra | Lähtö 07.18, EE1 08.10, bussissa 52 minuuttia. Kun kävely on testissä 6 minuuttia ja nousuvara 0, viimeinen lähtö kotoa on 07.12 ja ovelta koululle kuluu 58 minuuttia. |
| Linja 53, Cite Millewee | Aamuvuoro ohittaa pysäkin. Paluuvuoro näkyy ja on käytettävissä. |
| Linja 59, Itzig | Klo 16.45 Rue des Promenades ennen klo 16.46 Hubertuspadia. |
| Linja 60 ja 63 | Lounasvuorolla Am Eck 13.59 Hielin 13.56 jälkeen. Linjan 63 paluussa Hohensonne 14.40 ennen An der Heltiä 14.45. |
| Tasaminuutit ja puuttuvat tiedot | Järjestys on vakaa. Puuttuva aika tuottaa tuontihuomion. Hakutulos ei ilmoita puuttuvaa pysäkkiä käytettävänä vuorolla. |
| Kaksi mahdollista nousupysäkkiä | Myöhäisemmän kotilähdön tai aiemman saapumisen tarjoava pysäkki voittaa hakutilan mukaan, vaikka toinen pysäkki olisi lähempänä. |
| Paluuhaku | Matka alkaa koululta, kävely alkaa jättöpysäkiltä ja aikaraja huomioi odotuksen. |
| Isokronin raja | Negatiivinen kävelybudjetti hylätään. Esimerkkivuoro 07.18 antaa aikaisimmalla kotilähdöllä 07.10 korkeintaan 8 minuuttia kävelyyn ennen nousuvaraa. |
| Kartta ja virhetilat | Aikataulurivi ja merkki toimivat molempiin suuntiin mobiilissa. Reititön haku ja palvelun katkos tuottavat eri ilmoitukset. Kartta ja aikataulut toimivat katkon aikana. |

Luovutukseen kuuluvat lähdekoodi, tuontikomento, generoitu data ja tuontiraportti, testit sekä ohje paikalliseen käynnistykseen ja valittuun hostaukseen. Tee ensin yhden linjan kartta ja yksi todellinen kävelypyyntö. Näytä sen jälkeen linjoilla 53 ja 59, että kaikki haku- ja järjestystestit toimivat ennen laajennusta.

### Arviointipyyntö seuraavalle toteuttajalle

1. Tarkista XLSX-tuonnin rakenne, ilmoita tulkintavirheet ja ehdota tarvittaessa tietomallin muutosta.
2. Nimeä toteutusympäristössä käytettävissä olevat työkalut kartalle, datatuonnille, kävelyreititykselle ja hostaukselle. Listaa tarvittavat tunnukset ja ulkoiset riippuvuudet.
3. Tee tai arvioi pieni koe, jossa aikataulurivi keskittää kartan ja tunnettu kotipiste tuottaa saman vuoron riippumatta siitä, käytetäänkö testireititintä vai oikeaa reititintä vastaavalla kävelyajalla.
4. Ehdota toteutusjärjestys ja kirjaa avoimet päätökset. Näitä ovat ainakin kävelypalvelun valinta, liikennöintipäivät, jakelun yleisö ja isokronien laskentatapa.

## Lähteet

- [ATSEEE Luxembourg I -aikataulut](https://atseee.eu/timetables-european-school-luxembourg-1/) · [linja 53](https://atseee.eu/bus-line-53/) · [linja 59](https://atseee.eu/bus-line-59/) · [linja 60](https://atseee.eu/bus-line-60/) · [linja 63](https://atseee.eu/bus-line-63/) · [alkuperäinen kartta](https://maps.atseee.eu/atseee/mod_mobile/index.php?service=atseee_1)
- [Luxemburgin julkinen GTFS](https://data.public.lu/fr/datasets/horaires-et-arrets-des-transport-publics-gtfs/) · [NeTEx](https://data.public.lu/fr/datasets/horaires-et-arrets-des-transport-publics-netex/)
- [MapLibre GeoJSON](https://maplibre.org/maplibre-gl-js/docs/API/classes/GeoJSONSource/) · [OpenFreeMap](https://openfreemap.org/) · [Leaflet](https://leafletjs.com/reference.html) · [OSM-tiilien käyttöehdot](https://operations.osmfoundation.org/policies/tiles/) · [OpenLayers WMS](https://openlayers.org/en/latest/examples/wms-tiled)
- [openrouteservice](https://api.openrouteservice.org/) · [API-rajat](https://giscience.github.io/openrouteservice/hosted-api-restrictions) · [GraphHopper](https://docs.graphhopper.com/openapi) · [Valhalla](https://valhalla.github.io/valhalla/) · [FOSSGIS OSRM](https://routing.openstreetmap.de/about.html) · [OpenTripPlanner](https://docs.opentripplanner.org/) · [Cloudflare Pages Functions](https://developers.cloudflare.com/pages/functions/bindings/)
