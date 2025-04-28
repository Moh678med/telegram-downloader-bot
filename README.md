import os import asyncio from pyrogram import Client, filters from pyrogram.types import (InlineKeyboardButton, InlineKeyboardMarkup) import yt_dlp

بيانات البوت

API_ID = 24046590 API_HASH = "dae04dc969fd7f29ffb6199903407c9f" BOT_TOKEN = "8070044522:AAHTYxTUagA9X2GgCTMKU4MltM_UlT3VUrw"

LANGUAGE = {}  # تخزين اللغة المختارة لكل مستخدم

app = Client("SocialSaverBot", api_id=API_ID, api_hash=API_HASH, bot_token=BOT_TOKEN)

رسالة البداية

@app.on_message(filters.command("start") & filters.private) async def start(client, message): keyboard = InlineKeyboardMarkup([ [InlineKeyboardButton("العربية", callback_data="lang_ar"), InlineKeyboardButton("English", callback_data="lang_en")] ]) await message.reply_text("\u2728 أهلاً بك في SocialSaver!\n\nPlease choose your language:\n\nاختر لغتك:", reply_markup=keyboard)

تحديد اللغة

@app.on_callback_query() async def select_language(client, callback_query): if callback_query.data == "lang_ar": LANGUAGE[callback_query.from_user.id] = "ar" await callback_query.message.edit_text("\ud83d\udd0d أرسل رابط الفيديو أو الصوت الآن:") elif callback_query.data == "lang_en": LANGUAGE[callback_query.from_user.id] = "en" await callback_query.message.edit_text("\ud83d\udd0d Send the video or audio link now:")

استقبال الروابط

@app.on_message(filters.text & filters.private) async def download_media(client, message): user_lang = LANGUAGE.get(message.from_user.id, "ar") url = message.text status = await message.reply_text("\ud83d\udd0d جاري التحميل..." if user_lang == "ar" else "\ud83d\udd0d Downloading...")

try:
    ydl_opts = {
        'outtmpl': '%(title)s.%(ext)s',
        'format': 'bestvideo+bestaudio/best',
        'merge_output_format': 'mp4',
        'noplaylist': True,
        'quiet': True
    }

    with yt_dlp.YoutubeDL(ydl_opts) as ydl:
        info = ydl.extract_info(url, download=True)
        filename = ydl.prepare_filename(info)

    # إرسال ملف الفيديو
    await message.reply_video(filename, caption=("\u2705 تم التحميل!" if user_lang == "ar" else "\u2705 Downloaded!"))
    os.remove(filename)
    await status.delete()

except Exception as e:
    await status.edit_text(("\u274c حدث خطأ: " if user_lang == "ar" else "\u274c Error:") + str(e))

app.run()

