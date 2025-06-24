import requests
from telegram import Update
from telegram.ext import ApplicationBuilder, CommandHandler, ContextTypes

# Заміни на свій Webhook URL із Make
MAKE_WEBHOOK_URL = "[https://hook.eu.make.com/your-make-hook-id](https://hook.eu2.make.com/63hdviyi92vprlj4ytl4up54cl2emszh)"

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.effective_user.id
    username = update.effective_user.username

    # Надсилаємо POST-запит у Make
    requests.post(MAKE_WEBHOOK_URL, json={"user_id": user_id, "username": username})

    await update.message.reply_text("👋 Welcome to Soulvance! You've been registered.")

app = ApplicationBuilder().token("7565851419:AAFfQ5nLqjiY5vVM8r8izGazGqWw2g-6k9g").build()
app.add_handler(CommandHandler("start", start))
app.run_polling()
