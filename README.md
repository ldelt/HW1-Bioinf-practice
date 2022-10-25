# Лабораторный журнал
Вывод для всех команд, подразумевающих какой-либо вывод, сохранен в папку `output`, находящуюся в рабочей папке репозитория. 
## Подготовка

1. **Создаем рабочую директорию и переходим в нее**
```mkdir HW1```

2. **Внутри рабочей директории создаем папку, содержащую исходные файлы**
```cd ./HW1```
```mkdir raw```

3. **Загружаем референсный геном (*_genomic.fna.gz) и его аннотацию (*_genomic.gff.gz) по ссылке:**
**[https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/005/845/GCF_000005845.2_ASM584v2/](https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/005/845/GCF_000005845.2_ASM584v2/)**

4. **Загружаем риды штамма, резистентного к антибиотику, по ссылке:**
**[https://doi.org/10.6084/m9.figshare.10006541.v3](https://doi.org/10.6084/m9.figshare.10006541.v3)**

*Файлы загружаем и переносим в папку raw вручную без использования терминала*

5. **Распаковываем архивы с ридами, референсом и аннотацией:**
```cd ./raw```
```unzip 10006541.zip```
```gunzip  GCF_000005845.2_ASM584v2_genomic.fna.gz```
```gunzip  GCF_000005845.2_ASM584v2_genomic.gff.gz```

## Ознакомление с данными

1. **Проверяем корректность формата данных при помощи `head`:**
	- ```cd ..```
	- ```zless  ./raw/amp_res_1.fastq.gz | head -20 > head20_amp_res_1.txt```
	- ```zless  ./raw/amp_res_2.fastq.gz | head -20 > head20_amp_res_2.txt```
	- ```head -20 ./raw/GCF_000005845.2_ASM584v2_genomic.fna > ./output/head20_reference_fna.txt```
	- ```head -20 ./raw/GCF_000005845.2_ASM584v2_genomic.gff > ./output/head20_reference_gff.txt```

2. **Используем команду `cat`, чтобы вывести весь файл с референсом:**
	- ```cat  ./raw/GCF_000005845.2_ASM584v2_genomic.fna > ./output/cat_reference_fna.txt```
	
	Файл референса содержит лишь строку с индентификатором/названием и саму последовательность.
	
3. **Считаем количество ридов в файлах с прямыми и обратными ридами:** 
	- ```zless  ./raw/amp_res_1.fastq.gz | wc -l > ./output/wc_amp_res_1.txt```
	-  ```zless  ./raw/amp_res_1.fastq.gz | wc -l > ./output/wc_amp_res_1.txt```

	Количество строк в обоих файлах совпадает и равняется **1823504**. На каждый рид в файлах формата fastq отводится 4 строки - следовательно, в файлах по **455876** ридов.

## Проверка качества

1. **Проверка качества исходных ридов при помощи fastqc:**

	- ```fastqc -o .  ./raw/amp_res_1.fastq.gz  ./raw/amp_res_2.fastq.gz &> ./output/fastqc_amp_res.txt```

	Количество ридов совпадает с рассчитанным ранее при помощи `wc -l`.
	**Для прямых ридов.** Красным отмечены пункты `Per base sequence quality` и `Per tile sequence quality`. Желтым - `Per base sequence content` и `Per sequence GC content`. Остальное - зеленым.
	**Для обратных ридов.** Красным отмечен пункт `Per base sequence quality`. Желтым - `Per base sequence content`, `Per sequence GC content` и `Per tile sequence quality`. Остальное - зеленым.
	В идеале желательно привести все пункты к зеленому. Или, как минимум, к желтому.

## Фильтрация ридов по качеству

1. **Фильтрация при помощи Trimmomatic с заданными по условию параметрами:**

	- ```trimmomatic PE -phred33 ./raw/amp_res_1.fastq.gz ./raw/amp_res_2.fastq.gz  paired1.fq  single1.fq  paired2.fq  single2.fq LEADING:20 TRAILING:20 SLIDINGWINDOW:10:20 MINLEN:20 &> ./output/trimmomatic.txt```

	`LEADING:20` - обрезает прочтения из начала рида, если их качество ниже 20.
	`TRAILING:20` - обрезает прочтения из конца рида, если их качество ниже 20.
	`SLIDINGWINDOW:10:20` - сканирует риды с окном в 10 прочтений с 5'-конца и обрезает риды, когда их качество падает ниже 20.
	`MINLEN:20` - удаляет риды, длина которых ниже 20 прочтений.
	
	Вывод программы, включающий некоторую статистику, сохранен в файл `./output/trimmomatic.txt`
	
2. **Оценка количества ридов с помощью `wc -l`:**
	- ```wc -l paired1.fq > ./output/wc_paired1.txt```
	- ```wc -l paired2.fq > ./output/wc_paired2.txt```

	Количество строк в обоих файлах - **1785036**. Следовательно, количество ридов - **446259**, что совпадает со статистикой, выведенной через trimmomatic.

3. **Проверка качества оставшихся ридов:**
	- ```fastqc -o .  paired1.fq  paired2.fq &> ./output/fastqc_paired.txt```
	
	**Для прямых и обратных ридов.** Пункт `Per base sequence quality` сменил цвет с красного на зеленый. `Sequence Length Distribution` - с зеленого на желтый. Остальные - без изменений. В целом, тем не менее, качество стало лучше.

4. **Пробуем повысить порог качества до 30 для всех типов фильтрации**
	- ```trimmomatic PE -phred33 ./raw/amp_res_1.fastq.gz  ./raw/amp_res_2.fastq.gz 30_paired1.fq 30_single1.fq 30_paired2.fq 30_single2.fq LEADING:30 TRAILING:30 SLIDINGWINDOW:10:30 MINLEN:20 &> ./output/30_trimmomatic.txt```
	
	*Надеюсь, я верно понял, что MINLEN повышать не нужно*
	- ```fastqc -o .  30_paired1.fq  30_paired2.fq &> ./output/fastqc_30_paired.txt```

	Для прямых ридов качество существенно не изменилось в сравнении с фильтрацией по качеству 20, т.к. не изменился цвет ни одного из пунктов. Для обратных ридов пункт `Per sequence GC content` сменил цвет с желтого на зеленый. Однако, существенно упало общее количество ридов - с **446259** до **376340**, ввиду чего мне кажется, что данные параметры фильтрации не являются оптимальными.

## Выравнивание на референс

1. **Построение индекса с помощью BWA:**
	- ```bwa index ./raw/GCF_000005845.2_ASM584v2_genomic.fna &> ./output/bwa_index.txt```

2. **Выравнивание на референс**:
	- ```bwa mem ./raw/GCF_000005845.2_ASM584v2_genomic.fna  paired1.fq  paired2.fq > alignment.sam 2> ./output/bwa_mem.txt```

## Сжатие SAM-файла

1. **Конвертирование sam-файла в bam-файл:**
	- ```samtools view -S -b alignment.sam > alignment.bam```

## Сортировка и индексирование bam-файла

1. **Сортировка по координатам из референса:**
	- ```samtools sort alignment.bam -o alignment_sorted.bam```
2. **Индексирование отсортированного  bam-файла:**
	- ```samtools index alignment_sorted.bam```
3. **На данном этапе мы можем посмотреть на наше выравнивание в IGV browser'е.** 

	В пункте “Genomes” > ”Load Genome from File” выбираем референсный геном. В пункте `File` > `Open from file` выбираем полученный `alignment_sorted.bam`. Сессия сохранена в файле `ref_and_sorted_bam.xml`

## Variant calling
1. **Создание промежуточного файла **mpileup**:**

	- ```samtools mpileup -f ./raw/GCF_000005845.2_ASM584v2_genomic.fna  alignment_sorted.bam >  my.mpileup 2> ./output/my_mpileup.txt```

2. **Фильтрация данных при помощи VarScan при значении `--min-var-freq`, равном `0.70`:**

	- ```varscan mpileup2snp my.mpileup --min-var-freq 0.70 --variants --output-vcf 1 > VarScan_results.vcf 2> ./output/varscan.txt```

## "Ручное" предсказание эффектов 
1. **Открываем IGV-browser.** 
2. **Загружаем сессию `ref_and_sorted_bam.xml`**
3. **В пункте `File` > `Open from file` добавляем `VarScan_results.vcf` и исходный файл с аннотацией `GCF_000005845.2_ASM584v2_genomic.gff`, лежащий в папке raw.** Сессия сохранена в файле `manual_pred.xml`
4. **Пытаемся предсказывать эффект каждой мутации** *~~(В этот пункт надо будет что-то более внятное вставить, но у меня уже сил нет пытаться ща что-т анализировать. Надеюсь, ты это сделаешь. И надо не забыть убрать сей опус...)~~*

## Автоматическое предсказание эффектов 
1. **Загружаем файл, содержащий референсную последовательность и аннотацию, по ссылке ниже и помещаем его в папку raw:**
[https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/005/845/GCF_000005845.2_ASM584v2/GCF_000005845.2_ASM584v2_genomic.gbff.gz](https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/005/845/GCF_000005845.2_ASM584v2/GCF_000005845.2_ASM584v2_genomic.gbff.gz)**

2. **Создаем пустой файл `snpEff.config`:**
		- ```touch  snpEff.config```

3. **Добавляем в файл `snpEff.config` строку `k12.genome : ecoli_K12` при помощи текстового редактора nano.**

4. **Создаем директорию для базы данных:**
	- ```mkdir -p data/k12```

5. **Распаковываем архив с gbff-файлом, переименовываем и переносим файл в папку с базой данных:**
	- ``` gunzip ./raw/GCF_000005845.2_ASM584v2_genomic.gbff.gz```
	- ``` cp  ./raw/GCF_000005845.2_ASM584v2_genomic.gbff  data/k12/genes.gbk```

6. **Создаем базу данных:**
	- ```snpEff build -genbank -v k12 &> ./output/snpEff_build.txt```

7. **Аннотируем:**
	- ```snpEff ann k12 VarScan_results.vcf > VarScan_results_annotated.vcf```

8. **Смотрим и оцениваем результаты в IGV-browser'е.** 

	Открываем сессию `manual_pred.xml`, удаляем трек `VarScan_results.vcf`(правая кнопка мыши по нему > remove). В пункте `File` > `Open from file` добавляем `VarScan_results_annotated.vcf`. 
