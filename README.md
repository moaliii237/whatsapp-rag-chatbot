## Team and my contribution

A 2-person project with Bo Zhou. I wrote the chatbot code (the assistant logic, the review system, the order reminders and the WhatsApp integration). My teammate built the knowledge-base content.

---

# LCB 2 CSC Team 3 - Brabants Streekgoed chatbot

This was a group project for our LCB module. We built a customer service chatbot
for Brabants Streekgoed, which is a group of local farmers from Brabant that
sells directly to customers. The bot answers the common questions people ask
(products, delivery, pickup points, order days) and it works in a Jupyter
notebook and over WhatsApp.

## What it does

- Answers in English or Dutch. It picks the language from the user message and
  replies in the same one.
- Talks over WhatsApp through Green-API.
- Can set an order reminder. The customer can ask to be reminded on Monday or
  Tuesday (the order days) and give a time. If they do not give one it uses
  Tuesday 18:00, just before the deadline.
- Can take a review: it asks for a score from 1 to 5 and a comment and saves it
  to a JSON file.
- Only answers from the knowledge base, which is a text file with the shop
  information I copied from their website. It is told not to make things up.

## My part

I worked on the Python side: the chat function, the review flow, the reminder
logic and connecting it to the BUas Ollama server and to Green-API. The
knowledge base text and the documents (user manual, PDF) were a team effort.

## Tech

- Model: `qwen3:14b` on the BUas Ollama server
- Python 3.11, in a Jupyter notebook
- Green-API for WhatsApp

## How to run

1. Connect to the BUas network or VPN, otherwise the Ollama server is not
   reachable.
2. Install the packages:
   ```bash
   pip install requests ollama
   ```
3. Copy `.env.example` to `.env` and fill in the values (or set the same
   environment variables yourself):
   - `OLLAMA_HOST` - the BUas Ollama server URL
   - `GREEN_API_HOST`, `GREEN_API_ID_INSTANCE`, `GREEN_API_TOKEN_INSTANCE` -
     the Green-API credentials for the WhatsApp number
4. Open `LCB_2_CSC 1 (chat_bot)new_one.ipynb` and run the cells from top to
   bottom. The last cell starts the WhatsApp loop, you only need that if you
   have the Green-API credentials set.

## Result

It answers the test questions correctly in both languages and stays on the
knowledge base. The review and reminder flows work in the notebook. The
WhatsApp part worked with our Green-API test instance.

## Notes

- Reviews are written to `Reviews/reviews.json` in the project folder. That
  file and the `.env` file are gitignored.
