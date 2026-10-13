# Лаборатори №6 — Тест тохиолдол боловсруулах — Category-Partition ба PICT

**Хичээл:** F.CSA313 — Программ хангамжийн чанарын баталгаа ба тест

**Оюутан:** Б.Мөнгөншагай

**Оюутны код:** B232270013

# Хэсэг A: Category-Partition гараар — Шилжүүлгийн функц

POST /transfer { "from": ID, "to": ID, "amount": NUM, "currency": CODE }


Дүрмүүд:

- from данс байх ёстой, царцаагдаагүй (frozen биш) байх ёстой

- to данс байх ёстой, from-оос өөр байх ёстой

- amount > 0, дансны үлдэгдлээс хэтрэхгүй, өдрийн лимитээс хэтрэхгүй

- currency данснаас өөр бол ханшийн хөрвүүлэлт хийгдэнэ

- Амжилт: 201 {"result":"OK"} · Оролтын алдаа: 200 {"result":"ERROR_..."}

ERROR_NO_FROM, ERROR_FROZEN, ERROR_NO_TO, ERROR_SAME_ACCOUNT,

ERROR_BAD_AMOUNT (amount ≤ 0), ERROR_INSUFFICIENT, ERROR_LIMIT

- amount нь currency-гийн валютаар; үлдэгдэл, лимитийг хөрвүүлсний дараах дүнгээр шалгана

## A1 — Сонголт ба төлөөлөх утгууд

Тестлэх API:

POST /transfer

Оролтын параметрүүд:
from, to, amount, currency.

### Category-Partition хүснэгт

| Category | Choice | Төлөөлөх утга | Тайлбар |
|---|---|---|---|
| From | Valid | ACC1001 | Идэвхтэй данс |
| From | Frozen | ACC1002 | Царцаагдсан данс |
| From | Missing | ACC9999 | Бүртгэлгүй данс |
| To | Valid | ACC2001 | Өөр хүчинтэй данс |
| To | Missing | ACC9998 | Бүртгэлгүй хүлээн авагч |
| To | SameAsFrom | ACC1001 | Өөрийн данс руу шилжүүлэх |
| Amount | Zero | 0 | Дүн тэгтэй тэнцүү |
| Amount | Normal | 100000 | Зөвшөөрөгдөх дүн |
| Amount | OverBalance | 600000 | Үлдэгдлээс давсан дүн |
| Amount | OverLimit | 400000 | Өдрийн үлдсэн лимитээс давсан дүн |
| Currency | Same | MNT | Дансны валюттай ижил |
| Currency | Different | USD | Ханшийн хөрвүүлэлт шаардлагатай |

### Далд орчны нөхцөлүүд

Банкны шилжүүлгийн үр дүн нь зөвхөн хүсэлтийн
дөрвөн параметрээс хамаарахгүй.

Нэмэлтээр дараах нөхцөлийг харгалзан үзнэ:

1. Дансны үлдэгдэл (balance).
2. Өдрийн лимит (remaining daily limit).
3. Дансны үндсэн валют (account currency).
4. Валют хөрвүүлэх ханш (exchange rate).
5. Дансны төлөв (active/frozen).
6. Данс системд бүртгэлтэй эсэх.

Жишээ тестийн орчин:

- ACC1001: active, MNT, balance = 500000.
- ACC1002: frozen, MNT.
- ACC2001: active, MNT.
- ACC9999 болон ACC9998: бүртгэлгүй.
- Өдрийн үлдсэн лимит = 300000 MNT.

Normal = 100000 MNT:
үлдэгдэл болон лимитийн хүрээнд байна.

OverBalance = 600000 MNT:
үлдэгдлээс хэтэрнэ.

OverLimit = 400000 MNT:
өдрийн лимитээс хэтэрнэ, харин үлдэгдэлд багтана.

Currency = Different үед amount нь USD валютын
дүн байна. Дансны валют MNT тул хөрвүүлсний
дараах дүнг баланс болон лимиттэй харьцуулна.

## A2 — Хязгаарлалтууд

| Choice | Тэмдэглэгээ | Үндэслэл |
|---|---|---|
| From.Valid | [property] | Хүчинтэй эх данс |
| From.Frozen | [ERROR] | ERROR_FROZEN |
| From.Missing | [ERROR] [SINGLE] | ERROR_NO_FROM |
| To.Valid | [property] | Хүчинтэй хүлээн авагч |
| To.Missing | [ERROR] | ERROR_NO_TO |
| To.SameAsFrom | [ERROR] [IF] | Эх данстай ижил |
| Amount.Zero | [ERROR] | ERROR_BAD_AMOUNT |
| Amount.Normal | [property] | Зөвшөөрөгдөх дүн |
| Amount.OverBalance | [ERROR] [IF] | Үлдэгдэл хүрэлцэхгүй |
| Amount.OverLimit | [ERROR] [IF] | Лимит хүрэлцэхгүй |
| Currency.Same | [property] | Хөрвүүлэлтгүй |
| Currency.Different | [IF] | Хөрвүүлэлттэй |

### Хязгаарлалтын тайлбар

[ERROR] нь сонголт дангаараа алдаа үүсгэхийг
илэрхийлнэ.

[SINGLE] нь нэг төлөөлөх бие даасан тестээр
алдааг шалгах боломжтойг илэрхийлнэ.

From.Missing сонголтыг [SINGLE] гэж үзсэн.
Учир нь эх данс байхгүй үед шилжүүлэг хийх
боломжгүй бөгөөд ERROR_NO_FROM буцна.

[IF] нь бусад сонголт эсвэл орчны
нөхцөлөөс хамаарах дүрмийг илэрхийлнэ.

### Олон алдааны давуу эрэмбэ

From → To → Amount дарааллаар эхний
илэрсэн алдааг буцаахаар тогтоосон.

Энэ эрэмбийг эхлээд дансуудыг баталгаажуулж,
дараа нь дүнг шалгах логиктой гэж үзэн сонгосон.

Бодит API-д ийм эрэмбэ заагдаагүй тул
энэ нь тест дизайны таамаглал юм.

### IF нөхцөлүүд

IF From = Missing THEN Amount = Normal

IF To = Missing THEN Amount = Normal

Эдгээр нь хослолын тоог хязгаарлах загварчлалын
шийдвэр бөгөөд API-ийн заавал мөрдөх бизнес
дүрэм гэж үзээгүй.

### Комбинацийн тооцоо

N = 3 × 3 × 4 × 2 = 72

From.Missing үед 18 комбинаци хасагдана.
To.Missing үед 18 комбинаци хасагдана.

6 комбинаци давхар хасагдсан тул
нэг удаа буцааж нэмнэ.

M = 72 - 18 - 18 + 6 = 42

Үр дүн: 72 → 42

[SINGLE] тэмдэглэгээг 42 тоонд
дахин хасалт гэж оруулаагүй.

## A3 — Тестийн спецификаци

Нийт 12 тестийн спецификаци боловсруулав.

| ID | From | To | Amount | Currency | Expected |
|---|---|---|---|---|---|
| TC01 | Valid | Valid | Normal | Same | 201 OK |
| TC02 | Valid | Valid | Normal | Different | 201 OK |
| TC03 | Missing | Valid | Normal | Same | 200 ERROR_NO_FROM |
| TC04 | Frozen | Valid | Normal | Same | 200 ERROR_FROZEN |
| TC05 | Valid | Missing | Normal | Same | 200 ERROR_NO_TO |
| TC06 | Valid | SameAsFrom | Normal | Same | 200 ERROR_SAME_ACCOUNT |
| TC07 | Valid | Valid | Zero | Same | 200 ERROR_BAD_AMOUNT |
| TC08 | Valid | Valid | OverBalance | Same | 200 ERROR_INSUFFICIENT |
| TC09 | Valid | Valid | OverLimit | Same | 200 ERROR_LIMIT |
| TC10 | Missing | Missing | Normal | Different | 200 ERROR_NO_FROM |
| TC11 | Frozen | Missing | Normal | Same | 200 ERROR_FROZEN |
| TC12 | Valid | SameAsFrom | OverLimit | Different | 200 ERROR_SAME_ACCOUNT |

### Тестийн зорилго

TC01: Хүчинтэй дансуудын хооронд энгийн
амжилттай шилжүүлэг шалгана.

TC02: Валют хөрвүүлсний дараа амжилттай
шилжүүлэг хийгдэхийг шалгана.

TC03: [SINGLE] сонголтоор эх данс байхгүй
алдааг тусад нь шалгана.

TC04: Царцаагдсан дансны алдааг шалгана.

TC05: Хүлээн авагч байхгүй алдааг шалгана.

TC06: Өөрийн данс руу шилжүүлэх алдааг шалгана.

TC07: Тэг дүнтэй шилжүүлгийн алдааг шалгана.

TC08: Үлдэгдэл хүрэлцэхгүй алдааг шалгана.

TC09: Өдрийн лимит хэтэрсэн алдааг шалгана.

TC10: From ба To хоёулаа байхгүй үед
ERROR_NO_FROM давуу эрхтэйг шалгана.

TC11: Frozen ба Missing To давхцахад
ERROR_FROZEN буцахыг шалгана.

TC12: SameAsFrom болон OverLimit давхцахад
ERROR_SAME_ACCOUNT буцахыг шалгана.

### Oracle буюу хүлээгдэх үр дүн

Амжилттай шилжүүлэг:
HTTP 201, {"result":"OK"}

Алдаатай шилжүүлэг:
HTTP 200, {"result":"ERROR_..."}

Олон алдаа давхцсан үед:
From → To → Amount эрэмбийг хэрэглэнэ.

Different валюттай үед amount нь хүсэлтийн
валютаар илэрхийлэгдсэн бөгөөд хөрвүүлсний
дараах дүнгээр үлдэгдэл, лимитийг шалгана.
