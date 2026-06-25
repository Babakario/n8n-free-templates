# Script Dialogue Analyzer — راهنمای استفاده

این فایل راهنمای راه‌اندازی و نمونهٔ استفاده برای گردش‌کار n8n به نام "Script Dialogue Analyzer" است. این راهنما به زبان فارسی شامل نمونهٔ درخواست POST، توضیح پارامترها، مراحل راه‌اندازی credentialها و Pinecone/Google Sheets، مثال‌های بیشتر برای payload و راهنمای اضافه کردن اسکرین‌شات است.

## خلاصه
این workflow متن اسکریپت یا دیالوگ را دریافت می‌کند، آن را به قطعات تقسیم می‌کند، برای هر قطعه embedding تولید کرده و در Pinecone ایندکس می‌کند. سپس با استفاده از جستجوی برداری و یک عامل (agent) همراه با حافظه و مدل زبانی، پاسخ‌ها و تحلیل‌های مبتنی بر متن تولید می‌کند و لاگ‌ها را در Google Sheets ذخیره می‌کند.

## مسیر فایل JSON
Media/script_dialogue_analyzer.json

## نمونهٔ درخواست POST (cURL)
فرض می‌کنیم n8n روی https://your-n8n-host تنظیم شده و webhook مسیر `script_dialogue_analyzer` دارد.

- JSON ساده (همان‌طور که قبلاً نشان داده شد):

curl -X POST "https://your-n8n-host/webhook/script_dialogue_analyzer" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "نمونه اسکریپت",
    "author": "مثال",
    "text": "این یک دیالوگ نمونه است. شخصیت A: سلام! شخصیت B: سلام، چطوری؟ ...",
    "meta": { "source": "draft", "language": "fa" }
  }'

- ارسال فایل متنی با multipart/form-data (مثلاً اگر می‌خواهید فایل .txt آپلود کنید):

curl -X POST "https://your-n8n-host/webhook/script_dialogue_analyzer" \
  -F "file=@./my_script.txt" \
  -F "title=اسکریپت صحنه 1" \
  -F "meta={\"language\":\"fa\"}" \
  -H "Accept: application/json"

توجه: برای استفاده از آپلود فایل باید node Webhook را طوری پیکربندی کنید که فایل‌ها را بپذیرد و سپس در workflow از nodeهایی برای خواندن محتوای فایل استفاده شود (مثلاً Read Binary File یا تبدیل باینری به متن).

- مثال با Python (requests):

```python
import requests
url = "https://your-n8n-host/webhook/script_dialogue_analyzer"
payload = {
    "title": "نمونه اسکریپت از پایتون",
    "author": "Tester",
    "text": "دیالوگ A: سلام. دیالوگ B: خوبی؟"
}
resp = requests.post(url, json=payload)
print(resp.status_code, resp.text)
```

## نمونهٔ پاسخ webhook
بستگی به این دارد که در workflow خروجی‌ای تنظیم شده باشد یا خیر. یک پاسخ پیشنهادی که می‌توانید در node آخر قرار دهید:

HTTP 200 OK
{
  "status": "ok",
  "indexed_chunks": 12,
  "message": "Text received and indexed successfully.",
  "workflowRunId": "<n8n-execution-id>"
}

اگر می‌خواهید بلافاصله پس از POST اطلاعاتی مثل تعداد chunk‌های وارد شده یا شناسهٔ اجرا را دریافت کنید، باید یک node HTTP Response یا Set + Respond در انتهای جریان اضافه کنید.

## فیلدهای پیشنهادی برای payload و لاگ
برای راحتی آنالیز و فیلترینگ در آینده، پیشنهاد می‌کنم این ستون‌ها را در Google Sheets لاگ کنید:
- timestamp (زمان درخواست)
- title
- author
- language
- source (مثلاً draft, final, upload)
- indexed_chunks (تعداد chunkهای اندکس‌شده)
- query (اگر از RAG برای سوال/پاسخ استفاده شده)
- response_summary (خلاصهٔ خروجی عامل)
- matched_chunk_ids (شناسهٔ قطعاتی که هنگام بازیابی همسان شده‌اند)

نمونهٔ سطر (CSV-like):
2026-06-25T12:34:56Z, "نمونه اسکریپت", "مثال", "fa", "draft", 12, "آیا شخصیت A تهدیدآمیز است؟", "پاسخ...", "c1,c5,c11"

## اسکرین‌شات: کجا و چگونه بگیرید
من نمی‌توانم تصاویر را از محیط شما بگیرم، ولی راهنما برای اضافه کردن اسکرین‌شات در repo و نکات گرفتن اسکرین‌شات:

مکان‌های مفید برای اسکرین‌شات:
- صفحه اصلی workflow در n8n (نمای کلی canvas) — نام workflow و چینش nodeها
- تنظیمات node Webhook (نمای path و روش HTTP)
- Execution details در n8n برای یک اجرا (نمای خروجی هر node و لاگ‌ها)
- Pinecone console: نمای index و تعداد vectorهای موجود
- Google Sheets: نمای جدول لاگ (ستون‌ها و نمونهٔ ردیف‌های وارد شده)

نام‌گذاری پیشنهادی فایل‌ها و مسیر ذخیره در repo:
- Media/screenshots/script_dialogue_canvas.png
- Media/screenshots/webhook_settings.png
- Media/screenshots/execution_detail.png
- Media/screenshots/pinecone_index.png
- Media/screenshots/google_sheets_log.png

نکات فنی برای اسکرین‌شات:
- در n8n از حالت Fullscreen canvas استفاده کنید تا همهٔ nodeها در یک تصویر دیده شوند.
- برای Execution details، روی یک اجرا کلیک کرده و بخش هر node را باز کنید تا خروجی مربوطه قابل مشاهده باشد.
- قبل از گرفتن اسکرین‌شات کلیدها/توکن‌ها را مخفی کنید یا صفحه را طوری ببرید که مقادیر حساس نشان داده نشوند.

اگر شما تصاویر را آپلود کنید (فایل‌های PNG/JPEG)، من می‌توانم آن‌ها را در مسیرهای بالا در مخزن اضافه و commit کنم.

## گام‌های راه‌اندازی (خلاصه)
1. Import فایل JSON: Media/script_dialogue_analyzer.json
2. تنظیم credentialها: HuggingFace، Pinecone، OpenAI، Google Sheets
3. ساخت ایندکس Pinecone با نام `script_dialogue_analyzer` (یا بروز کردن نام در nodeها)
4. وارد کردن `SHEET_ID` و اطمینان از وجود شیت "Log"
5. بررسی پارامترها: chunkSize، chunkOverlap، مدل امبدینگ
6. Activate و ارسال POST تست

## رفع اشکال سریع
- اگر فایل آپلود می‌کنید و متن خوانده نشد: اطمینان حاصل کنید در n8n node مربوط به خواندن باینری (Binary to Text) اضافه شده است.
- خطاهای Pinecone: بعد از ساخت ایندکس، تست کنید با SDK یا کنسول Pinecone که ایندکس خالی یا حاوی بردارهاست.
- برای خطاهای OpenAI/HuggingFace: محدودیت نرخ یا خطاهای ابعادی (dimension) امبدینگ را بررسی کنید.

## پیشنهادی برای توسعه‌های بعدی
- افزودن endpoint GET برای سلامت (health) یا وضعیت ایندکس
- وارد کردن متادیتای دقیق‌تر مانند scene_number, character_list
- اضافه کردن یک UI ساده (نشان دادن نتایج جستجوی معنایی)

---

فایل راهنما در مسیر `Media/script_dialogue_analyzer_README.md` به‌روز شد و مثال‌های بیشتر payload و راهنمای اسکرین‌شات اضافه شد.

اگر می‌خواهید من تصاویر را هم اضافه کنم یا این تغییر را در یک شاخهٔ جدید قرار دهم و برای آن Pull Request باز کنم، بگویید کدام گزینه را می‌خواهید تا انجام دهم.
