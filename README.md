## Team and my contribution

A 2-person project with Bo Zhou. I wrote the chatbot code (the assistant logic, the review system, the order reminders and the WhatsApp integration). My teammate built the knowledge-base content.

---

# LCB 2 CSC Team 3 - Brabants Streekgoed chatbot

This was a group project for our LCB module. We built a customer service chatbot
for Brabants Streekgoed, which is a cooperative of local farmers from Brabant
that sells directly to customers. The problem they have is that a lot of the
same questions come in over and over (which products, how delivery works, where
the pickup points are, when the order deadline is), and a small shop cannot sit
on WhatsApp all day answering them. The bot handles those questions in Dutch or
English, and it only answers from the shop's real information so it does not give
people wrong delivery costs or made-up products.

It runs in a Jupyter notebook and over WhatsApp.

## What it does

- Answers in Dutch or English. It reads the language from the user message and
  replies in the same one. For the model answers I add a short instruction at
  the end of the message telling it to keep the same language as the question.
- Only answers from the knowledge base. The whole shop information (products,
  pickup points, order days, delivery cost, FAQ) is put into the system prompt,
  and the prompt tells the model not to invent anything that is not in there. If
  it does not know, it points the customer to the shop WhatsApp number 0641088180.
- Takes a review. The customer types `review`, the bot asks for a score from 1
  to 5, then asks for a comment, and saves it to a JSON file with a timestamp. I
  also wrote helpers for the average score and the most recent reviews.
- Sets an order reminder. If the message has a reminder word (in Dutch or
  English) the bot reads the day and time out of the text and confirms it. Only
  Monday and Tuesday make sense because the order deadline is Tuesday 23:59, so
  if no day is given it defaults to Tuesday and if no time is given it uses
  18:00, just before the deadline.
- Runs over WhatsApp through Green-API. It polls for new messages, answers them
  and sends the reply back, keeping a separate conversation history per user.

## How it works (my approach)

The main entry point is the `chat()` function. The thing I had to think about
most was that not every message should go to the model. A review or a reminder I
want to handle myself in code so the answers are exact and cheap, and only the
real questions go to the model. So `chat()` checks in order:

1. Is this part of a review? `handle_review_command()` runs a small state machine
   per chat: idle, then waiting for a rating, then waiting for a comment. I keep
   the state in a dict keyed by `chat_id` so several WhatsApp users can be in a
   review at the same time without mixing up. When the rating comes in I scan the
   message for the first digit 1 to 5, and when the comment comes in I save it.
   The closing line changes with the score (a happy one for 4 to 5, a "we will do
   better" one for low scores).
2. Is this a reminder? `handle_reminder_logic()` looks for the keywords
   (`remind me`, `herinner mij`, `reminder`, `herinnering`), then tries to read a
   day and a time. For the time I used a regex that needs a clock-like token, so
   "10:00", "14:30", "10 uur" or "2 pm" are read as a time but a plain number
   like "5 stars" is not. It also handles am/pm. Then it works out the next
   Monday or Tuesday from today's date.
3. If it is neither, the message goes to the model with the system prompt and the
   conversation history.

I made some decisions on purpose because this is a school project:

- I used a JSON file for the reviews instead of a database. For the amount of
  reviews a school project gets, a file is enough and there is nothing to set up.
- I put the full knowledge base text straight into the system prompt instead of
  building a retrieval / vector setup. The shop information fits in one short
  text file (about 6.5 KB), so the model can just read all of it every time. The
  prompt is strict about only using what is in the knowledge base.
- I read the server URL and the Green-API keys from environment variables, not
  hardcoded, so I do not put the BUas server address or the API tokens in the
  notebook.

The WhatsApp loop (`handle_whatsapp_bot()`) talks to Green-API with plain
`requests`. It pulls the next notification, pulls the text out of the webhook
body, runs it through `chat()`, posts the reply back, and then deletes the
notification from the queue so the same message is not answered twice. It is
wrapped in a try/except with a short sleep so one bad message does not kill the
loop.

One honest note: in the notebook there are two reminder paths. The full
`handle_reminder_logic()` with day and time parsing is used in the interactive
chat, and the WhatsApp loop uses a simpler "remind/herinner" check that replies
with a fixed line. There is also a `send_whatsapp_message()` that just prints,
which I used as a placeholder so I could test the reminder logic without a live
number.

## Tech

- Model: `qwen3:14b`, self-hosted on the BUas Ollama server
- Python 3.11 in a Jupyter notebook
- `ollama` Python client to talk to the model
- `requests` for the Green-API WhatsApp calls
- standard library: `json`, `re`, `os`, `datetime`, `time`
- Green-API for the WhatsApp connection

## How to run

You need two things that are not in this repo: access to the BUas Ollama server
(so you must be on the BUas network or VPN) and a Green-API instance if you want
the WhatsApp part. Without the VPN the model calls will fail.

1. Connect to the BUas network or VPN.
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
   bottom. The first cell checks the Ollama connection. You can chat in the
   notebook with the interactive cell. The last cell starts the WhatsApp loop,
   you only need that one if you have the Green-API credentials set.

## Result

It answers the test questions correctly in both Dutch and English and stays on
the knowledge base. I tested it with questions like "How can I place an order?",
"What are the delivery costs?", "Until when can I change my order?", "Are your
products organic?" and "What is the minimum order amount?", and it gave the right
answers from the shop information (for example the 6,95 euro delivery cost and
the Tuesday 23:59 deadline). The review flow saves to JSON correctly and the
average comes out right. The WhatsApp part worked with our Green-API test
instance.

Limitations I am aware of: there is no automatic test, I checked it by hand. The
WhatsApp reminder path is simpler than the notebook one. And the bot is only as
good as the knowledge base text, if something is not written in there it will say
it does not know.

## Files

- `LCB_2_CSC 1 (chat_bot)new_one.ipynb` - all the code (my part)
- `Knowledge_Base/knowledge_compact.txt` - the shop information (teammate's part)
- `.env.example` - the environment variables to copy to `.env`
- `requirements.txt` - the two main packages
- `Gebruikershandleiding_Chatbot.docx`, `User_Manual_Chatbot.docx`,
  `Chatbot (2).pdf` - the documents, a team effort
- `Reviews/reviews.json` - created at runtime when the first review is saved.
  This file and `.env` are gitignored.
