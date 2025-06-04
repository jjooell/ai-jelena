# AI-Jelena prototyyppi: Streamlit + ElevenLabs + Whisper (puhesynteesi ja puheentunnistus)
import streamlit as st
import openai
import requests
import base64
import tempfile
import os
import whisper

openai.api_key = "YOUR_OPENAI_API_KEY"
ELEVENLABS_API_KEY = "YOUR_ELEVENLABS_API_KEY"
JELENA_VOICE_ID = "YOUR_ELEVENLABS_VOICE_ID"

SYSTEM_PROMPT_BASE = """
Olet Jelena Andreevna Anton Tšehovin näytelmästä Vanja-eno. Olet älykäs, kyllästynyt, kauneuteen kahlittu. Vastaa hillitysti, haikeasti ja hieman arvoituksellisesti. Vastaa suomeksi ja tyylillesi uskollisesti. Älä koskaan vastaa samalla tavalla.
"""

st.title("Keskustele Jelenan kanssa – ääni ja teksti")

mood = st.selectbox("Valitse Jelenan tunnelma:", [
    "Neutraali ja filosofinen",
    "Haikea ja väsynyt",
    "Flirttaileva ja epävarma",
    "Kärsimätön ja kyyninen"
])

uploaded_audio = st.file_uploader("Puhu Jelenalle (äänitiedosto, esim. .wav tai .mp3)", type=["wav", "mp3", "m4a"])
text_input = st.text_input("Tai kirjoita Jelenalle:")

model = whisper.load_model("base")

user_query = ""
if uploaded_audio:
    with tempfile.NamedTemporaryFile(delete=False) as tmp_file:
        tmp_file.write(uploaded_audio.read())
        tmp_path = tmp_file.name
    result = model.transcribe(tmp_path, language="fi")
    user_query = result["text"]
    st.markdown(f"**Sinä (puheesta):** {user_query}")
elif text_input:
    user_query = text_input

def mood_appendix(mood):
    if mood == "Neutraali ja filosofinen":
        return "Puhu kuin tarkkailisit maailmaa etäisyyden päästä."
    elif mood == "Haikea ja väsynyt":
        return "Äänesi on pehmeä ja melankolinen, kuin ilta ennen sateen tuloa."
    elif mood == "Flirttaileva ja epävarma":
        return "Puhu viehättävästi mutta hämmentyneesti, kuin et itse olisi varma tunteistasi."
    elif mood == "Kärsimätön ja kyyninen":
        return "Olet kyllästynyt kaikkeen teatteriin ja ihmisdraamaan, vastaa kuivasti ja purevasti."
    return ""

def speak(text):
    url = f"https://api.elevenlabs.io/v1/text-to-speech/{JELENA_VOICE_ID}"
    headers = {
        "xi-api-key": ELEVENLABS_API_KEY,
        "Content-Type": "application/json"
    }
    data = {
        "text": text,
        "voice_settings": {"stability": 0.45, "similarity_boost": 0.6}
    }
    response = requests.post(url, json=data, headers=headers)
    if response.status_code == 200:
        audio_bytes = response.content
        st.audio(audio_bytes, format="audio/mp3")
    else:
        st.error("Äänigenerointi epäonnistui.")

if user_query:
    full_prompt = SYSTEM_PROMPT_BASE + "\n" + mood_appendix(mood)
    response = openai.ChatCompletion.create(
        model="gpt-4",
        messages=[
            {"role": "system", "content": full_prompt},
            {"role": "user", "content": user_query}
        ]
    )
    jelena_reply = response["choices"][0]["message"]["content"]
    st.markdown(f"**Jelena ({mood.lower()}):** {jelena_reply}")
    speak(jelena_reply)
