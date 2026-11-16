
import os
import threading
import requests

from kivy.app import App
from kivy.clock import Clock
from kivy.lang import Builder
from kivy.uix.boxlayout import BoxLayout
from kivy.uix.label import Label

KV = """
BoxLayout:
    orientation: "vertical"
    padding: 15
    spacing: 12

    canvas.before:
        Color:
            rgba: 0.015, 0.04, 0.10, 1
        Rectangle:
            pos: self.pos
            size: self.size

    Label:
        text: "[b]J.A.R.V.I.S[/b]"
        markup: True
        font_size: "32sp"
        color: 0.2, 0.8, 1, 1
        size_hint_y: 0.14

    Label:
        text: "PERSONAL AI ASSISTANT"
        color: 0.3, 0.8, 1, 1
        size_hint_y: 0.06

    TextInput:
        id: api
        hint_text: "Enter your Groq API key"
        password: True
        multiline: False
        size_hint_y: 0.09

    Label:
        id: output
        text: "Hello! I am JARVIS. How can I help you?"
        color: 0.8, 0.95, 1, 1
        text_size: self.width, None
        halign: "left"
        valign: "top"

    TextInput:
        id: message
        hint_text: "Talk to JARVIS..."
        multiline: True
        size_hint_y: 0.18

    Button:
        text: "SEND TO JARVIS"
        size_hint_y: 0.12
        background_color: 0.0, 0.45, 0.8, 1
        on_release: app.ask_jarvis()

    Label:
        text: "JARVIS AI SYSTEM"
        color: 0.2, 0.7, 1, 1
        size_hint_y: 0.07
"""

class JarvisApp(App):

    def build(self):
        self.root = Builder.load_string(KV)
        return self.root

    def ask_jarvis(self):
        api_key = self.root.ids.api.text.strip()
        question = self.root.ids.message.text.strip()

        if not api_key or not question:
            self.root.ids.output.text = "Enter API key and message."
            return

        self.root.ids.message.text = ""
        self.root.ids.output.text = "JARVIS is thinking..."

        threading.Thread(
            target=self.get_answer,
            args=(api_key, question),
            daemon=True
        ).start()

    def get_answer(self, api_key, question):
        try:
            response = requests.post(
                "https://api.groq.com/openai/v1/chat/completions",
                headers={
                    "Authorization": f"Bearer {api_key}",
                    "Content-Type": "application/json"
                },
                json={
                    "model": "llama-3.3-70b-versatile",
                    "messages": [
                        {
                            "role": "system",
                            "content": "You are JARVIS, a helpful personal AI assistant. Reply naturally in the language used by the user."
                        },
                        {
                            "role": "user",
                            "content": question
                        }
                    ],
                    "temperature": 0.7
                },
                timeout=60
            )

            response.raise_for_status()
            answer = response.json()["choices"][0]["message"]["content"]

        except Exception as e:
            answer = "Connection error: " + str(e)

        Clock.schedule_once(
            lambda dt: self.show_answer(answer)
        )

    def show_answer(self, answer):
        self.root.ids.output.text = answer


if __name__ == "__main__":
    JarvisApp().run()
