import streamlit as st
import numpy as np
from scipy.io import wavfile
import io
import re

# Sivun asetukset
st.set_page_config(page_title="GEDCOM Musiikkigeneraattori", layout="centered")

st.title("🎵 GEDCOM-tiedosto musiikiksi")
st.write("""
Tämä sovellus lukee GEDCOM-tiedoston päivämäärät ja muuttaa ne musiikiksi.
**Logiikka:** Tammikuu = C, Helmikuu = D, jne. Jokaista kuukautta soitetaan 
niin monta kertaa peräkkäin kuin se esiintyy tiedostossa.
""")

# Määritellään taajuudet (C4 = Middle C)
# C D E F G A B C D E F G (C Major scale)
NOTE_FREQS = {
    1: 261.63,  # JAN - C4
    2: 293.66,  # FEB - D4
    3: 329.63,  # MAR - E4
    4: 349.23,  # APR - F4
    5: 392.00,  # MAY - G4
    6: 440.00,  # JUN - A4
    7: 493.88,  # JUL - B4
    8: 523.25,  # AUG - C5
    9: 587.33,  # SEP - D5
    10: 659.25, # OCT - E5
    11: 698.46, # NOV - F5
    12: 783.99  # DEC - G5
}

MONTH_MAP = {
    'JAN': 1, 'FEB': 2, 'MAR': 3, 'APR': 4, 'MAY': 5, 'JUN': 6,
    'JUL': 7, 'AUG': 8, 'SEP': 9, 'OCT': 10, 'NOV': 11, 'DEC': 12
}

def parse_gedcom_months(content):
    """Etsii kaikki DATE-rivit ja palauttaa löydetyt kuukaudet listana."""
    months_found = []
    # Gedcom päivämäärät ovat muotoa: 2 DATE 14 JAN 1900
    # Regex etsii kuukauden lyhenteitä
    pattern = re.compile(r'\b(JAN|FEB|MAR|APR|MAY|JUN|JUL|AUG|SEP|OCT|NOV|DEC)\b', re.IGNORECASE)
    
    for line in content.splitlines():
        if "DATE" in line:
            match = pattern.search(line)
            if match:
                month_str = match.group(1).upper()
                months_found.append(MONTH_MAP[month_str])
    return months_found

def generate_sine_wave(freq, duration, sample_rate=44100, amplitude=0.5):
    """Luo siniaallon annetulla taajuudella ja kestolla."""
    t = np.linspace(0, duration, int(sample_rate * duration), False)
    wave = amplitude * np.sin(2 * np.pi * freq * t)
    
    # Lisätään pieni "envelope" (fade in/out) naksahdusten estämiseksi
    envelope_len = int(sample_rate * 0.01) # 10ms
    if len(wave) > 2 * envelope_len:
        wave[:envelope_len] *= np.linspace(0, 1, envelope_len)
        wave[-envelope_len:] *= np.linspace(1, 0, envelope_len)
    
    return wave

# Käyttöliittymä
uploaded_file = st.file_uploader("Lataa GEDCOM-tiedosto (.ged)", type=['ged'])

# Asetukset
col1, col2 = st.columns(2)
with col1:
    note_duration = st.slider("Nuotin kesto (sekuntia)", 0.05, 0.5, 0.1, 0.05)
with col2:
    volume = st.slider("Äänenvoimakkuus", 0.1, 1.0, 0.5)

if uploaded_file is not None:
    # Luetaan tiedosto
    content = uploaded_file.getvalue().decode("utf-8", errors='ignore')
    
    with st.spinner('Analysoidaan sukupuuta...'):
        months = parse_gedcom_months(content)
        
        # Lasketaan kuukausien määrät (histogrammi)
        month_counts = {i: 0 for i in range(1, 13)}
        for m in months:
            month_counts[m] += 1
            
    st.success(f"Löydetty yhteensä {len(months)} päivämäärää!")
    
    # Näytetään tilastot
    st.bar_chart(month_counts)
    
    with st.spinner('Generoidaan musiikkia...'):
        audio_parts = []
        sample_rate = 44100
        
        # Luodaan ääni: Käydään läpi kuukaudet 1-12
        # Soitetaan jokaista kuukautta niin monta kertaa kuin se löytyi
        for m in range(1, 13):
            count = month_counts[m]
            freq = NOTE_FREQS[m]
            
            if count > 0:
                # Luodaan yksi "piippaus" ja toistetaan se count kertaa
                # Huom: Teemme pienen tauon nuottien väliin (silence)
                tone = generate_sine_wave(freq, note_duration, sample_rate, volume)
                silence = np.zeros(int(sample_rate * 0.05)) # 50ms tauko
                
                # Yhdistetään nuotti + tauko yhdeksi paketiksi
                note_sequence = np.concatenate([tone, silence])
                
                # Toistetaan pakettia 'count' kertaa
                month_sequence = np.tile(note_sequence, count)
                audio_parts.append(month_sequence)
        
        if audio_parts:
            full_audio = np.concatenate(audio_parts)
            
            # Normalisointi ja muunnos 16-bit PCM muotoon
            full_audio = full_audio / np.max(np.abs(full_audio)) * 32767
            full_audio = full_audio.astype(np.int16)
            
            # Kirjoitetaan virtuaaliseen tiedostoon
            wav_buffer = io.BytesIO()
            wavfile.write(wav_buffer, sample_rate, full_audio)
            
            # Soitin
            st.audio(wav_buffer, format='audio/wav')
            
            st.write("### Selite:")
            st.write("Matalimmat äänet ovat alkuvuodesta (Tammikuu = C), korkeimmat loppuvuodesta (Joulukuu = G).")
            st.write("Mitä pidempään sama sävel soi, sitä enemmän tapahtumia kyseisessä kuussa on.")
        else:
            st.warning("Tiedostosta ei löytynyt tunnistettavia päivämääriä.")
