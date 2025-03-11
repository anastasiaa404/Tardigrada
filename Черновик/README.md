# Шаг 1.
- У нас тут работа немножко data-driven, целей может быть много разных.
- Есть комплекс "паразит-хозяин", в нем неизвестная Tardigrada (предположительно Murrayidae) и облигатный внутриклеточный паразит, кто-то из Apicomplexa.
- Хочется научиться разделять их геномы и определять систематическое положение методами филогеномики

## 1. Скачали все и риды:
- /media/eternus1/nfs/projects/users/mrayko/microsporidia_reads/jan_25/2024-12-25-DNBSEQ-PE300-Nasonova
```
AL5_24_fastqc_out                           F350035857_L01_CAP1-1-EN41_2.fq.gz
DNBSEQ-G400_PE300_2024-12-25_Nasonova.xlsx  F350035857_L01_CAP1-2-EN42_1.fq.gz
F350035857_L01_AL15.24-5-EN272_1.fq.gz      F350035857_L01_CAP1-2-EN42_2.fq.gz
F350035857_L01_AL15.24-5-EN272_2.fq.gz      F350035857_L01_OOM3-4-EN47_1.fq.gz
F350035857_L01_API1-4-EN43_1.fq.gz          F350035857_L01_OOM3-4-EN47_2.fq.gz
F350035857_L01_API1-4-EN43_2.fq.gz          F350035857_L01_OOM3-6-EN48_1.fq.gz
F350035857_L01_API1-6-EN44_1.fq.gz          F350035857_L01_OOM3-6-EN48_2.fq.gz
F350035857_L01_API1-6-EN44_2.fq.gz          F350035857_L02_AL5.24_2-EN270_1.fq.gz
F350035857_L01_API2-6-EN45_1.fq.gz          F350035857_L02_AL5.24_2-EN270_2.fq.gz
F350035857_L01_API2-6-EN45_2.fq.gz          F350035857_L02_AL8.24_3-EN271_1.fq.gz
F350035857_L01_API2-8-EN46_1.fq.gz          F350035857_L02_AL8.24_3-EN271_2.fq.gz
F350035857_L01_API2-8-EN46_2.fq.gz          F350035857_L02_NKA_1-EN269_1.fq.gz
F350035857_L01_CAP1-1-EN41_1.fq.gz          F350035857_L02_NKA_1-EN269_2.fq.gz
```
## 2. Запускаем Кракена на ридах, 
начнем пока только с OOM3 (один образец, две реакции MDA), либо с API1 и API2 (два образца, в каждом по две реакции).
```
kraken2 --db /media/eternus1/nfs/projects/databases/kraken2_db/ --threads 8 --paired ~/tardigrada/F350035857_L01_OOM3-6-EN48_1.fq.gz ~/tardigrada/F350035857_L01_OOM3-6-EN48_2.fq.gz --report ~/tardigrada/OOM3-6-EN48_report.txt --output ~/tardigrada/OOM3-6-EN48_classified.txt
```
- получаем: 
   ![Снимок экрана 2025-02-26 234304](https://github.com/user-attachments/assets/570cfc3d-7f68-4b34-a885-3220e78a9969)
(база данных кракена на сервер: /media/eternus1/nfs/projects/databases/kraken2_db/ )
## 3. Дальнейший план действий:
    1. Сделать сборку
    2. Сделать биннинг
    3. Посмотреть, какой из бинов какому организму соответствует
    4. Вытащить из них набор ортологичных генов
    5. Построить филогеномное дерево, понять, кто у нас вообще тут живёт
  
- Сейчас актуален шаг 1, это нужно просто запустить сборку с помощью spades.
- Собирать за один раз один образец - если там две реакции, то указать, что это две библиотеки, т.е. 4 файла с ридами на вход.
## 4. Запускаем spades на 2х парах ридов OOM:
- не забываем работать в окружении спейдес, где и установили его:
```
source ~/.bashrc
conda activate spades-env
```
```
spades.py   -1 F350035857_L01_OOM3-6-EN48_1.fq.gz -2 F350035857_L01_OOM3-6-EN48_2.fq.gz   --pe1-1 F350035857_L01_OOM3-4-EN47_1.fq.gz --pe1-2 F350035857_L01_OOM3-4-EN47_2.fq.gz   -o spades_output   -t 16
```
- Тут у меня прервался процесс и вспомнила про существование nohup)))
```
 nohup spades.py   -1 F350035857_L01_OOM3-6-EN48_1.fq.gz -2 F350035857_L01_OOM3-6-EN48_2.fq.gz   --pe1-1 F350035857_L01_OOM3-4-EN47_1.fq.gz --pe1-2 F350035857_L01_OOM3-4-EN47_2.fq.gz   -o spades_output   -t 40 > spades.log 2>&1 &
```

- Запускаем спейдес, на следующих образца: API1 и API2, используем screen/
```
mkdir -p /media/eternus1/nfs/projects/users/aanferova/tardigrada/spades_API1

screen -S spades_api1

spades.py \
-1 /media/eternus1/nfs/projects/users/mrayko/microsporidia_reads/jan_25/2024-12-25-DNBSEQ-PE300-Nasonova/F350035857_L01_API1-4-EN43_1.fq.gz \
-2 /media/eternus1/nfs/projects/users/mrayko/microsporidia_reads/jan_25/2024-12-25-DNBSEQ-PE300-Nasonova/F350035857_L01_API1-4-EN43_2.fq.gz \
--pe1-1 /media/eternus1/nfs/projects/users/mrayko/microsporidia_reads/jan_25/2024-12-25-DNBSEQ-PE300-Nasonova/F350035857_L01_API1-6-EN44_1.fq.gz \
--pe1-2 /media/eternus1/nfs/projects/users/mrayko/microsporidia_reads/jan_25/2024-12-25-DNBSEQ-PE300-Nasonova/F350035857_L01_API1-6-EN44_2.fq.gz \
-o /media/eternus1/nfs/projects/users/aanferova/tardigrada/spades_API1 \
-t 16
```
- API2:
```
mkdir -p /media/eternus1/nfs/projects/users/aanferova/tardigrada/spades_API2

screen -S spades_api2

spades.py \
-1 /media/eternus1/nfs/projects/users/mrayko/microsporidia_reads/jan_25/2024-12-25-DNBSEQ-PE300-Nasonova/F350035857_L01_API2-6-EN45_1.fq.gz \
-2 /media/eternus1/nfs/projects/users/mrayko/microsporidia_reads/jan_25/2024-12-25-DNBSEQ-PE300-Nasonova/F350035857_L01_API2-6-EN45_2.fq.gz \
--pe1-1 /media/eternus1/nfs/projects/users/mrayko/microsporidia_reads/jan_25/2024-12-25-DNBSEQ-PE300-Nasonova/F350035857_L01_API2-8-EN46_1.fq.gz \
--pe1-2 /media/eternus1/nfs/projects/users/mrayko/microsporidia_reads/jan_25/2024-12-25-DNBSEQ-PE300-Nasonova/F350035857_L01_API2-8-EN46_2.fq.gz \
-o /media/eternus1/nfs/projects/users/aanferova/tardigrada/spades_API2 \
-t 16
```
## 4.2 barrnap
- В результатах сборки достать 16S:

```
barrnap  scaffolds.fasta   --outseq rrna.fasta
```

## 5. QUAST:
```
quast.py -o quast_output contigs.fasta
```
- Результаты QUAST:
 ![Снимок экрана 2025-03-06 172814](https://github.com/user-attachments/assets/3701732a-d9a8-4d9a-a750-a99dc1f04e81)

### Оценка качества сборки

1. **Полнота сборки**:
   - Общая длина сборки (≈ 103 Мб) и N50 (70 kbp) указывают на то, что сборка достаточно полная, особенно если ты работаешь с геномом эукариот (например, тихоходки).
   - Однако наличие большого количества мелких контигов (20173 контигов ≥ 0 bp) говорит о некоторой фрагментированности.

2. **Качество сборки**:
   - N50 в 70 kbp — это хороший результат, но его можно улучшить, особенно если ты работаешь с крупным геномом.
   - Самый длинный контиг (787 kbp) — это отличный показатель, который говорит о том, что SPAdes смог собрать крупные участки генома.

3. **Фрагментированность**:
   - Наличие 20173 контигов указывает на то, что сборка может быть фрагментированной. Это может быть связано с:
     - Ограничениями данных (например, короткие риды).
     - Сложностью генома (например, повторяющиеся участки).

## 6.BUSCO:

