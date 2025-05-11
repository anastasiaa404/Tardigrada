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

## 2.2. Визиализация Кракена
- скачиваем результаты и смотрим их на сайте: https://fbreitwieser.shinyapps.io/pavian/
```
scp -i ~/.ssh/id_rsa aanferova@77.234.216.99:/media/eternus1/nfs/projects/users/aanferova/tardigrada/AL15.24-5-EN272_report.txt /home/anastassia/itmo/tardigrada/AL15.24-5-EN272_report.txt
```
- ![Снимок экрана 2025-03-11 164512](https://github.com/user-attachments/assets/b4001ff5-7b0b-4ade-bd00-1ef4617e1ac6)
- ![Снимок экрана 2025-03-11 164715](https://github.com/user-attachments/assets/ea696819-d5e8-45a0-a2a6-675331dbb74d) ![Снимок экрана 2025-03-11 164749](https://github.com/user-attachments/assets/2e38a464-1e12-481d-854e-7ae07170a780)



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
### 4.1.2. Запускаем trimmidat и перезапускаем спейдес:
- Меняли только пути 
```
   spades.py \
-1 /media/eternus1/nfs/projects/users/mrayko/microsporidia_reads/jan_25/2024-12-25-DNBSEQ-PE300-Nasonova/trimmed/OOM3-4-EN47/paired_1.fq.gz \
-2 /media/eternus1/nfs/projects/users/mrayko/microsporidia_reads/jan_25/2024-12-25-DNBSEQ-PE300-Nasonova/trimmed/OOM3-4-EN47/paired_2.fq.gz \
--pe1-s /media/eternus1/nfs/projects/users/mrayko/microsporidia_reads/jan_25/2024-12-25-DNBSEQ-PE300-Nasonova/trimmed/OOM3-4-EN47/unpaired_1.fq.gz \
--pe1-s /media/eternus1/nfs/projects/users/mrayko/microsporidia_reads/jan_25/2024-12-25-DNBSEQ-PE300-Nasonova/trimmed/OOM3-4-EN47/unpaired_2.fq.gz \
--pe2-1 /media/eternus1/nfs/projects/users/mrayko/microsporidia_reads/jan_25/2024-12-25-DNBSEQ-PE300-Nasonova/trimmed/OOM3-6-EN48/paired_1.fq.gz \
--pe2-2 /media/eternus1/nfs/projects/users/mrayko/microsporidia_reads/jan_25/2024-12-25-DNBSEQ-PE300-Nasonova/trimmed/OOM3-6-EN48/paired_2.fq.gz \
--pe2-s /media/eternus1/nfs/projects/users/mrayko/microsporidia_reads/jan_25/2024-12-25-DNBSEQ-PE300-Nasonova/trimmed/OOM3-6-EN48/unpaired_1.fq.gz \
--pe2-s /media/eternus1/nfs/projects/users/mrayko/microsporidia_reads/jan_25/2024-12-25-DNBSEQ-PE300-Nasonova/trimmed/OOM3-6-EN48/unpaired_2.fq.gz \
-o /mnt/projects/aanferova/tardigrada/spades_OOM \
-t 20
```
## 4.2 barrnap
- В результатах сборки достать 16S:

```
barrnap  scaffolds.fasta   --outseq rrna.fasta
```

## 5.Запускали срипт Миши, буско и биннинг (maxbin)

## 6. Строим деревья:
1. Что конкретно надо сделать пошагово:
2. Пробежать barrnap по всем файлам maxbin_real.00*.fasta, чтобы вытащить 18S рРНК.
3.  Посмотреть, что нашлось — может быть, у некоторых бинов будет 18S, у некоторых нет.
4. Собрать найденные 18S в один fasta-файл.
5. Сделать BLAST этих последовательностей против базы nt или silva, чтобы понять, кто это.
6. (Опционально) Построить филогенетическое дерево через IQ-TREE.

### 6.1. Достаем 18S:
```
# Создадим папку для результатов
mkdir barrnap_results

# Для каждого бина запустить barrnap
for bin in maxbin_real.00*.fasta; do
    barrnap --kingdom euk --outseq barrnap_results/${bin%.fasta}_18S.fasta $bin > barrnap_results/${bin%.fasta}_18S.gff
done

cat barrnap_results/*_18S.fasta > all_bins_18S.fasta
```
- В файле все рРНК, поэтому вырежем, чтобы оставить только 18S:
```
# Создадим отдельный файл только с 18S
grep -A 1 "18S" /media/eternus1/nfs/projects/users/aanferova/tardigrada/all_bins_18S.fasta | grep -v "^--$" > /media/eternus1/nfs/projects/users/aanferova/tardigrada/all_bins_only18S.fasta
```
### 6.2 BLAST:
- используем BLAST NCBI Nucleutide, загружали по отдельности каждый бин и скачивали по 3 ближайших последовательности к нашему бину

- **Короткий анализ:**
Последовательность | Ближайший родич | Комментарий
NODE_746 | (очень высокое % id) | Скорее всего тихоходка 
NODE_1780 | (тоже высокое % id) | Возможно тихоходка 
NODE_1778 | (чуть ниже %) | Может быть паразит (Apicomplexa) 
NODE_4348 | (очень хорошее %) | Похоже на тихоходку 
NODE_2328 | (низкое покрытие) | Возможно паразит 
NODE_1167 | (высокий % id) | Скорее всего тихоходка 
NODE_3395 | (высокий % id) | Скорее всего тихоходка
- это только для реакции OOM а потом еще для API1 и API2

### 6.3: Дерево
- использовали для постоения IQTREE, и для визуализации ITol
   ![KViCej-MjqXxauJRhtByZg-_1_](https://github.com/user-attachments/assets/93818ae8-c33a-483c-81b6-d2b37a4de33c)

Обозначения цветом:
- красный: бактериальные геномы
- синий:  микроспоридии (паразитические грибоподобные организмы)
- желтый: грибы (в том числе Oomycota) 
- фиолетовый: представители Apicomplexa
- зеленый: последовательности Tardigrada

#### 6.3.2. Дерево по ортологам
- запустить буско на всех бинах
- начнем с **аминокислотных (белковых) последовательностей**
- копируем все ортологи в одну папку:
```
mkdir -p /mnt/projects/aanferova/tardigrada/tardigrada_busco_aa

# Копируем аминокислотные последовательности из всех BUSCO-папок в целевую директорию
for dir in /media/eternus1/nfs/projects/users/mrayko/microsporidia/OOM/maxbin2/busco_metaeuk_maxbin_real.00*_eukaryota; do
    if compgen -G "$dir/run_eukaryota_odb10/busco_sequences/single_copy_busco_sequences/*.faa" > /dev/null; then
        cp "$dir"/run_eukaryota_odb10/busco_sequences/single_copy_busco_sequences/*.faa /mnt/projects/aanferova/tardigrada/tardigrada_busco_aa/
    else
        echo "Нет .faa файлов в: $dir"
    fi
done
```
- Шаг 1: Создать подкаталоги по каждому BUSCO ID
Это поможет нам собрать все последовательности одного и того же ортолога вместе:
```
cd /mnt/projects/aanferova/tardigrada/tardigrada_busco_aa

# Создать директории для каждого уникального BUSCO ID
for f in *.faa; do
    id="${f%%.faa}"
    mkdir -p "orthologs/$id"
    cp "$f" "orthologs/$id/"
done
```
Шаг 2: Объединить все последовательности по каждому ортологу
```
cd orthologs

for dir in */; do
    cat "$dir"/*.faa > "${dir%/}.fasta"
done
```
Шаг 3: Перенести все .fasta-файлы в отдельную директорию для выравнивания
```
mkdir ../orthologs_ready
mv *.fasta ../orthologs_ready/
```
- 
| Шаг             | Инструмент            |
| --------------- | --------------------- |
| Поиск ортологов | BUSCO                 |
| Выравнивание    | MAFFT                 |
| Конкатенация    | AMAS / FASconCAT-G    |
| Филогения       | IQ-TREE               |
