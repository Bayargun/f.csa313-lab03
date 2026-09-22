# Lab-03 - Quality Scenario, SLO , k6 Threshold

Нэр : Б.Баяргүн
Оюутны код:B222270085

## k6 version

k6 v2.2.0 (commit/00a9a1b7f5, go1.26.5, linux/amd64)

## PASS тестийн үр дүн

20 VU, 1 минутын ачааллын тестээр дараах үр дүн гарсан.

| Үзүүлэлт            | SLO          | Бодит үр дүн    | Төлөв |
| ------------------- | ------------ | --------------- | ----- |
| `/cart/add` latency | p95 < 2 ms   | p95 = 1 ms      | PASS  |
| `/report` latency   | p95 < 450 ms | p95 = 388.93 ms | PASS  |
| `/pay` error rate   | < 8%         | 3.54%           | PASS  |
| Checks              | > 90%        | 98.81%          | PASS  |

`/cart/add` endpoint-ийн эхний хэмжилтээр p95 = 1.11 ms орчим
гарсан тул хэт сул босго сонгохоос зайлсхийж p95 < 2 ms гэсэн
Performance SLO сонгосон. Эцсийн PASS туршилтаар p95 = 1 ms
гарсан бөгөөд босгыг хангасан.

/report endpoint нь зориудаар 200–400 ms сааталтай тул
p95 < 450 ms босго сонгосон. Туршилтаар p95 = 388.93 ms
гарч босгыг хангасан.

/pay endpoint нь ойролцоогоор 5% санамсаргүй алдаа үүсгэхээр
хийгдсэн тул error rate < 8% босго сонгосон. Туршилтаар
error rate = 3.54% гарсан бөгөөд Reliability SLO-г хангасан.

## Chaos test

Chaos туршилтыг 20 VU, 2 минутын хугацаатай ажиллуулсан.
Туршилтын үед серверийг зориудаар 10 секунд зогсоож дараа нь
дахин асаасан.

| Үзүүлэлт                | SLO          | Бодит үр дүн | Төлөв |
| ----------------------- | ------------ | ------------ | ----- |
| `/cart/add` latency     | p95 < 2 ms   | 1.04 ms      | PASS  |
| `/report` latency       | p95 < 450 ms | 389.77 ms    | PASS  |
| Availability (`checks`) | > 90%        | 78.23%       | FAIL  |
| `/pay` error rate       | < 8%         | 24.48%       | FAIL  |

Нийт 5820 check-ээс 4553 нь амжилттай, 1267 нь амжилтгүй
болсон. Иймээс request-based availability:

Availability = 4553 / 5820 × 100 = 78.23%

Availability SLO нь 90%-иас дээш байх ёстой байсан тул chaos
туршилтын үед availability threshold FAIL болсон.

2 минут цонхонд 90% availability SLO-ийн
time-based error budget:

120 × (1 - 0.90) = 12 секунд.

Серверийг ойролцоогоор 10 секунд зогсоосон нь time-based
12 секундын error budget-аас бага боловч request-based
availability 78.23% болсон. Учир нь сервер унтарсан үед
хүсэлтүүд connection refused алдаатайгаар маш хурдан буцдаг.
Иймээс богино хугацаанд олон failed request үүсэж,
request-based availability нь time-based availability-аас
доогуур гарсан.

Мөн /pay endpoint-ийн error rate 24.48% болж, 8%-ийн
Reliability SLO-г зөрчсөн. Сервер унтарсан үед /pay хүсэлтүүд
мөн амжилтгүй болсон учраас нэг серверийн эвдрэл availability
болон reliability SLO-д хоёуланд нь нөлөөлсөн.

### FAIL тестийн үр дүн

`/report` endpoint-ийн threshold-ийг зориуд `p(95) < 100 ms`
болгож туршсан.

- `/cart/add`: p95 = 943.91 µs — PASS
- `/report`: p95 = 391.11 ms — FAIL
- `/pay` error rate = 4.09% — PASS
- checks = 98.63% — PASS

`/report` endpoint нь 200–400 ms сааталтай учраас`p(95) < 100 ms` босгыг хангаж чадаагүй. k6 нь
`http_req_duration{name:report}` threshold зөрчигдсөнийг илрүүлсэн.
