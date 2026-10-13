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
