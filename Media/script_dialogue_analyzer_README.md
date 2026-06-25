# Script Dialogue Analyzer — راهنمای استفاده

این فایل راهنمای راه‌اندازی و نمونهٔ استفاده برای گردش‌کار n8n به نام "Script Dialogue Analyzer" است. این راهنما به زبان فارسی شامل نمونهٔ درخواست POST، توضیح پارامترها، و مراحل راه‌اندازی credentialها و Pinecone/Google Sheets می‌باشد.

## خلاصه
این workflow متن اسکریپت یا دیالوگ را دریافت می‌کند، آن را به قطعات تقسیم می‌کند، برای هر قطعه embedding تولید کرده و در Pinecone ایندکس می‌کند. سپس با استفاده از جستجوی برداری و یک عامل (agent) همراه با حافظه و مدل زبانی، پاسخ‌ها و تحلیل‌های مبتنی بر متن تولید می‌کند و لاگ‌ها را در Google Sheets ذخیره می‌کند.

## مسیر فایل JSON
Media/script_dialogue_analyzer.json

## نمونهٔ درخواست POST (cURL)
فرض می‌کنیم n8n روی https://your-n8n-host تنظیم شده و webhook مسیر `script_dialogue_analyzer` دارد:

curl -X POST "https://your-n8n-host/webhook/script_dialogue_analyzer" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "نمونه اسکریپت",
    "author": "مثال",
    "text": "این یک دیالوگ نمونه است. شخصیت A: سلام! شخصیت B: سلام، چطوری؟ ...",
    "meta": { "source": "draft", "language": "fa" }
  }'

توضیحات فیلدها:
- `text`: متن کامل اسکریپت/دیالوگ — مهم‌ترین فیلد.
- `title`، `author`، `meta`: دلخواه؛ برای لاگ و متادیتا مناسب‌اند.

> توجه: بسته به کانفیگ webhook شما، ممکن است نیاز باشد فیلدها یا ساختار JSON را تغییر دهید. این مثال عمومی است.

## مراحل راه‌اندازی
1. وارد n8n شوید و فایل JSON `Media/script_dialogue_analyzer.json` را Import کنید (Import > Workflow > Upload file).
2. اعتبارنامه‌ها (Credentials) را بسازید یا مقداردهی کنید:
   - HuggingFace: توکن API (نام credential در workflow: "HuggingFace" / HF_API)
   - Pinecone: API key و environment (نام credential: "Pinecone account" / PINECONE_API)
   - OpenAI: API key (نام credential: "OpenAI" / OPENAI_API)
   - Google Sheets (OAuth2): credential برای نوشتن در شیت (نام credential: "Sheets" / SHEETS_API)

3. در Pinecone یک index بسازید با نام `script_dialogue_analyzer` (یا نام دلخواه و سپس همان نام را در nodeهای Insert و Query در workflow بروز کنید). دقت کنید dim (ابعاد) با مدل embedding شما هم‌خوانی داشته باشد.
4. مقداردهی `SHEET_ID` در node Google Sheets: شناسهٔ Spreadsheet خود را در فیلد `documentId` قرار دهید و مطمئن شوید شیتی با نام "Log" وجود دارد یا نام را مطابق شیت‌تان تغییر دهید.
5. پارامترهای مهم را بررسی و در صورت نیاز تنظیم کنید:
   - Splitter: `chunkSize = 400`, `chunkOverlap = 40` (قابل تغییر)
   - Embeddings: مدل = "default" (می‌توانید مدل HuggingFace مشخص‌تری انتخاب کنید)
   - Agent: `text = {{ $json }}` یعنی کل JSON دریافتی به عامل فرستاده می‌شود
6. Workflow را فعال (Activate) کنید.
7. با استفاده از مثال cURL یا از ابزار HTTP دیگر یک POST به webhook ارسال کنید.

## بررسی و رفع اشکال
- اگر بردارها در Pinecone ثبت نمی‌شوند: مطمئن شوید ایندکس موجود است و credential Pinecone صحیح است.
- خطای احراز هویت (401/403): کلیدها و نام credentialها را چک کنید.
- اگر لاگ‌ها در Google Sheets ظاهر نمی‌شوند: بررسی کنید `SHEET_ID` درست باشد و credential گوگل دسترسی نوشتن داشته باشد.
- برای خطاهای دقیق، در n8n به Execution log بروید و روی اجرای مربوطه کلیک کنید تا خروجی هر node و پیغام خطا را ببینید.

## پیشنهادات توسعه
- اضافه کردن endpoint تأیید (GET webhook) تا وضعیت سلامت workflow را بررسی کنید.
- اضافه کردن مرحلهٔ preprocessing برای حذف نویز و قالب‌بندی بهتر دیالوگ‌ها (مثلاً جدا کردن نام شخصیت‌ها).
- ذخیرهٔ متادیتا (title, author, scene) همراه بردارها برای فیلترهای پیچیده‌تر هنگام بازیابی.

---
فایل راهنما در مسیر `Media/script_dialogue_analyzer_README.md` در مخزن شما اضافه شد.

اگر می‌خواهید من:
- این فایل را در یک شاخه جدید قرار دهم و pull request ایجاد کنم، یا
- مستقیماً یک commit به شاخهٔ پیش‌فرض بزنم
را انجام دهم، بگویید کدام گزینه را ترجیح می‌دهید تا بر اساس آن اقدام کنم.