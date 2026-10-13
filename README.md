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
- Өдрийн үлдсэн лимит = 600000 MNT.

Normal = 100000 MNT:
үлдэгдэл болон лимитийн хүрээнд байна.

OverBalance = 550000 MNT:
үлдэгдлээс хэтэрнэ.

OverLimit = 400000 MNT:
OverBalance ба OverLimit нэг орчинд хэрэгжих боломгүй учир энд өдрийн лимитийг 350000 гэж үзнэ.
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

## B1 — Browser Pairwise Testing

### Browser model

Browser model нь 6 параметртэй:

- Content: Allow, Restrict, Block
- Cookies: Allow, Restrict, Block
- Popups: Yes, No
- Addons: Yes, No
- Attacks: Yes, No
- Forgeries: Yes, No

### Тестийн үр дүн

| Туршилт | Мөрийн тоо |
|---|---:|
| Бүрэн комбинац | 144 |
| PICT 2-way | 10 |
| PICT 3-way | 20 |

### Үр дүнгийн тайлбар

PICT 2-way тестээр 10 мөр үүссэн бөгөөд энэ нь
лекц дээр гараар байгуулсан 9 мөрөөс 1-ээр олон байна.

PICT нь greedy алгоритм ашиглан тестийн
комбинац үүсгэдэг тул хамгийн цөөн мөртэй
шийдлийг заавал олдоггүй.

3-way тестээр 20 мөр үүссэн бөгөөд энэ нь
бүрэн 144 комбинацаас 124 мөрөөр цөөн байна.

2-way нь параметрүүдийн хос утгуудыг,
3-way нь гурвалсан утгуудыг хамруулдаг.


## B2 — Transfer PICT Pairwise Testing

### Ашигласан моделууд

1. transfer-nc.model — Хязгаарлалтгүй model.
2. transfer.model — IF хязгаарлалттай model.
3. transfer-neg.model — Negative (~) утгатай model.

### PICT тестийн үр дүн

| Үзүүлэлт | transfer-nc | transfer | transfer-neg |
|---|---:|---:|---:|
| Нийт тестийн мөр | 13 | 12 | 16 |
| Happy path | 1 | 0 | 2 |
| Олон алдаатай мөр | 11 | 9 | 0 |
| Нэг мөрийн хамгийн олон алдаа | 3 | 3 | 1 |

### Хязгаарлалтгүй model

transfer-nc.model нь бүх параметрийн утгуудыг
ямар нэг хязгаарлалтгүйгээр хослуулсан.

Нийт 13 тестийн мөр үүссэн бөгөөд 11 мөрөнд
хоёр буюу түүнээс олон алдаатай утга давхцсан.

Энэ нь error masking үүсэх эрсдэлтэй.
Учир нь эхний илэрсэн алдаа бусад алдааг
шалгах боломжийг хааж болно.

### IF хязгаарлалттай model

transfer.model нь дараах нөхцөлүүдтэй:

IF [From] = "Missing" THEN [Amount] = "Normal";
IF [To] = "Missing" THEN [Amount] = "Normal";

Энэ model-оос нийт 12 тест үүссэн.

PICT-ийн гаралтад happy path байгаагүй.
Мөн 9 мөрөнд олон алдаа зэрэг орсон.

Иймээс IF хязгаарлалтууд error masking-ийг
бүрэн шийдвэрлэж чадахгүй байна.

### Negative утгатай model

transfer-neg.model нь ERROR сонголтуудын өмнө
~ тэмдэглэгээ ашигласан.

Нийт 16 тестийн мөр үүссэн.
Үүнээс 2 нь happy path байсан.

Нэг мөрөнд хамгийн ихдээ нэг алдаатай
утга орсон бөгөөд олон алдаатай мөр байхгүй.

Ингэснээр error masking-ийн эрсдэл багассан.

### Negative утга ба IF хязгаарлалтын ялгаа

~ тэмдэглэгээ нь нэг мөрөнд олон negative
сонголт орохоос сэргийлдэг.

Гэхдээ ~ нь бизнесийн бүх хамаарлыг
автоматаар тодорхойлохгүй.

Жишээлбэл SameAsFrom нь From данстай
ижил байх нөхцөлтэй бөгөөд үүнийг
зөвхөн ~ тэмдэглэгээгээр бүрэн илэрхийлэхгүй.

Иймээс бизнесийн нөхцөлүүдийг шалгахын тулд
зарим IF хязгаарлалт хэрэгтэй хэвээр байна.

### Happy path-ийн шалгалт

transfer-nc.model: 1 happy path.

transfer.model: 0 happy path.

transfer-neg.model: 2 happy path.

Хүчинтэй From, хүчинтэй To болон Normal Amount
гэсэн гурван нөхцөл зэрэг биелсэн мөрийг
happy path гэж тодорхойлсон.

PICT нь pairwise covering array үүсгэдэг ч
happy path заавал оруулна гэсэн баталгаа байхгүй.

### Алдааны давуу эрэмбэ

Олон алдаа зэрэг илэрвэл A хэсэгт тогтоосон
From → To → Amount дарааллыг ашиглана.

Энэ нь тест дизайны таамагласан эрэмбэ юм.

Жишээ нь:

From = Missing
To = Missing
Amount = Normal

гэсэн мөрийн oracle:

HTTP 200
{"result":"ERROR_NO_FROM"}

### Нотолгооны файлууд

- results/transfer-nc.txt
- results/transfer.txt
- results/transfer-neg.txt

## B2 — PICT гаралтаас боловсруулсан бодит тест кейсүүд

Тестүүдийг `results/transfer-neg.txt` файлаас сонгосон.
Мөрийн дугаарт толгойн мөрийг тооцоогүй.

### TC-B01 — Happy path

**PICT мөр:** 1

| From | To | Amount | Currency |
|---|---|---|---|
| Valid | Valid | Normal | Same |

**Setup:**
- ACC1001: Active, MNT, үлдэгдэл 500000 MNT.
- Өдрийн үлдсэн лимит: 300000 MNT.
- ACC2001: хүчинтэй, өөр данс.

**Input:**

```json
{
  "from": "ACC1001",
  "to": "ACC2001",
  "amount": 100000,
  "currency": "MNT"
}
```

**Oracle:**

HTTP 201

```json
{"result":"OK"}
```

Үндэслэл: Дүн нь эерэг, үлдэгдэл болон өдрийн
лимитийн хүрээнд байгаа учраас шилжүүлэг амжилттай.

### TC-B02 — Missing From

**PICT мөр:** 8

| From | To | Amount | Currency |
|---|---|---|---|
| ~Missing | Valid | Normal | Same |

**Setup:**
- ACC9999: системд байхгүй.
- ACC2001: хүчинтэй данс.
- Шилжүүлэх дүн: 100000 MNT.

**Input:**

```json
{
  "from": "ACC9999",
  "to": "ACC2001",
  "amount": 100000,
  "currency": "MNT"
}
```

**Oracle:**

HTTP 200

```json
{"result":"ERROR_NO_FROM"}
```

Үндэслэл: Эх данс байхгүй учраас
ERROR_NO_FROM алдаа буцна.

### TC-B03 — OverLimit

**PICT мөр:** 16

| From | To | Amount | Currency |
|---|---|---|---|
| Valid | Valid | ~OverLimit | Same |

**Setup:**
- ACC1001: Active, үлдэгдэл 500000 MNT.
- Өдрийн үлдсэн лимит: 300000 MNT.
- ACC2001: хүчинтэй, өөр данс.

**Input:**

```json
{
  "from": "ACC1001",
  "to": "ACC2001",
  "amount": 400000,
  "currency": "MNT"
}
```

**Oracle:**

HTTP 200

```json
{"result":"ERROR_LIMIT"}
```

Үндэслэл: 400000 MNT нь 500000 MNT
үлдэгдэлд багтсан боловч 300000 MNT
өдрийн лимитээс хэтэрсэн.

### Тест кейсүүдийн нэгтгэл

| ID | PICT мөр | HTTP | Result |
|---|---:|---:|---|
| TC-B01 | 1 | 201 | OK |
| TC-B02 | 8 | 200 | ERROR_NO_FROM |
| TC-B03 | 16 | 200 | ERROR_LIMIT |

## B3 — Category-Partition ба PICT-ийн харьцуулалт

| Үзүүлэлт | Category-Partition (A3) | PICT |
|---|---|---|
| Тест боловсруулах | Гараар | Автоматаар |
| Бизнесийн дүрэм | Дэлгэрэнгүй тодорхойлно | Constraint шаарддаг |
| Алдааны семантик | Илүү ойлгомжтой | Oracle тусад нь шаарддаг |
| Хосын хамралт | Баталгаатай биш | Pairwise хамралттай |
| Error masking | Гараар хянадаг | Negative (~) утгаар багасгана |
| Давуу тал | Expected result тодорхой | Олон комбинацийг хамардаг |
| Сул тал | Цаг их шаарддаг | Greedy тул minimum баталгаагүй |

### Дүгнэлт

1. Энэ лабораторийн ажлаар Category-Partition болон PICT аргыг ашиглан банкны шилжүүлгийн тестийн дизайн боловсруулсан.
2. Category-Partition арга нь бизнесийн дүрэм болон алдааны ангиллыг тодорхойлох боломж олгосон.
3. Гараар боловсруулсан A3 хүснэгт нь 12 тестийн спецификаци болон тэдгээрийн хүлээгдэх үр дүнг тодорхой харуулсан.
4. PICT нь параметрүүдийн хосын хамралтыг автоматаар хангах тестийн багц үүсгэсэн.
5. Browser model дээр 2-way тестээр 10 мөр, 3-way тестээр 20 мөр үүссэн.
6. Transfer model-ийн хязгаарлалтгүй хувилбар 13, IF хязгаарлалттай хувилбар 12 тестийн мөр үүсгэсэн.
7. Negative тэмдэглэгээтэй загвар 16 тест үүсгэж, олон алдаатай мөрийг арилгасан.
8. Хамгийн их бодол шаардсан хэсэг нь алдааны давуу эрэмбэ, IF нөхцөл болон бодит тестийн oracle-г тодорхойлох байсан.
9. Иймээс Category-Partition аргаар алдааны семантикийг тодорхойлж, PICT-ээр хосын хамралтыг хангах нь хоёр аргын давуу талыг хослуулсан шийдэл юм.

