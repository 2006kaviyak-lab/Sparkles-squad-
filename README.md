import os
import json
import pyttsx3
from gtts import gTTS
from ebooklib import epub
from PyPDF2 import PdfReader
from pydub import AudioSegment
from pydub.playback import play

def extract_epub_content(epub_file):
    book = epub.read_epub(epub_file)
    content = []
    
    # Extract chapters/content from EPUB
    for item in book.get_items():
        if item.get_type() == ebooklib.ITEM_DOCUMENT:
            content.append(item.get_body().decode('utf-8'))
    
    return content

def extract_pdf_content(pdf_file):
    reader = PdfReader(pdf_file)
    content = []
    
    # Extract text from each page of the PDF
    for page in reader.pages:
        content.append(page.extract_text())
    
    return content

def create_abook(content, output_file):
    # Example: Save content to a custom .abook JSON format
    abook_data = {
        "title": "Converted Book Title",  # Could be parsed from metadata
        "author": "Author Name",          # Could be parsed from metadata
        "chapters": content               # List of extracted content
    }
    
    with open(output_file, 'w') as f:
        json.dump(abook_data, f, indent=4)
    
    print(f"Converted book saved as {output_file}")

def text_to_speech(content, output_audio, engine="pyttsx3", speed=150, pitch=50):
    if engine == "pyttsx3":
        # Using pyttsx3 (offline)
        engine = pyttsx3.init()
        engine.setProperty('rate', speed)  # Speed of speech
        engine.setProperty('pitch', pitch)  # Pitch level
        voices = engine.getProperty('voices')
        engine.setProperty('voice', voices[1].id)  # Set voice to a specific one (e.g., female)

        # Save speech to audio file
        engine.save_to_file(content, output_audio)
        engine.runAndWait()
    
    elif engine == "gTTS":
        # Using Google TTS (online)
        tts = gTTS(content, lang='en')
        tts.save(output_audio)
    
    print(f"Audio saved as {output_audio}")

def play_audio(audio_file):
    # Play the saved audio file
    sound = AudioSegment.from_mp3(audio_file)
    play(sound)

def convert_to_abook(input_file, output_file, audio_file, tts_engine="pyttsx3", speed=150, pitch=50):
    if input_file.lower().endswith('.epub'):
        content = extract_epub_content(input_file)
    elif input_file.lower().endswith('.pdf'):
        content = extract_pdf_content(input_file)
    else:
        print("Unsupported file format. Please use EPUB or PDF.")
        return
    
    create_abook(content, output_file)
    
    # Combine text-to-speech functionality
    full_text = " ".join(content)  # Combine all content into one string for TTS
    text_to_speech(full_text, audio_file, engine=tts_engine, speed=speed, pitch=pitch)
    
    # Optionally play the audio
    play_audio(audio_file)

# Example usage
input_file = 'example.epub'  # Replace with your EPUB or PDF file
output_file = 'converted.abook'  # Output file name
audio_file = 'converted_audio.mp3'  # Output audio file name

# Call the conversion function with custom voice and tone options
convert_to_abook(input_file, output_file, audio_file, tts_engine="pyttsx3", speed=175, pitch=75)
