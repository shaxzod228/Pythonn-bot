# Pythonn-bot
import logging
from aiogram import Bot, Dispatcher, types
from aiogram.types import Message
from aiogram.utils import executor
import yt_dlp
import os

API_TOKEN = "7587725684:AAHk_3GbzsHD3w8WxVDMwVLjWSQvjc2n-EQ"

# Logging
logging.basicConfig(level=logging.INFO)

# Bot & Dispatcher
bot = Bot(token=API_TOKEN)
dp = Dispatcher(bot)

# Yuklab olish funksiyasi
def download_audio(url: str):
    ydl_opts = {
        'format': 'bestaudio/best',
        'postprocessors': [{
            'key': 'FFmpegExtractAudio',
            'preferredcodec': 'mp3',
            'preferredquality': '192',
        }],
        'outtmpl': 'downloads/%(title)s.%(ext)s',
        'quiet': True,
    }

    with yt_dlp.YoutubeDL(ydl_opts) as ydl:
        info = ydl.extract_info(url, download=True)
        filename = ydl.prepare_filename(info)
        audio_file = filename.replace(".webm", ".mp3").replace(".m4a", ".mp3")
        return audio_file

# Start komandasi
@dp.message_handler(commands=['start'])
async def send_welcome(message: Message):
    await message.reply("🎵 Menga YouTube, Instagram yoki TikTok havolasini yuboring — men sizga audio faylini yuboraman.")

# Linkni ko‘rib chiqish
@dp.message_handler()
async def handle_link(message: Message):
    url = message.text.strip()

    if any(domain in url for domain in ["youtube.com", "youtu.be", "instagram.com", "tiktok.com"]):
        await message.reply("🔄 Yuklab olinmoqda... Biroz kuting.")
        try:
            file_path = download_audio(url)
            with open(file_path, 'rb') as audio:
                await message.reply_audio(audio)
            os.remove(file_path)
        except Exception as e:
            await message.reply(f"❌ Xatolik yuz berdi:\n{str(e)}")
    else:
        await message.reply("⛔ Faqat YouTube, Instagram yoki TikTok havolalarini yuboring.")

if __name__ == '__main__':
    if not os.path.exists('downloads'):
        os.makedirs('downloads')
    executor.start_polling(dp, skip_updates=True)
