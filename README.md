# Telegram2
import asyncio
from telegram import Update, InputFile
from telegram.ext import (
    Application, CommandHandler, MessageHandler, ConversationHandler,
    ContextTypes, filters
)
import logging
import os

# فعال‌سازی لاگ‌ها
logging.basicConfig(level=logging.INFO)

# جایگزین کنید با ID عددی ادمین
ADMIN_ID = 237056482

# سوالات
questions = [
    "1. نام و نام خانوادگی شما چیست؟",
    "2. نام پدر شما چیست؟",
    "3. تاریخ تولد شما چیست؟ (روز/ماه/سال)",
    "4. شماره ملی شما چیست؟",
    "5. آدرس محل سکونت شما چیست؟",
    "6. میزان تحصیلات شما چیست؟",
    "7. شماره تماس شما چیست؟",
    "8. آیدی شبکه‌های اجتماعی خود را بنویسید (در صورت نداشتن 'ندارم' را ارسال کنید)",
    "9. از چه راهی با بزرگمهر آشنا شده‌اید؟",
    "10. تاریخ پایان بیمه ورزشی شما چیست؟",
    "11. لطفاً تصویر کارت ملی خود را ارسال بفرمایید."
]

# مراحل گفتگو
ASKING = range(1)

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text("ممنون از اینکه باشگاه بزرگمهر قائنی را برای عضویت انتخاب کرده‌اید. لطفاً به دقت به سوالات زیر پاسخ دهید...")
    await asyncio.sleep(3)
    context.user_data['answers'] = []
    context.user_data['question_index'] = 0
    await update.message.reply_text(questions[0])
    return ASKING

async def handle_response(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if update.message.text:
        context.user_data['answers'].append(update.message.text)
        index = context.user_data['question_index'] + 1

        if index < len(questions):
            context.user_data['question_index'] = index
            await update.message.reply_text(questions[index])
            return ASKING
        else:
            answers = context.user_data['answers']
            response = "\n".join(f"{i+1}. {q}\nپاسخ: {a}" for i, (q, a) in enumerate(zip(questions, answers)))
            await update.message.reply_text("ممنون از پاسخ‌های شما. اطلاعات شما برای ادمین ارسال شد.")
            await context.bot.send_message(chat_id=ADMIN_ID, text=f"کاربر {update.effective_user.full_name} پاسخ داد:\n\n{response}")
            return ConversationHandler.END

    elif update.message.photo:
        file_id = update.message.photo[-1].file_id
        file = await context.bot.get_file(file_id)
        await file.download_to_drive(f"{update.effective_user.id}_national_card.jpg")
        await context.bot.send_photo(chat_id=ADMIN_ID, photo=open(f"{update.effective_user.id}_national_card.jpg", 'rb'))
        await update.message.reply_text("تصویر کارت ملی دریافت شد. با تشکر.")
        return ConversationHandler.END

async def cancel(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text("گفتگو لغو شد.")
    return ConversationHandler.END

if __name__ == '__main__':
    TOKEN = os.getenv("BOT_TOKEN")

    app = Application.builder().token(TOKEN).build()

    conv_handler = ConversationHandler(
        entry_points=[CommandHandler('start', start)],
        states={
            ASKING: [MessageHandler(filters.TEXT & ~filters.COMMAND, handle_response),
                     MessageHandler(filters.PHOTO, handle_response)],
        },
        fallbacks=[CommandHandler('cancel', cancel)],
    )

    app.add_handler(conv_handler)

    print("ربات در حال اجرا است...")
    app.run_polling()

python-telegram-bot==20.6
#!/bin/bash
python bot.py
services:
  - type: web
    name: telegram-bot
    env: python
    plan: free
    buildCommand: ""
    startCommand: bash start.sh
    envVars:
      - key: BOT_TOKEN
        value: YOUR_TELEGRAM_BOT_TOKEN_HERE
        
  requirements.txt
  python-telegram-bot==20.8
aiohttp
git add requirements.txt
git commit -m "Add requirements.txt"
git push

