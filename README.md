# Sticker Engine

An exteraGram plugin that converts outgoing text messages into transparent WebP stickers with adversarial noise to disrupt mass data harvesting, OCR, and OSINT databases.

---

## 🇷🇺 Русский (Основное описание)

**Anti-OSINT Sticker Engine** - это плагин для exteraGram, который автоматически преобразует ваши исходящие текстовые сообщения в прозрачные стикеры формата WebP, применяя состязательное зашумление (adversarial noise) и микроискажения. Проект разработан для защиты приватности и противодействия массовому нецелевому парсингу чатов ботами автоматического сбора данных (такими как Funstat, Telelog, Глаз Бога и другие OSINT-системы).

### Как это работает (Техническая часть)
Плагин перехватывает исходящий текст на стороне клиента и рендерит его на графическом холсте (Canvas) в среде Java/Android, внедряя несколько уровней защиты:
1. **Обход парсеров текста:** Вместо текстового типа данных отправляется объект медиа (стикер). Простые парсеры баз данных упираются в пустые значения.
2. **Микросдвиги (Micro-jitter):** Координаты символов и теней сдвигаются на случайные доли пикселя при каждом рендере. Это заставляет алгоритмы сглаживания (anti-aliasing) видеокарты генерировать уникальные граничные пиксели для каждого кадра, ломая классическое сравнение хэшей изображений.
3. **Искажение геометрии (Canvas Skewing):** Холст слегка наклоняется по оси X на случайный угол. Это ломает детекцию базовой линии (baseline) в нейросетях OCR.
4. **Состязательное зашумление (Adversarial Noise):** С помощью наложения маски `SRC_ATOP` на буквы наносятся случайные линии и точки разной прозрачности. Это не мешает человеческому глазу распознавать текст (благодаря гештальт-восприятию), но полностью ломает бинаризацию и сегментацию символов в системах распознавания текста (Tesseract, EasyOCR и др.).
5. **Динамическое сжатие:** Качество кодирования WebP колеблется в диапазоне 76-84%, гарантируя уникальный хэш файла (SHA-256) для каждого отправленного сообщения.

### Дисклеймер и модель угроз
**Важно:** Этот инструмент разработан исключительно для защиты от **массового, автоматизированного и нецелевого сбора информации** (mass dragnet surveillance).
* **Целевые атаки:** Если вы являетесь объектом индивидуальной целевой разработки со стороны спецслужб или высококвалифицированных хакеров (APT), этот плагин вас не спасет. 
* В случае направленной атаки злоумышленники могут использовать кейлоггеры на устройстве, перехват скриншотов экрана (screen scraping) или уязвимости нулевого дня в ОС.
* Инструмент предназначен для повышения общего уровня цифровой гигиены в публичных чатах.

---

## 🇬🇧 English

This plugin intercepts outgoing text and renders it onto a WebP vector canvas using adversarial perturbations.

### Core Features
* **Anti-Scraping:** Converts plaintext to sticker media, bypassing raw string loggers.
* **Adversarial Perturbation:** Introduces randomized lines and salt-and-pepper noise clipped directly to the text bounds (`SRC_ATOP`).
* **Geometric Distortion:** Randomly skews the canvas and applies sub-pixel coordinate jitter, breaking baseline detection in standard neural network OCR models.
* **Entropy Generation:** Randomizes WebP compression quality between 76% and 84% to ensure every message has a unique file hash.

### Threat Model & Disclaimer
This tool is meant to protect against **mass, indiscriminate data harvesting** (e.g., OSINT databases and automated scrapers). It does **not** protect against targeted surveillance. If you are specifically targeted by highly funded adversaries or state-level threat actors, they can bypass this technique using OS-level keyloggers, screen capturing, or device-level compromises.

---

## 🇨🇳 简体中文

本插件通过将文本转换为带有对抗性扰动的透明 WebP 贴纸，来干扰大规模数据抓取和自动化 OCR 识别。

### 核心功能
* **防原始文本爬取**：将文本转换为媒体贴纸，使常规文本收集机器人无法直接提取文本。
* **对抗性噪点**：在字符表面混合随机的透明线条与噪点（基于 `SRC_ATOP`），人类肉眼易读，但能有效干扰 OCR（如 Tesseract、EasyOCR 等）的字符分割与识别。
* **几何微调**：对画布进行随机微小倾斜（Skew），并对字符坐标进行亚像素级的抖动，破坏 OCR 神经网络对基准线（Baseline）的定位。
* **文件哈希混淆**：动态调整 WebP 压缩质量（76%-84%），确保每次发送相同文字产生的二进制文件和 SHA-256 哈希完全不同。

### 威胁模型与免责声明
本工具仅用于防御**大规模、无差别的自动化数据收集**（例如各类 OSINT 数据库和社工库）。它**无法**防御针对个人的定向监控。如果面临国家级或拥有大量资源的针对性威胁，攻击者仍可通过系统级键盘记录、屏幕截图或设备提权等手段获取您的信息。

---

## 🇮🇷 فارسی

این پلاگین با تبدیل پیام‌های متنی به استیکرهای شفاف WebP مجهز به نویز تقابلی، از جمع‌آوری انبوه داده‌ها توسط خزنده‌های خودکار جلوگیری می‌کند.

### ویژگی‌های فنی
* **مبارزه با خزش متنی:** تبدیل متن به رسانه جهت خالی گذاشتن فیلد متنی دیتابیس خزنده‌ها.
* **نویز تقابلی لایه حروف:** ترسیم خطوط و نقاط تصادفی با آلفای متغیر روی سطح حروف جهت مخدوش کردن بخش‌بندی تصاویر در موتورهای OCR.
* **اعوجاج هندسی بوم:** اعمال شیب افقی بسیار جزئی روی کل بوم و لرزش صدم‌پیکسلی موقعیت رسم حروف برای به خطا انداختن سیستم‌های تراز افقی هوش مصنوعی.
* **تغییر مداوم هش فایل:** تصادفی‌سازی کیفیت فشرده‌سازی WebP بین ۷۶ تا ۸۴ درصد جهت دگرگونی مداوم بایت‌های فایل و تولید هش‌های یکتا در هر ارسال.

### مدل تهدید و سلب مسئولیت
این ابزار صرفاً برای مقابله با **پایش فله‌ای، انبوه و غیرهدفمند کلمات کلیدی** طراحی شده است. در صورت وجود تهدیدات هدفمند شخصی (Targeted Surveillance) از سوی نهادهای دولتی یا مهاجمان پیشرفته، آن‌ها می‌توانند این سد را از طریق بردارهای فرعی مانند بدافزارهای ضبط کیبورد، تصویربرداری از صفحه نمایش یا آسیب‌پذیری‌های امنیتی سیستم‌عامل دور بزنند.

---

## Installation & Setup / Инструкция по установке

### 1. Font Configuration / Настройка шрифта
To prevent OS-level font fingerprinting, you must supply a custom TrueType Font (`.ttf`):
1. Locate a standard `.ttf` font file (e.g., Arial, DejaVuSans, etc.).
2. Copy this file into your Telegram client's cache directory on Android:
   `Android/data/<your.client.package>/cache/`
3. In the plugin settings, enter the exact filename of the font (e.g., `myfont.ttf`). Do not leave this field blank.

### 2. Plugin Deployment / Установка плагина
1. Download the Python plugin file.
2. Load the plugin into your modded Telegram client (e.g., exteraGram) by uploading it in save message.
3. Turn on the "Enable Sticker Generator" switch in the plugin settings.
