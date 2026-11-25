```markdown
# Лаборатори №2: Гүйцэтгэлийн хэмжүүр (K6)
Программ хангамжийн чанарын баталгаа ба туршилт (F.CSA313)

**B222270823 А. Наранзул**

---

## Орчин

- OS: Windows 11
- k6 version:
```text
k6 v0.54.0 (windows/amd64)
```
- Тестийн зорилтот сервер: `https://test.k6.io`

## Алхам 1. Baseline (5 VU, 30s)

| Хэмжигдэхүүн | Утга |
|---|---|
| avg | 778.37ms |
| p90 | 1.77s |
| p95 | 2.43s |
| throughput (http_reqs) | 100 нийт хүсэлт, 3.05/s |
| error rate (http_req_failed) | 0.00% (0/100) |

Бага ачаалал буюу 5 VU, 30 секундийн хугацаанд `baseline.js` скриптийг ажиллуулж анхны жишиг үзүүлэлтүүдийг `results/baseline.txt` файлд хадгалсан. Дэлгэцийн зургаар `screenshots/run_baseline.png` хавсаргав. Энэхүү `p95` утгыг Алхам 4-ийн SLO босго тодорхойлоход суурь болгон ашигласан.

## Алхам 2. Ачааллыг шатлан өсгөх (5 / 30 / 100 VU, тус бүрд 1 минут)

| VU | p90 | p95 | Throughput | Error rate |
|---|---|---|---|---|
| 5 | 1.41s | 1.80s | 4.00/s (250 нийт) | 0.00% |
| 30 | 1.76s | 2.55s | 16.36/s (1236 нийт) | 0.00% |
| 100 | 2.25s | 3.26s | 39.80/s (3582 нийт) | 0.22% (8/3582) |

**Хавсаргасан текст файл болон зураг:**
- `screenshots/run_5vu.png`, `results/run-05vu.txt`
- `screenshots/run_30vu.png`, `results/run-30vu.txt`
- `screenshots/run_100vu.png`, `results/run-100vu.txt`
- `screenshots/run_stages.png`, `results/stages.txt` — stages (`5 → 30 → 100 → 0` VU) ашигласан үр дүн

`stages`-тэй хувилбар (`stages.js`, 5→30→100→0 VU) нь ачаалал аажмаар өсөх болон буурах үеийн системийн ерөнхий төлөвийг ажиглах зорилгоор хийгдсэн бөгөөд дээрх хүснэгтийн нарийвчилсан тоон утгуудыг тус тусад нь ажиллуулсан гаралтын файлуудаас авсан.

## Алхам 3 — Threshold (SLO)

**SLO тооцоо:** Системийн хэвийн ачааллын үеийн хариу өгөх хугацаанаас хэт удаашрахаас сэргийлж `p(95) < 1500ms` болон алдааны хувийг 1%-иас бага байлгах (`rate < 0.01`) зорилтот босго (SLO) тогтоов.

```javascript
thresholds: {
  http_req_duration: ['p(95)<1500'],
  http_req_failed:   ['rate<0.01'],
},
```

| Тест | Threshold | Бодит утга | Үр дүн |
|---|---|---|---|
| PASS (30 VU, 1m) | p(95)<1500ms | 1.34s | ✓ PASS |
| PASS | rate<0.01 | 0.00% | ✓ PASS |
| FAIL (30 VU, 1m) | p(95)<100ms | 372.53ms | ✗ FAIL |
| FAIL | rate<0.01 | 0.00% | ✓ PASS |

**Хавсаргасан файл болон зураг:**
- `results/threshold-pass.txt`, `screenshots/threshold_passed.png`
- `results/threshold-fail.txt`, `screenshots/threshold_failed.png`

FAIL тестийн үед k6 нь `ERRO[0061] thresholds on metrics 'http_req_duration' have been crossed` гэсэн алдаа заан non-zero exit code буцаасан нь CI/CD pipeline дээр Quality Gate ажиллаж, гүйцэтгэлийн шаардлага хангаагүй кодыг автоматаар зогсоох зарчимтай бүрэн нийцэж байна.

## Дүгнэлт

Энэхүү лабораторийн ажлаар k6 хэрэгслийг ашиглан системд зэрэгцээ хэрэглэгчийн ачаалал өгөхөд гүйцэтгэлийн үндсэн хэмжүүрүүд хэрхэн өөрчлөгддөгийг туршиж үзлээ. Туршилтын үр дүнгээс харахад зэрэгцээ хэрэглэгчийн тоо (VU) 5-аас 100 болж өсөхөд `p95` latency нэмэгдэж, нэг хэрэглэгчид ирэх хариуны хугацаа уртассан бол `throughput` буюу секундэд боловсруулах нийт хүсэлтийн тоо ачааллыг дагаж өссөн байна. Энэ нь систем илүү олон хүсэлт зэрэг боловсруулах чадвартай ч ачаалал ихсэх тусам хүлээлгийн хугацаа сунадаг болохыг харуулж байна. Baseline хэмжилтэд тулгуурлан тодорхойлсон `p(95) < 1500ms` босго утга нь 30 VU ачааллын үед бүрэн биелж PASS үр дүн үзүүлсэн бол босгыг зориудаар `p(95) < 100ms` болгож чангаруулахад k6 non-zero exit code буцааж CI/CD орчинд Quality Gate хэрхэн ажилладгийг бодитоор баталлаа. Дүгнэж хэлэхэд, зөвхөн дундаж (`avg`) үзүүлэлт бус харин `p90`, `p95` хувийн жингийн хэмжүүрүүдийг ашиглан SLO тохируулах нь бодит хэрэглэгчдийн туршлагыг үнэлэх оновчтой арга болохыг ойлгож авлаа.



## Reproduce хийх

```powershell
# 1. Baseline тест (5 VU, 30s)
k6 run baseline.js | tee results/baseline.txt

# 2. Ачааллыг шатлан өсгөх тестүүд (5 / 30 / 100 VU, 1m)
k6 run --vus 5 --duration 1m script.js | tee results/run-05vu.txt
k6 run --vus 30 --duration 1m script.js | tee results/run-30vu.txt
k6 run --vus 100 --duration 1m script.js | tee results/run-100vu.txt

# 3. Threshold PASS / FAIL шалгах (30 VU, 1m)
k6 run --vus 30 --duration 1m script.js | tee results/threshold-pass.txt
k6 run --vus 30 --duration 1m script.js | tee results/threshold-fail.txt

# 4. Stages туршилт (5 -> 30 -> 100 -> 0 VU)
k6 run stages.js | tee results/stages.txt
```
```
