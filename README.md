# مقدمة إلى محاكاة Chip-8 باستخدام لغة البرمجة Rust

https://github.com/aquova/chip8-book

هذا برنامج تعليمي تمهيدي حول كيفية تطوير أول محاكاة لـ Chip-8 باستخدام لغة Rust، مع استهداف كل من أجهزة الكمبيوتر المكتبية ومتصفحات الويب عبر WebAssembly. يفترض هذا البرنامج التعليمي عدم وجود خبرة سابقة في المحاكاة، ومعرفة أساسية فقط بلغة Rust. يقدم الدليل أولاً نظرة عامة على المكونات المختلفة للمحاكاة، وكيف تعمل جميع أجزاء النظام المحاكى، وما هي الخطوات التي يحتاج مطور المحاكاة إلى تنفيذها. يتبع ذلك شرح تفصيلي خطوة بخطوة لتنفيذ محاكاة Chip-8، مع وصف كل قسم من الأكواد وسبب الحاجة إليه.

- يمكن العثور على الكود المصدري للمحاكاة المكتملة في مجلد `code`
- الكود المصدري لكتاب PDF موجود في مجلد `src`
- نماذج ألعاب Chip-8 موجودة في مجلد `roms`

يمكنك تنزيل أحدث نسخة من الكتاب (المترجم) من هنا: https://github.com/TechComet/chip8-book/releases
يمكنك تنزيل أحدث نسخة من الكتاب (النسخة الأصلية) من هنا: https://github.com/aquova/chip8-book/releases

لإنشاء نسخة من PDF بنفسك، قم أولاً بتثبيت [pandoc](https://pandoc.org/) ثم قم بتشغيل الأمر `make pdf` (أو `make epub` لنسخة ePub). يتم تقديم تفاصيل حول كيفية إعداد بيئة البناء للكود المصدري في ملف PDF، ولكن بمجرد التثبيت، يمكن بناء المحاكاة المكتملة باستخدام `make desktop` أو `make web`

## الاعتمادات

تم توفير ألعاب Chip-8 من [Zophar's Domain](https://www.zophar.net/pdroms/chip8/chip-8-games-pack.html). المؤلف الأصلي غير معروف.
