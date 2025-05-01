import telebot

import yt_dlp

import re

import os

from telebot.types import InlineKeyboardButton, InlineKeyboardMarkup

from flask import Flask

from threading import Thread



# إعدادات Flask للتشغيل المستمر

app = Flask('')



@app.route('/')

def home():

    return "البوت يعمل!"



def run():

    app.run(host='0.0.0.0', port=8080)



def keep_alive():

    Thread(target=run).start()



# بيانات البوت

# بيانات البوت

API_TOKEN = '8070044522:AAHTYxTUagA9X2GgCTMKU4MltM_UlT3VUrw'

# معرف الأدمن فقط

ADMIN_ID = 7049839053  # 

OWNER_ID = 7049839053  # 

bot = telebot.TeleBot(API_TOKEN)

os.makedirs('downloads', exist_ok=True)

subscribers_file = 'subscribers.txt'



def is_valid_url(url):

    return re.match(r'https?://[^\s]+', url)



def download_video(url):

    try:

        ydl_opts = {

            'format': 'best',

            'outtmpl': 'downloads/%(title).80s.%(ext)s',

            'noplaylist': True,

            'quiet': True,

            'no_warnings': True,

        }

        if os.path.exists("cookies.txt"):

            ydl_opts['cookiefile'] = 'cookies.txt'



        with yt_dlp.YoutubeDL(ydl_opts) as ydl:

            info = ydl.extract_info(url, download=True)

            path = ydl.prepare_filename(info)

            return path, None

    except Exception as e:

        return None, str(e)



def add_user(user_id):

    if not os.path.exists(subscribers_file):

        with open(subscribers_file, 'w'):

            pass

    with open(subscribers_file, 'r') as f:

        users = f.read().splitlines()

    if str(user_id) not in users:

        with open(subscribers_file, 'a') as f:

            f.write(str(user_id) + '\n')



def get_all_users():

    if not os.path.exists(subscribers_file):

        return []

    with open(subscribers_file, 'r') as f:

        return f.read().splitlines()



@bot.message_handler(commands=['start'])

def start(message):

    add_user(message.chat.id)

    text = (

        "مرحباً بك في بوت SocialSaver!\n\n"

        "أرسل رابط فيديو من TikTok أو YouTube أو Instagram أو Twitter أو Snapchat،"

        " وسأقوم بتحميله لك بأعلى جودة وبدون شعار."

    )

    keyboard = InlineKeyboardMarkup()

    keyboard.add(

        InlineKeyboardButton("مشاركة البوت", switch_inline_query=""),

        InlineKeyboardButton("تابع قناتنا", url="https://t.me/SocialSaverChannel")

    )

    bot.send_message(message.chat.id, text, reply_markup=keyboard)



@bot.message_handler(func=lambda m: is_valid_url(m.text))

def handle_download(message):

    add_user(message.chat.id)

    status = bot.reply_to(message, "جارٍ التحميل... انتظر لحظات")

    path, error = download_video(message.text)

    if error:

        bot.edit_message_text(f"حدث خطأ:\n{error}", message.chat.id, status.message_id)

    else:

        try:

            with open(path, 'rb') as f:

                bot.send_video(message.chat.id, f, caption="تم التحميل بنجاح!", reply_to_message_id=message.message_id)

            os.remove(path)

        except:

            bot.send_message(message.chat.id, "حدث خطأ أثناء إرسال الملف.")



@bot.message_handler(commands=['stats'])

def stats(message):

    if message.from_user.id == ADMIN_ID:

        users = get_all_users()

        bot.reply_to(message, f"عدد المشتركين: {len(users)}")

    else:

        bot.reply_to(message, "هذه الميزة مخصصة لصاحب البوت فقط.")



@bot.message_handler(commands=['broadcast'])

def ask_broadcast(message):

    if message.from_user.id != ADMIN_ID:

        return bot.reply_to(message, "هذه الميزة مخصصة لصاحب البوت فقط.")



    msg = bot.reply_to(message, "أرسل الآن الرسالة أو الوسائط التي تريد إرسالها لجميع المشتركين.")



@bot.message_handler(func=lambda m: True, content_types=['text', 'photo', 'video'])

def send_to_all_users(msg):

    if msg.chat.id != ADMIN_ID:

        return

    users = get_all_users()

    sent, failed = 0, 0

    for user in users:

        try:

            if msg.content_type == 'text':

                bot.send_message(user, msg.text)

            elif msg.content_type == 'photo':

                bot.send_photo(user, msg.photo[-1].file_id, caption=msg.caption or '')

            elif msg.content_type == 'video':

                bot.send_video(user, msg.video.file_id, caption=msg.caption or '')

            sent += 1

        except:

            failed += 1

    bot.send_message(ADMIN_ID, f"تم الإرسال إلى {sent}، فشل في {failed}")

    bot.unregister_message_handler(send_to_all_users)



@bot.message_handler(func=lambda m: True)

def fallback(message):

    bot.reply_to(message, "يرجى إرسال رابط فيديو صالح من أحد المواقع المدعومة.")

keep_alive()

bot.remove_webhook()

bot.polling()
