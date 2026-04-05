import speech_recognition as sr
import pyttsx3
import os
from AppOpener import open as open_app
import openai

# 1. SETUP PETER'S BRAIN & VOICE
# Replace with your actual OpenAI API key
openai.api_key = "YOUR_OPENAI_API_KEY" 
engine = pyttsx3.init()

def speak(text):
    print(f"Peter: {text}")
    engine.say(text)
    engine.runAndWait()

# 2. MULTILINGUAL LISTENING
def listen():
    r = sr.Recognizer()
    with sr.Microphone() as source:
        r.adjust_for_ambient_noise(source, duration=1)
        print("Listening...")
        audio = r.listen(source)
    try:
        # language='en-IN' covers English/Hindi; change to your preference
        return r.recognize_google(audio, language='en-IN').lower() 
    except Exception:
        return ""

# 3. PETER'S PERSONALITY (Claude's Knowledge + Grok's Wit)
def peter_ai_response(user_input):
    try:
        response = openai.Completion.create(
            engine="text-davinci-003",  # Use the appropriate engine
            prompt=f"You are Peter, a witty best friend. {user_input}",
            max_tokens=150
        )
        return response.choices[0].text.strip()
    except Exception as e:
        return f"Sorry, I encountered an error: {e}"

# 4. COMMAND LOGIC
if __name__ == "__main__":
    speak("Peter's in the house. What's up?")
    while True:
        query = listen()
        if not query: continue
        
        print(f"You: {query}")

        # COMMAND: Open Apps
        if "open app" in query or "launch" in query:
            app_name = query.replace("open app", "").replace("launch", "").strip()
            open_app(app_name, match_closest=True)
            speak(f"Launching {app_name} for you.")

        # COMMAND: Open Folders/Files (requires full path or common folders)
        elif "open folder" in query or "open file" in query:
            path = query.replace("open folder", "").replace("open file", "").strip()
            try:
                os.startfile(path) # Works on Windows to open any file/folder
                speak(f"Opening {path}.")
            except:
                speak("I couldn't find that path. Make sure it's correct.")

        # COMMAND: Exit
        elif "go to sleep" in query or "exit" in query:
            speak("Catch you later!")
            break
        
        # DEFAULT: Best Friend Chat
        else:
            try:
                response = peter_ai_response(query)
                speak(response)
            except Exception as e:
                speak(f"Sorry, I encountered an error: {e}")
