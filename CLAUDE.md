# CLAUDE.md — ATSEEE-koulubussikartta

Julkinen verkkosivu Eurooppa-koulu I:n (Kirchberg, EE1) ATSEEE-koulubusseille: linjakartta, aikataulut ja kodista kouluun -reittihaku. Myöhemmin saavutettavuusalueet (isokronit) ja asuntohaun rajapinta.

Täysi määrittely: `docs/speksi.md`. Tämä tiedosto kertoo, mitä on päätetty speksin jälkeen. Ristiriidassa tämä tiedosto voittaa.

## Päätökset

- **Julkaisu netissä alusta asti.** Cloudflare Pages (ilmaistaso), kytketty tähän repoon. Jokainen PR saa esikatselulinkin, merge `main`iin julkaisee.
- **Staattinen sivu.** Ei tietokantaa, ei build-frameworkia ellei välttämätön. Selain laskee aikataulut paikallisesti `web/data/data.json`:sta.
- **Kartta:** MapLibre GL JS + OpenFreeMap (avaimeton). Näytä attribuutio ja dataversio.
- **Kävelyaika kahdessa tasossa:**
  1. Oletus ja varareitti: linnuntie × 1,3 kiertokerroin, 4,5 km/h. Toimii ilman verkkopalvelua ja toimii testireitittimenä.
  2. Tarkennus: openrouteservice (foot-walking) Cloudflare Pages Functionin `/api/walk` kautta. API-avain on Cloudflaren salaisuus `ORS_API_KEY`, **ei koskaan repossa**. Funktio rajaa koordinaatit Luxemburgin + rajaseudun bboxiin, pyyntökoon ja aikakatkaisun.
- **Ilmaistason rajat:** ORS Standard ≈ 2 000 directions ja 500 matrix-pyyntöä vuorokaudessa. Yksi reittihaku = korkeintaan yksi matrix-pyyntö + reittiviivat vain näytettäville vaihtoehdoille.
- **Isokronit lasketaan rakennusvaiheessa** skriptillä valmiiksi GeoJSON-tasoiksi (aikataulu ei muutu kesken lukuvuoden). Ei isokronipyyntöjä ajon aikana.
- **Ei liikennöintikalenteria.** Bussit ajavat koulupäivinä samoilla aikatauluilla. Käyttäjä valitsee aamu / lounas / ilta. `Service`-tietue jätetään pois.
- **Saksan puoli:** linjoilla 63, 64 ja 67 on pysäkkejä Saksassa. Hostattu ORS kattaa ne, ei erillistä OSM-otetta.
- **Käyttäjämäärä aluksi pieni**, ei mainosta. Skaalaus ratkaistaan jos tarvitaan.
- **Kieli:** käyttöliittymä aluksi suomeksi, rakenne valmis englannille/ranskalle. Koodi, tunnisteet ja commit-viestit englanniksi.

## Lähdeaineisto

`data/source/atseee-aikataulut.xlsx`, välilehti `aikataulut`. Tarkistettu 16.11.2026:

- 20 lohkoa sarakkeessa A: 19 linjaa + `N CPE KI` (otsikko ilman pysäkkirivejä, ohitetaan ja raportoidaan).
- 225 pysäkkiriviä, jokaisen lohkon viimeinen rivi on `KIRCHBERG, EE1`.
- Lohkon 2. rivi on otsikkorivi (`Bus stop`, `Lat (WGS84)`, …, `Morning`, `Lunch`, `Evening`).
- K–M: `datetime.time` tai `x` (vuoro ohittaa pysäkin). Tyhjä = tieto puuttuu → tuontihuomio.
- Linjaotsikoissa on alussa NBSP (`\xa0`) → normalisoi.
- Sarakkeet AJ–AK sisältävät erillisen linjaluettelon → ohita.
- Solu J1 sisältää attribuutiotekstin (OSM, GTFS, Géoportail) → näytä käyttöliittymässä.
- GTFS-tunnukset merkkijonoina etunollineen.
- Muut välilehdet (`kartta`, `Reitit`) eivät ole tuonnin lähde.

## Rakenne

```
data/source/        lähde-Excel (syöte, muokataan vain kun ATSEEE päivittää)
scripts/            import_xlsx.py ym. (Python 3, openpyxl)
web/                staattinen sivu, julkaistaan Cloudflare Pagesiin
web/data/data.json  generoitu, commitoidaan
functions/api/      Cloudflare Pages Functions (walk)
tests/              pytest (tuonti) + Vitest tai node:test (reittilaskenta)
docs/               speksi ja tuontiraportti
```

## Työtapa

- Tuonti: `python scripts/import_xlsx.py` → `web/data/data.json` + `docs/import-report.md`. Aja uudelleen aina kun Excel päivittyy ja commitoi molemmat.
- CI (GitHub Actions): testit + tarkistus että `data.json` vastaa Exceliä.
- Hyväksymistestit speksin luvusta 7 kirjataan testeiksi (esim. linja 53 Mambra: lähtö 07.18, EE1 08.10, kävely 6 min → kotoa 07.12, 58 min).
- Pienet, tarkistettavat PR:t. PR-kuvaukseen lyhyt suomenkielinen yhteenveto ja miten testata esikatselulinkillä (myös mobiilissa).
- Älä lisää riippuvuuksia ilman syytä. Ei seurantaskriptejä, ei evästeitä. Kotipistettä ei tallenneta palvelimelle; se välitetään vain kävelyreititykseen.

## Toteutusjärjestys

1. Tuonti + tuontiraportti kaikille 19 linjalle
2. Linjakartta: linjavalinta, pysäkkihaku (aksentiton), aikataulu ↔ kartta molempiin suuntiin, mobiilissa alapaneeli
3. Reittihaku arviokävelyllä + hyväksymistestit linjoilla 53 ja 59
4. ORS-kävely Pages Functionin kautta, eri virheilmoitus reitittömälle ja palvelukatkolle
5. Valmiiksi lasketut isokronit + asuntohaun rajapinta
