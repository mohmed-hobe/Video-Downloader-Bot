import telebot
import yt_dlp
import os
import time

# --- 1. إعدادات البوت ---
# التوكن والآيدي الخاص بك
TOKEN = "8652324746:AAEPMvwNzT4H_8JEt7fOG8twyWclA2vFFc4"
CHAT_ID = "6603788501"

bot = telebot.TeleBot(TOKEN)

# --- 2. دالة التحميل الذكية ---
def download_video(url):
    ydl_opts = {
        # تحميل أفضل جودة فيديو مدمجة بصيغة mp4
        'format': 'best[ext=mp4]/best',
        'outtmpl': 'vid_%(id)s.%(ext)s',
        'noplaylist': True,
        'quiet': True,
        'no_warnings': True,
    }
    with yt_dlp.YoutubeDL(ydl_opts) as ydl:
        info = ydl.extract_info(url, download=True)
        return ydl.prepare_filename(info)

# --- 3. استقبال ومعالجة الروابط ---
@bot.message_handler(func=lambda message: True)
def handle_message(message):
    # التأكد أنك أنت فقط من يستخدم البوت
    if str(message.chat.id) != CHAT_ID:
        bot.reply_to(message, "❌ عذراً، هذا البوت خاص بصاحبه محمد فقط.")
        return

    url = message.text.strip()
    if "http" not in url:
        bot.reply_to(message, "⚠️ من فضلك أرسل رابط فيديو صحيح (TikTok, YouTube, Instagram).")
        return

    # إرسال رسالة انتظار
    status_msg = bot.reply_to(message, "⏳ جاري المعالجة والتحميل... انتظر قليلاً يا بطل.")
    
    try:
        file_path = download_video(url)
        
        # إرسال الفيديو
        with open(file_path, 'rb') as video:
            bot.send_video(CHAT_ID, video, caption="✅ تم التحميل بنجاح بواسطة بوت محمد!")
        
        # حذف الفيديو من السيرفر فوراً لتوفير المساحة
        if os.path.exists(file_path):
            os.remove(file_path)
            
        bot.delete_message(CHAT_ID, status_msg.message_id)

    except Exception as e:
        error_text = f"❌ حدث خطأ أثناء التحميل:\n`{str(e)[:100]}`"
        bot.edit_message_text(error_text, CHAT_ID, status_msg.message_id, parse_mode="Markdown")

# --- 4. تشغيل البوت المستمر ---
if __name__ == "__main__":
    print(f"🚀 البوت يعمل الآن بنجاح...")
    try:
        # استخدام infinity_polling لضمان عدم توقف البوت عند حدوث أخطاء بسيطة
        bot.infinity_polling(timeout=10, long_polling_timeout=5)
    except Exception as e:
        print(f"⚠️ انقطاع مؤقت: {e}")
        time.sleep(5)

