import telebot
from pptx import Presentation
from reportlab.pdfgen import canvas
import os

# استبدل بـ Token الخاص بك
API_TOKEN = 'YOUR_TELEGRAM_BOT_TOKEN'
bot = telebot.TeleBot(API_TOKEN)

# مجلد لحفظ الملفات
UPLOAD_FOLDER = "uploads"
os.makedirs(UPLOAD_FOLDER, exist_ok=True)

@bot.message_handler(commands=['start', 'help'])
def send_welcome(message):
    bot.reply_to(message, "أهلاً بك! أرسل ملف PPT لتحويله إلى PDF.")

@bot.message_handler(content_types=['document'])
def handle_document(message):
    file_id = message.document.file_id
    file_info = bot.get_file(file_id)
    file_path = file_info.file_path

    # تنزيل الملف
    downloaded_file = bot.download_file(file_path)
    ppt_file = os.path.join(UPLOAD_FOLDER, message.document.file_name)

    with open(ppt_file, 'wb') as new_file:
        new_file.write(downloaded_file)

    # تحويل الملف إلى PDF
    pdf_file = ppt_file.replace('.pptx', '.pdf')
    try:
        convert_ppt_to_pdf(ppt_file, pdf_file)
        with open(pdf_file, 'rb') as pdf:
            bot.send_document(message.chat.id, pdf)
    except Exception as e:
        bot.reply_to(message, f"حدث خطأ أثناء التحويل: {e}")
    finally:
        # حذف الملفات المؤقتة
        os.remove(ppt_file)
        if os.path.exists(pdf_file):
            os.remove(pdf_file)

def convert_ppt_to_pdf(ppt_path, pdf_path):
    prs = Presentation(ppt_path)
    c = canvas.Canvas(pdf_path)

    for slide in prs.slides:
        for shape in slide.shapes:
            if shape.has_text_frame:
                text = shape.text
                c.drawString(100, 800, text)  # إضافة النص إلى الـ PDF (تنسيق بسيط)
        c.showPage()

    c.save()

if __name__ == "__main__":
    print("البوت يعمل...")
    bot.polling()
