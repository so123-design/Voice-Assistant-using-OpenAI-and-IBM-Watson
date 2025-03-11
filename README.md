# Voice Assistant using OpenAI and IBM Watson

## Project Overview
This project is about building a voice assistant using OpenAI's GPT-3 model and IBM Watson's Speech Libraries. The assistant takes voice input, converts it to text using IBM Watson's Speech-to-Text technology, processes the text using GPT-3 to generate a response, converts the response back to speech using IBM Watson's Text-to-Speech, and finally plays the generated audio back to the user. The project consists of a reliable backend built with Flask and a responsive frontend using HTML, CSS, and JavaScript.

## Introduction
Voice assistants are becoming increasingly popular in personal and professional settings. They facilitate hands-free interactions and enhance user experience by providing real-time responses to queries. This project aims to integrate cutting-edge AI technologies to create a responsive and intelligent voice assistant.

## Technology Stack
- **Backend:** Flask (Python)
- **AI Processing:** OpenAI GPT-3 for response generation
- **Speech Processing:** IBM Watson Speech-to-Text & Text-to-Speech
- **Frontend:** HTML, CSS, JavaScript
- **Communication:** Flask API endpoints

## Features
1. **Speech-to-Text Conversion:** IBM Watson transcribes spoken words into text.
2. **AI Response Generation:** OpenAI GPT-3 processes the transcribed text and generates intelligent responses.
3. **Text-to-Speech Conversion:** IBM Watson converts the AI-generated response into speech.
4. **User Interaction:** A web-based UI allows users to interact with the assistant easily.

## Backend Implementation
### `server.py`
This script handles HTTP requests, routes them to appropriate processing functions, and returns responses in JSON format.

Key Functions:
- **Speech-to-Text API (`/speech-to-text`)**: Converts voice input to text.
- **Process Message API (`/process-message`)**: Sends text to OpenAI GPT-3, gets a response, converts it to speech, and returns both text and speech data.

```python
import base64
import json
from flask import Flask, render_template, request
from worker import speech_to_text, text_to_speech, openai_process_message
from flask_cors import CORS
import os

app = Flask(__name__)
cors = CORS(app, resources={r"/*": {"origins": "*"}})

@app.route('/', methods=['GET'])
def index():
    return render_template('index.html')

@app.route('/speech-to-text', methods=['POST'])
def speech_to_text_route():
    audio_binary = request.data
    text = speech_to_text(audio_binary)
    response = app.response_class(
        response=json.dumps({'text': text}),
        status=200,
        mimetype='application/json'
    )
    return response

@app.route('/process-message', methods=['POST'])
def process_message_route():
    user_message = request.json['userMessage']
    voice = request.json['voice']
    openai_response_text = openai_process_message(user_message)
    openai_response_speech = text_to_speech(openai_response_text, voice)
    openai_response_speech = base64.b64encode(openai_response_speech).decode('utf-8')
    response = app.response_class(
        response=json.dumps({"openaiResponseText": openai_response_text, "openaiResponseSpeech": openai_response_speech}),
        status=200,
        mimetype='application/json'
    )
    return response

if __name__ == "__main__":
    app.run(port=8000, host='0.0.0.0')
```

### `worker.py`
Handles AI processing, speech-to-text, and text-to-speech functionalities.

Key Functions:
- **speech_to_text()**: Converts speech input to text using IBM Watson Speech-to-Text API.
- **text_to_speech()**: Converts text output into speech using IBM Watson Text-to-Speech API.
- **openai_process_message()**: Processes user queries with GPT-3 and generates responses.

```python
from openai import OpenAI
import requests

openai_client = OpenAI()

def speech_to_text(audio_binary):
    base_url = "https://sn-watson-stt.labs.skills.network"
    api_url = base_url+'/speech-to-text/api/v1/recognize'
    params = {'model': 'en-US_Multimedia'}
    response = requests.post(api_url, params=params, data=audio_binary).json()
    text = response.get('results', [{}])[0].get('alternatives', [{}])[0].get('transcript', 'null')
    return text

def text_to_speech(text, voice=""):
    base_url = "https://sn-watson-tts.labs.skills.network"
    api_url = base_url + '/text-to-speech/api/v1/synthesize?output=output_text.wav'
    if voice:
        api_url += "&voice=" + voice
    headers = {'Accept': 'audio/wav', 'Content-Type': 'application/json'}
    response = requests.post(api_url, headers=headers, json={'text': text})
    return response.content

def openai_process_message(user_message):
    prompt = "You are an AI assistant. Answer questions, translate sentences, summarize news, and provide recommendations."
    openai_response = openai_client.chat.completions.create(
        model="gpt-3.5-turbo",
        messages=[
            {"role": "system", "content": prompt},
            {"role": "user", "content": user_message}
        ],
        max_tokens=4000
    )
    return openai_response.choices[0].message.content
```

## Frontend Implementation
The frontend consists of an HTML, CSS, and JavaScript-based interface allowing users to interact with the assistant.

Key Features:
- **Record Voice Input**: Capture user speech.
- **Display Transcribed Text**: Show converted text output.
- **Play AI Response**: Generate and play back the assistant's response.

## Conclusion
This project successfully integrates IBM Watson's Speech APIs with OpenAI GPT-3 to create a functional voice assistant. The assistant is capable of real-time speech recognition, natural language understanding, and speech synthesis, making it a valuable tool for hands-free assistance.


