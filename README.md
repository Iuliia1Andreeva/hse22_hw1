# Сборка генома #
Сначала подключимся к серверу, используя команду
```
ssh --- -p 32222 -i mykey
```
Затем ищем исходные необходимые файлы, используя команду
```
cd /usr/share/data-minor-bioinf/assembly/
```
Теперь создадим ссылки на файлы, чтобы работать с ними было удобнее
```
cd
ln -s /usr/share/data-minor-bioinf/assembly/oil_R2.fastq
ln -s /usr/share/data-minor-bioinf/assembly/oilMP_S4_L001_R1_001.fastq
ln -s /usr/share/data-minor-bioinf/assembly/oilMP_S4_L001_R2_001.fastq
ln -s /usr/share/data-minor-bioinf/assembly/oil_R1.fastq
```
Теперь учтем, что SEED=112 и начнем скачивать кусочки файлов.
```
seqtk sample -s112 oil_R1.fastq 5000000 > sub1.fastq
seqtk sample -s112 oil_R2.fastq 5000000 > sub2.fastq
seqtk sample -s112 oilMP_S4_L001_R1_001.fastq 1500000 > matep1.fastq
seqtk sample -s112 oilMP_S4_L001_R2_001.fastq 1500000 > matep2.fastq
```
Затем будем создавать отчет для fastQS
```
mkdir fastqc
ls sub* matep* | xargs -tI{} fastqc -o fastqc {}
```
Аналогично для **MultiQC**
```
mkdir multiqc
multiqc -o multiqc fastqc
```
Обрежем чтения
```
platanus_trim sub*
platanus_internal_trim matep*
```
И удалим изначальные файлы
```
rm sub1.fastq
rm sub2.fastq
rm matep1.fastq
rm matep2.fastq
```
А теперь сделаем отчет в MultiQS с обрезанными данными

```
cd multiqs
mkdir multiqc
multiqc -o multiqc_trimmed fastqc_trimmed
```
Теперь вернемся в локальный терминал и скачаем репорт для обрезанных чтений

```
scp -P 32222 -i mykey yuvandreeva_1@92.242.58.92:"/home/yuvandreeva_1/multiqc_trimmed/multiqc_report.html" /Users/uliaandreeva
```
Теперь соберем контиги 
```
time platanus assemble -o Poil -f sub1.fastq.trimmed sub2.fastq.trimmed 2> assemble.log
```

Теперь соберем скаффолды

```
time platanus scaffold -o Poil -c Poil_contig.fa -IP1 sub1.fastq.trimmed sub2.fastq.trimmed -OP2 matep1.fastq.int_trimmed matep2.fastq.int_trimmed 2> scaffold.log
```
Уменьшим число промежутков

```
time platanus gap_close -o Poil -c Poil_scaffold.fa -IP1 sub1.fastq.trimmed sub2.fastq.trimmed -OP2 matep1.fastq.int_trimmed matep2.fastq.int_trimmed 2> gapclose.log
```
И посчитаем gaps
```

time platanus gap_close -o Poil -c Poil_scaffold.fa -IP1 sub1.fastq.trimmed sub2.fastq.trimmed -OP2 matep1.fastq.int_trimmed matep2.fastq.int_trimmed 2> gapclose.log
```
**ИТОГ**
В файлах Poil_contig.fa, Poil_scaffold.fa, Poil_gapClosed.fa лежат файлы, которые нам предстоит анализировать в следующем пункте