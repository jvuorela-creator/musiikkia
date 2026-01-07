# GEDCOM Musiikkigeneraattori

Tämä on Streamlit-sovellus, joka muuttaa sukututkimusdatan (GEDCOM) ääneksi.

## Toimintaperiaate
1. Ohjelma lukee GEDCOM-tiedoston ja etsii `DATE`-kentät.
2. Se laskee, kuinka monta tapahtumaa osuu kullekin kuukaudelle.
3. Kuukaudet muutetaan säveliksi:
   - Tammikuu: C4
   - Helmikuu: D4
   - ...
   - Joulukuu: G5
4. Ohjelma soittaa kutakin säveltä peräkkäin niin monta kertaa kuin kyseinen kuukausi esiintyy aineistossa.

## Käyttöohjeet
1. Lataa `.ged` tiedosto.
2. Säädä tarvittaessa nuotin kestoa (tempoa).
3. Kuuntele tulos soittimesta.

## Asennus (paikallisesti)
```bash
pip install -r requirements.txt
streamlit run streamlit_app.py
