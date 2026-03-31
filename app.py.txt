import os
import requests
from flask import Flask, request, jsonify
import google.generativeai as genai

app = Flask(__name__)

PAGE_ACCESS_TOKEN = os.environ.get("PAGE_ACCESS_TOKEN", "EAAaZCPXFyD5MBRIRVjfoJMLfvLbAhmNyhKOnnOH621zYZAnSJWdhpZBZBSCHZCYeV4BxDwuZAGZBZB4QQ5qMZBeL6CULOa6AKthoEwl2HVlfZBeOzUjbBxyBwJp1JPfzmlJYSVG42S1EkGD4KEWZCrjzyoI8jZAYvAZAFm4K61sMKBy4wru1UBNdoz4JbqZAMnDstkoZCVe3nIAgMCCZCAZDZD")
VERIFY_TOKEN = os.environ.get("VERIFY_TOKEN", "botcontestador")
GEMINI_API_KEY = os.environ.get("GEMINI_API_KEY", "AIzaSyCNJwk6iPd7NCLtLkJ2kAh0Wqmx_DS26jo")

genai.configure(api_key=GEMINI_API_KEY)

def responder_gemini(mensaje):
    try:
        model = genai.GenerativeModel("gemini-1.5-flash")
        prompt = f"Eres un asistente de ventas de electrónicos en Colombia. Responde amable y corto en español. Cliente dice: {mensaje}"
        respuesta = model.generate_content(prompt)
        return respuesta.text
    except Exception as e:
        return "Hola, en este momento no puedo responder. Intenta más tarde."

def enviar_mensaje(recipient_id, texto):
    url = f"https://graph.facebook.com/v19.0/me/messages?access_token={PAGE_ACCESS_TOKEN}"
    payload = {"recipient": {"id": recipient_id}, "message": {"text": texto}}
    requests.post(url, json=payload)

@app.route("/", methods=["GET"])
def home():
    return "Bot activo", 200

@app.route("/webhook", methods=["GET"])
def verificar():
    mode = request.args.get("hub.mode")
    token = request.args.get("hub.verify_token")
    challenge = request.args.get("hub.challenge")
    if mode == "subscribe" and token == VERIFY_TOKEN:
        return challenge, 200
    return "Token inválido", 403

@app.route("/webhook", methods=["POST"])
def recibir():
    data = request.get_json()
    if data.get("object") == "page":
        for entry in data["entry"]:
            for event in entry.get("messaging", []):
                if "message" in event and "text" in event["message"]:
                    sender_id = event["sender"]["id"]
                    texto = event["message"]["text"]
                    respuesta = responder_gemini(texto)
                    enviar_mensaje(sender_id, respuesta)
    return jsonify({"status": "ok"}), 200

if __name__ == "__main__":
    port = int(os.environ.get("PORT", 5000))
    app.run(host="0.0.0.0", port=port, debug=False)
```

---

**2.** Guarda y ejecuta en CMD:
```
cd Desktop/bot-messenger
git add .
git commit -m "fix app"
git push origin main