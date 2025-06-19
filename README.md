# horror-telegram-bot.
bot.py

import os, random, requests
from telegram import Update, InlineKeyboardButton, InlineKeyboardMarkup
from telegram.ext import ApplicationBuilder, CommandHandler, MessageHandler, CallbackQueryHandler, filters, ContextTypes

scary = [
    "Она всё ещё смотрит на тебя...", "Он идёт по ту сторону экрана.",
    "Ты слышал это?", "Кто-то за твоей спиной.", "Оно видит тебя."
]
tpl = {
    "under_bed": "ребёнок под кроватью",
    "mirror": "демон в зеркале",
    "tv": "лицо из телевизора",
    "forest": "один в лесу",
    "shower": "силуэт в душе",
    "corridor": "пустой коридор"
}

def gen(prompt):
    url, key = os.getenv("VIDEO_API_URL"), os.getenv("VIDEO_API_KEY")
    if not url or not key:
        return "https://example.com/fake_horror_video.mp4"
    r = requests.post(url, headers={"Authorization":f"Bearer {key}"}, json={"prompt":prompt})
    return r.json().get("url")

async def start(update: Update, ctx: ContextTypes.DEFAULT_TYPE):
    kb = [[InlineKeyboardButton("Под кроватью 👁", "under_bed")],
          [InlineKeyboardButton("В зеркале 🪞", "mirror")],
          [InlineKeyboardButton("Из телевизора 📺", "tv")],
          [InlineKeyboardButton("В лесу 🌲", "forest")],
          [InlineKeyboardButton("В душе 🚿", "shower")],
          [InlineKeyboardButton("В коридоре 👣", "corridor")]]
    await update.message.reply_text("👁 Выбери сцену или напиши свою:", reply_markup=InlineKeyboardMarkup(kb))

async def tmpl(update, ctx):
    q = update.callback_query
    await q.answer()
    prompt = tpl[q.data]
    await q.message.reply_text(f"Генерирую: *{prompt}*...", parse_mode="Markdown")
    url = gen(prompt); phrase = random.choice(scary)
    await q.message.reply_video(video=url, caption=f"💀 {phrase}")

async def prmpt(update, ctx):
    prompt = update.message.text
    await update.message.reply_text("🧠 Думаю над ужасом...")
    url = gen(prompt); phrase = random.choice(scary)
    await update.message.reply_video(video=url, caption=f"🎥 {prompt}\n\n💀 {phrase}")

async def scream(update, ctx):
    await update.message.reply_text("Boo! 👻")

if __name__ == "__main__":
    app = ApplicationBuilder().token(os.getenv("TELEGRAM_BOT_TOKEN")).build()
    app.add_handler(CommandHandler("start", start))
    app.add_handler(CommandHandler("scream", scream))
    app.add_handler(CallbackQueryHandler(tmpl))
    app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, prmpt))
    print("Bot запущен")
    app.run_polling()
    requirements.txt

python-telegram-bot==20.3
requests
python-dotenv
.env.example

TELEGRAM_BOT_TOKEN=your_token_here
VIDEO_API_URL=https://your-video-api
VIDEO_API_KEY=your_api_key_here
