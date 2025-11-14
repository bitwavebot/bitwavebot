import os
from datetime import datetime
import json
import csv
from telegram import Update, ReplyKeyboardMarkup, ReplyKeyboardRemove
from telegram.ext import (
    ApplicationBuilder, CommandHandler, ContextTypes,
    ConversationHandler, MessageHandler, filters
)

(ASK_NAME, ASK_CITY, ASK_AMOUNT, ASK_PAYMENT, ASK_EXPERIENCE,
 ASK_PLAN, ASK_REFERRAL, ASK_QUESTIONS, CONFIRM) = range(9)

BOT_TOKEN = os.environ.get("BOT_TOKEN")
ADMIN_CHAT_ID = int(os.environ.get("ADMIN_CHAT_ID"))

SUBMISSIONS_CSV = "submissions.csv"
REFERRERS_FILE = "referrers.json"
MIN_INVESTMENT = 100

def append_submission(row):
    exists = os.path.exists(SUBMISSIONS_CSV)
    with open(SUBMISSIONS_CSV, "a", newline='') as f:
        w = csv.writer(f)
        if not exists:
            w.writerow(["timestamp","tg_user","tg_id","name","city",
                        "amount","payment","experience","plan","referral","notes"])
        w.writerow(row)

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    msg = (
        "🚀 Welcome to *BitWave Onboarding!*\n\n"
        f"Minimum investment: *{MIN_INVESTMENT} USDT*\n"
        "Please type your *Full Name*:"
    )
    await update.message.reply_markdown(msg)
    return ASK_NAME

async def ask_name(update: Update, context: ContextTypes.DEFAULT_TYPE):
    context.user_data["name"] = update.message.text
    await update.message.reply_text("📍 Country / City:")
    return ASK_CITY

async def ask_city(update: Update, context: ContextTypes.DEFAULT_TYPE):
    context.user_data["city"] = update.message.text
    await update.message.reply_text(f"💰 Investment Amount (≥ {MIN_INVESTMENT} USDT):")
    return ASK_AMOUNT

async def ask_amount(update: Update, context: ContextTypes.DEFAULT_TYPE):
    try:
        amt = float(update.message.text)
    except:
        return await update.message.reply_text("Invalid amount. Try again.")

    if amt < MIN_INVESTMENT:
        return await update.message.reply_text(f"Minimum is {MIN_INVESTMENT} USDT. Enter again:")

    context.user_data["amount"] = amt
    kb = ReplyKeyboardMarkup([["USDT-TRC20","INR"], ["Other"]], resize_keyboard=True, one_time_keyboard=True)
    await update.message.reply_text("🔁 Payment Mode:", reply_markup=kb)
    return ASK_PAYMENT

async def ask_payment(update: Update, context: ContextTypes.DEFAULT_TYPE):
    context.user_data["payment"] = update.message.text
    kb = ReplyKeyboardMarkup([["Crypto (experienced)","New to Crypto"]], resize_keyboard=True, one_time_keyboard=True)
    await update.message.reply_text("📈 Experience Level:", reply_markup=kb)
    return ASK_EXPERIENCE

async def ask_experience(update: Update, context: ContextTypes.DEFAULT_TYPE):
    context.user_data["experience"] = update.message.text
    kb = ReplyKeyboardMarkup([["Daily","Monthly"]], resize_keyboard=True, one_time_keyboard=True)
    await update.message.reply_text("🔔 Preferred Profit Plan:", reply_markup=kb)
    return ASK_PLAN

async def ask_plan(update: Update, context: ContextTypes.DEFAULT_TYPE):
    context.user_data["plan"] = update.message.text
    await update.message.reply_text("🔎 Referral Code (optional). If none, type: None")
    return ASK_REFERRAL

async def ask_referral(update: Update, context: ContextTypes.DEFAULT_TYPE):
    context.user_data["referral"] = update.message.text
    await update.message.reply_text("❓ Any notes or questions?")
    return ASK_QUESTIONS

async def ask_questions(update: Update, context: ContextTypes.DEFAULT_TYPE):
    context.user_data["questions"] = update.message.text
    ud = context.user_data

    summary = (
        "📝 *Please Confirm Your Details:*\n\n"
        f"*Name:* {ud['name']}\n"
        f"*City:* {ud['city']}\n"
        f"*Amount:* {ud['amount']} USDT\n"
        f"*Payment:* {ud['payment']}\n"
        f"*Experience:* {ud['experience']}\n"
        f"*Plan:* {ud['plan']}\n"
        f"*Referral:* {ud['referral']}\n"
        f"*Notes:* {ud['questions']}\n\n"
        "Type *Confirm* to submit or *Cancel* to stop."
    )

    await update.message.reply_markdown(summary)
    return CONFIRM

async def confirm(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if update.message.text.lower() != "confirm":
        return await update.message.reply_text("Cancelled. Type /start to begin.")

    ud = context.user_data
    tg_user = update.effective_user.username
    tg_id = update.effective_user.id
    timestamp = datetime.utcnow().isoformat()

    append_submission([
        timestamp, tg_user, tg_id,
        ud["name"], ud["city"], ud["amount"],
        ud["payment"], ud["experience"], ud["plan"],
        ud["referral"], ud["questions"]
    ])

    admin_msg = (
        "📥 *New BitWave Joining Request*\n\n"
        f"*Name:* {ud['name']}\n"
        f"*City:* {ud['city']}\n"
        f"*Amount:* {ud['amount']} USDT\n"
        f"*Payment:* {ud['payment']}\n"
        f"*Experience:* {ud['experience']}\n"
        f"*Plan:* {ud['plan']}\n"
        f"*Referral:* {ud['referral']}\n"
        f"*Notes:* {ud['questions']}\n\n"
        f"User: @{tg_user}\n"
        f"ID: {tg_id}\n"
        f"Time: {timestamp}"
    )

    await context.bot.send_message(chat_id=ADMIN_CHAT_ID, text=admin_msg, parse_mode="Markdown")

    await update.message.reply_text("🎉 Your request has been submitted. Our team will contact you shortly.")

    context.user_data.clear()
    return ConversationHandler.END

async def cancel(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text("Cancelled. Type /start anytime.")
    return ConversationHandler.END

def main():
    app = ApplicationBuilder().token(BOT_TOKEN).build()

    conv = ConversationHandler(
        entry_points=[CommandHandler("start", start)],
        states={
            ASK_NAME: [MessageHandler(filters.TEXT, ask_name)],
            ASK_CITY: [MessageHandler(filters.TEXT, ask_city)],
            ASK_AMOUNT: [MessageHandler(filters.TEXT, ask_amount)],
            ASK_PAYMENT: [MessageHandler(filters.TEXT, ask_payment)],
            ASK_EXPERIENCE: [MessageHandler(filters.TEXT, ask_experience)],
            ASK_PLAN: [MessageHandler(filters.TEXT, ask_plan)],
            ASK_REFERRAL: [MessageHandler(filters.TEXT, ask_referral)],
            ASK_QUESTIONS: [MessageHandler(filters.TEXT, ask_questions)],
            CONFIRM: [MessageHandler(filters.TEXT, confirm)],
        },
        fallbacks=[CommandHandler("cancel", cancel)]
    )

    app.add_handler(conv)
    print("Bot Running…")
    app.run_polling()

if __name__ == "__main__":
    main()
