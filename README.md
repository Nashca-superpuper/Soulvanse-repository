import requests
from telegram import Update
from telegram.ext import (
    ApplicationBuilder,
    CommandHandler,
    MessageHandler,
    filters,
    ContextTypes,
    ConversationHandler,
)

# Стадії розмови
ASK_LOGIN, ASK_NAME = range(2)

# Встав реальний вебхук з Make
MAKE_WEBHOOK_URL = "(https://hook.eu2.make.com/63hdviyi92vprlj4ytl4up54cl2emszh)"

# Початок — вітаємо і питаємо логін
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text(
        "👋 Welcome to Soulvance!\nPlease enter your login (Telegram @username or email):"
    )
    return ASK_LOGIN

# Отримуємо логін, питаємо ім’я
async def ask_login(update: Update, context: ContextTypes.DEFAULT_TYPE):
    context.user_data["login"] = update.message.text
    await update.message.reply_text("Great! Now enter your name:")
    return ASK_NAME

# Отримуємо ім’я, надсилаємо у Make
async def ask_name(update: Update, context: ContextTypes.DEFAULT_TYPE):
    context.user_data["name"] = update.message.text
    user_id = update.effective_user.id

    # Готуємо і надсилаємо дані у Make
    data = {
        "user_id": user_id,
        "login": context.user_data["login"],
        "name": context.user_data["name"]
    }

    requests.post(MAKE_WEBHOOK_URL, json=data)

    await update.message.reply_text("✅ You’ve been registered. Let the battle begin!")
    return ConversationHandler.END

# Якщо користувач скасує процес
async def cancel(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text("❌ Registration canceled.")
    return ConversationHandler.END

# Основна логіка запуску бота
app = ApplicationBuilder().token("7565851419:AAFfQ5nLqjiY5vVM8r8izGazGqWw2g-6k9g").build()

# Додаємо обробник розмови
conv_handler = ConversationHandler(
    entry_points=[CommandHandler("start", start)],
    states={
        ASK_LOGIN: [MessageHandler(filters.TEXT & ~filters.COMMAND, ask_login)],
        ASK_NAME: [MessageHandler(filters.TEXT & ~filters.COMMAND, ask_name)],
    },
    fallbacks=[CommandHandler("cancel", cancel)],
)

app.add_handler(conv_handler)

# Запуск
app.run_polling()
