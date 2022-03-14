# がん研究会PBL　解析　〜ゲノム解析〜
※下記チュートリアルはMacを前提に記載しています。その他のOSを利用の場合は、リンクなど適宜変更して下さい。
## インストール
ホームディレクトリに移動し、```PBL```というディレクトリを作成します。
```
cd
mkdir PBL
cd PBL
```
必要なツールをインストールします。
```
curl -OL https://ftp-trace.ncbi.nlm.nih.gov/sra/sdk/2.11.3/sratoolkit.2.11.3-mac64.tar.gz
curl -OL http://www.usadellab.org/cms/uploads/supplementary/Trimmomatic/Trimmomatic-0.39.zip
curl -OL https://sourceforge.net/projects/bio-bwa/files/bwa-0.7.17.tar.bz2
curl -OL https://github.com/samtools/samtools/releases/download/1.14/samtools-1.14.tar.bz2
curl -OL https://github.com/broadinstitute/gatk/releases/download/4.2.4.1/gatk-4.2.4.1.zip
curl -OL https://github.com/broadinstitute/picard/releases/download/2.26.10/picard.jar
curl -OL https://s3.amazonaws.com/plink1-assets/plink_mac_20210606.zip
curl -OL http://faculty.washington.edu/browning/beagle/beagle.28Jun21.220.jar
curl -OL https://snpeff.blob.core.windows.net/versions/snpEff_latest_core.zip
```
圧縮ファイルを解凍します。
```
mkdir plink
cd plink
unzip ../plink_mac_20210606.zip
cd ..
unzip Trimmomatic-0.39.zip
unzip gatk-4.2.4.1.zip
unzip snpEff_latest_core.zip 
tar -zxvf sratoolkit.2.11.3-mac64.tar.gz
tar -jxvf bwa-0.7.17.tar.bz2
tar -jxvf samtools-1.14.tar.bz2
```
bwaをコンパイルします。
```
cd bwa-0.7.17
make
cd ..
```
Samtoolsをコンパイルします。
```
cd samtools-1.14
./configure
make
cd ..
```
解析に必要なファイルをダウンロード、解凍します。
```
curl -OL https://github.com/nojima-q/2021-12-13-15_PBL_analysis/raw/main/Truseq_stranded_totalRNA_adapter.fa
curl -OL ftp://ftp.1000genomes.ebi.ac.uk//vol1/ftp/technical/reference/phase2_reference_assembly_sequence/hs37d5.fa.gz
curl -OL https://zenodo.org/record/3359882/files/ALL.wgs.phase3_shapeit2_mvncall_integrated_v5b.20130502.sites.vcf.gz
curl -OL https://zenodo.org/record/3359882/files/ALL.wgs.phase3_shapeit2_mvncall_integrated_v5b.20130502.sites.vcf.gz.tbi
curl -OL http://bochet.gcc.biostat.washington.edu/beagle/genetic_maps/plink.GRCh37.map.zip
unzip plink.GRCh37.map.zip
curl -OL http://bochet.gcc.biostat.washington.edu/beagle/1000_Genomes_phase3_v5a/b37.vcf/chr1.1kg.phase3.v5a.vcf.gz
curl -OL http://bochet.gcc.biostat.washington.edu/beagle/1000_Genomes_phase3_v5a/b37.vcf/chr1.1kg.phase3.v5a.vcf.gz.tbi
```

## 使用データ
下記のpaired-endでシーケンスされた２サンプルのデータを使用します。\
下記のpaired-endでシーケンスされた２サンプルのデータを使用します。\
肺がん患者の肺検体でWESした公共データ（[SRP114315](https://www.ncbi.nlm.nih.gov/Traces/study/?acc=SRP114315&o=acc_s%3Aa)）から正常部位、腫瘍部位１サンプルずつ取り出したものです。
今回は時間短縮のため、それぞれ１０万リードランダムサンプリングしたファイルを用意しました。\
下記リンクからダウンロード可能です。\
[Normal_1_100K.fastq.gz](https://github.com/nojima-q/2022.01.12-14_PBL_analysis/raw/main/Normal_1_100K.fastq.gz)\
[Normal_2_100K.fastq.gz](https://github.com/nojima-q/2022.01.12-14_PBL_analysis/raw/main/Normal_2_100K.fastq.gz)\
[Tumor_1_100K.fastq.gz](https://github.com/nojima-q/2022.01.12-14_PBL_analysis/raw/main/Tumor_1_100K.fastq.gz)\
[Tumor_2_100K.fastq.gz](https://github.com/nojima-q/2022.01.12-14_PBL_analysis/raw/main/Tumor_2_100K.fastq.gz)\
Macのデフォルト設定では```Download```ディレクトリに保存されますので、```PBL```ディレクトリに移動させます。
```
mv ~/Downloads/*_100K.fastq.gz ~/PBL/
```


## 1-1 公共データベースの紹介
![20190605_metacore](https://user-images.githubusercontent.com/85273234/144177090-bbba1e07-08de-4acf-bf6f-b7395a1e104d.jpg)
### NCBI SRA (https://www.ncbi.nlm.nih.gov/sra)
<img width="1190" alt="スクリーンショット 2021-12-01 14 22 50" src="https://user-images.githubusercontent.com/85273234/144176897-1c463d7f-ca18-41cf-979e-b70fb2db9f0e.png">

### DDBJ Sequence Read Archive (https://www.ddbj.nig.ac.jp/dra/index.html)
<img width="1792" alt="スクリーンショット 2021-12-01 14 23 16" src="https://user-images.githubusercontent.com/85273234/144177226-15d63718-b705-4f71-b103-dc84aa997a14.png">

### EBI ENA (https://www.ebi.ac.uk/ena/browser/home)
<img width="1167" alt="スクリーンショット 2021-12-01 14 23 58" src="https://user-images.githubusercontent.com/85273234/144177440-38a84e15-0555-4ae3-9984-08590f751b7f.png">


## 1-2 データ取得方法
１．解析したデータセットのAccession numberをNCBI SRA Run Selectorに入力し、『Search』をクリック。
<img width="1792" alt="スクリーンショット 2021-12-01 15 13 56" src="https://user-images.githubusercontent.com/85273234/144181792-1ac601bf-88d8-472e-a30f-d554e3b7d5a1.png">
２．データセット内の全てのデータを解析する場合は、Total行の『Accession List』をクリックし、サンプルごとのAccession numberが記載されたテキストファイル（SRR_Acc_List.txt）をダウンロードする。一部のデータのみ解析する場合は、必要なデータに☑をいれSelected行の『Accession List』をクリックし、SRR_Acc_List.txtをダウンロードする。
<img width="1792" alt="スクリーンショット 2021-12-01 15 19 26" src="https://user-images.githubusercontent.com/85273234/144182595-94ed6341-7722-4efb-8eec-5891f67ca4ae.png">
SRR_Acc_List.txtの内容\
<img width="201" alt="スクリーンショット 2021-12-01 15 24 07" src="https://user-images.githubusercontent.com/85273234/144183038-7209d17e-546a-43a9-8216-0f85d0b5d0b1.png">

SRA Toolkitの```prefetch```、```fastq-dump```を使ってデータを取得する。まず、```prefetch```でsraファイルがダウンロードされる。１つのsraファイルが20GBを超える場合は、-Xまたは--max-size 50Gなどのように最大数を変更する。--option-fileは使わずに直接Accession numberを入力して個別にダウンロードすることも可能です。
```
prefetch --option-file ~/PBL/SRR_Acc_list.txt
```
次に```fastq-dump```でsraファイルからfastqファイルを取得する。```PATH```にfastqファイルを格納したいディレクトリのパスを記載。\
```
find . -name '*.sra' -exec fastq-dump --gzip --split-files --outdir ~/PBL {} \;
```
- --gzip：圧縮ファイルとして出力する
- --split-files：レイアウトがpaired-endの際に指定する。single-endのときは不要。

## 2 FASTQファイルの形式、クオリティーコントロールについて
### １．FASTQファイルとは
@SRR8615662.1 1 length=101\
TGATGGCCCTGCCTTCGTGGGAACAGAGGCTAAGGCCTTGAG\
+SRR8615662.1 1 length=101\
CCCFFFFFHHHHHJJJJIJJJJIJIJJJJJJJJJJJJJJJJJFIIGIJJIJIJJIJJJJJHHHHHFFFFF\
\
１行目：＠から始まり、リードIDが記載されている\
２行目：シーケンスしたリードの塩基配列\
３行目：＋を記載\
４行目：２行目に記述した各塩基のクオリティ値

### ２．FASTQファイルのクオリティーコントロール
FastQCを使って、FASTQファイルの品質を確認します。\
端末から行う場合、下記のコマンドを実行します。
```
mkdir ~/fastqc_results
~/FastQC/fastqc -t 40 -o ~/fastqc_results/ ~/sample1_1_100K.fastq.gz
```
- -t：スレッド数（使用するPC環境に合わせて設定して下さい。）
- -o：出力ディレクトリ

アプリケーション版を使用する場合は、File>Openでファイルを指定し実行して下さい。\
複数ファイル指定することで、バッチ処理が可能です。
<img width="912" alt="スクリーンショット 2021-12-02 17 19 34" src="https://user-images.githubusercontent.com/85273234/144384259-16f77dc6-b572-4e5e-b865-31aa4e7429d0.png">

### ３．FastQC解析結果
各QC結果の説明は、下記のブログでよくまとめられています。\
https://imamachi-n.hatenablog.com/entry/2017/03/27/234350

#### Per base sequence quality
リードの各塩基のクオリティスコアを示しています。 Phred quality scoreがだいたいグリーンの領域（Scoreが28以上）に収まっているかどうか確認します。 結果として、クオリティが低いリードは含まれていないことが確認できます。\
<img width="882" alt="スクリーンショット 2021-12-03 12 19 30" src="https://user-images.githubusercontent.com/85273234/144539782-0ec533eb-3533-4e1a-94f6-7b25533e3463.png">

#### Per tile sequence quality
フローセルの各タイルごとのクオリティスコアを示しています。\
Illumina社製の次世代シーケンサーでは、「フローセル」と呼ばれるガラス基板上でDNA合成反応を行います。このフローセルは「タイル」と呼ばれる単位に区切られており、各タイルごとに塩基配列を決定しています。\
シーケンスをかけたときに、例えば特定のタイルに気泡やゴミが入っているとクオリティの低下が見られることがあります。\
特定のタイルで著しいクオリティの低下が見られる場合は、シークエンス時に上記のような問題があったと考えられます。

<img width="883" alt="スクリーンショット 2021-12-03 12 19 40" src="https://user-images.githubusercontent.com/85273234/144539828-47dd135b-650c-46ea-9c12-0467566b7cbf.png">

#### Per sequence quality scores
各リードの全長のクオリティスコアの分布を示しています。 
<img width="854" alt="スクリーンショット 2021-12-03 12 19 59" src="https://user-images.githubusercontent.com/85273234/144539876-2bb62a89-aed8-486b-892c-212b3e9943c9.png">

#### Per base sequence content
各塩基のA, T, G, Cの含有量を示しています。 RNA-seqの場合、それぞれの含有量はほぼ25%ずつになりますが、PAR-CLIPのようにRNA結合タンパク質と結合しているRNA配列を抽出してきている場合、それぞれの含有率に偏りが見られます。
<img width="882" alt="スクリーンショット 2021-12-03 12 20 10" src="https://user-images.githubusercontent.com/85273234/144539910-8ddb14a5-cba5-4ae3-bcb1-efe42fe7320f.png">

#### Per sequence GC content
リードのGC contentsの分布を示しています。 
<img width="861" alt="スクリーンショット 2021-12-03 12 20 18" src="https://user-images.githubusercontent.com/85273234/144539949-a255551b-f00c-4c80-907c-b5c93767308a.png">

#### Per base N content
各塩基中に含まれるNの含有率（塩基を読めなかった箇所）を示しています。 
<img width="880" alt="スクリーンショット 2021-12-03 12 20 28" src="https://user-images.githubusercontent.com/85273234/144540515-9f9c67a5-b570-4262-8097-4d8f36ab8279.png">

#### Sequence Length Distribution
リード長の分布を示しています。 
<img width="855" alt="スクリーンショット 2021-12-03 12 20 37" src="https://user-images.githubusercontent.com/85273234/144540533-0a74b05a-fc62-4892-aad9-50443deeed4c.png">

#### Sequence Duplication Levels
Duplidate readsの含まれている数を示しています。 
<img width="858" alt="スクリーンショット 2021-12-03 12 20 48" src="https://user-images.githubusercontent.com/85273234/144540550-89d8a421-a93c-4a71-96c0-66b0d764af24.png">

#### Overrepresented sequences
頻出する特徴配列が示されています。リード中にアダプター配列などが混入している場合、その配列が示されます。
<img width="879" alt="スクリーンショット 2021-12-03 13 17 24" src="https://user-images.githubusercontent.com/85273234/144544469-dc72869e-1df7-4de3-af65-a9d6d6ef6dc1.png">

#### Adapter Content
各塩基ごとに見たときのリード中に含まれているアダプターの割合を示しています。 あくまで、FastQCに登録されているアダプター配列しか確認していないので、登録されていないアダプター配列を使っていた場合、そのアダプター配列がリード中に混入していても確認できないことがあります。 
<img width="898" alt="スクリーンショット 2021-12-03 13 15 53" src="https://user-images.githubusercontent.com/85273234/144544371-74ddb7ed-97d1-4ffa-b934-50ef78ebd7e5.png">

## 3 アダプターの除去およびリードのトリミング
Trimmomaticを使って、アダプターの除去および低スコアな塩基のトリミングを同時に行います。
アダプター配列を記載したFATSAファイルは[ここ](https://github.com/nojima-q/2021-12-13-15_PBL_analysis/raw/main/Truseq_stranded_totalRNA_adapter.fa)からダウンロード可能です。（今回は、illumina社のTruSeqシリーズの配列を記載しています。自前データで実行する際は、ライブラリー作製キットで使用している配列が記載されたFASTAファイルを用意して実行して下さい。）\
下記はpaired-endでシーケンスしたFASTQファイルの場合です。
```
java -jar ~/PBL/Trimmomatic-0.39/trimmomatic-0.39.jar PE -threads 8 -phred33 ~/PBL/Normal_1_100K.fastq.gz ~/PBL/Normal_2_100K.fastq.gz ~/PBL/Normal_1_100K_trim_paired.fastq.gz ~/PBL/Normal_1_100K_trim_unpaired.fastq.gz ~/PBL/Normal_2_100K_trim_paired.fastq.gz ~/PBL/Normal_2_100K_trim_unpaired.fastq.gz ILLUMINACLIP:Truseq_stranded_totalRNA_adapter.fa:2:30:10 LEADING:20 TRAILING:20 SLIDINGWINDOW:4:20 MINLEN:25
java -jar ~/PBL/Trimmomatic-0.39/trimmomatic-0.39.jar PE -threads 8 -phred33 ~/PBL/Tumor_1_100K.fastq.gz ~/PBL/Tumor_2_100K.fastq.gz ~/PBL/Tumor_1_100K_trim_paired.fastq.gz ~/PBL/Tumor_1_100K_trim_unpaired.fastq.gz ~/PBL/Tumor_2_100K_trim_paired.fastq.gz ~/PBL/Tumor_2_100K_trim_unpaired.fastq.gz ILLUMINACLIP:Truseq_stranded_totalRNA_adapter.fa:2:30:10 LEADING:20 TRAILING:20 SLIDINGWINDOW:4:20 MINLEN:25
```
- PE：レイアウトがpaired-endシーケンスのときに指定。single-endの場合はSEを指定します。
- -threads：スレッド数（使用するPC環境に合わせて設定して下さい。）
- ILLUMINACLIP：Truseq_stranded_totalRNA_adapter.faはillumina社のTruSeqシリーズのアダプター配列をFASTA形式で記載してもの。後ろの数字は、許容ミスマッチ数:palindrome clip threshold:simple clip thresholdを表す。
- LEADING：5'末端から設定したスコア値未満の塩基をトリム
- TRAILING：3'末端から設定したスコア値未満の塩基をトリム
- SLIDINGWINDOW：左の数字はウィンドウサイズ、右の数字は平均クオリティ値を表します。ウィンドウサイズの範囲内の塩基のスコア平均が設定値よりも低ければ、3'末端側の全ての塩基をトリム。\
- MINLEN：設定値未満の塩基数になったリードを除去する。

### リードトリミング後のFastQC結果
#### Per base sequence quality
リードの各塩基のクオリティスコアを示しています。 Phred quality scoreがだいたいグリーンの領域（Scoreが28以上）に収まっているかどうか確認します。 結果として、クオリティが低いリードは含まれていないことが確認できます。\
<img width="891" alt="スクリーンショット 2021-12-08 13 10 58" src="https://user-images.githubusercontent.com/85273234/145147570-d1b08712-befe-4826-81a4-35c60b3cf85d.png">

#### Per tile sequence quality
フローセルの各タイルごとのクオリティスコアを示しています。\
Illumina社製の次世代シーケンサーでは、「フローセル」と呼ばれるガラス基板上でDNA合成反応を行います。このフローセルは「タイル」と呼ばれる単位に区切られており、各タイルごとに塩基配列を決定しています。\
シーケンスをかけたときに、例えば特定のタイルに気泡やゴミが入っているとクオリティの低下が見られることがあります。\
特定のタイルで著しいクオリティの低下が見られる場合は、シークエンス時に上記のような問題があったと考えられます。

<img width="886" alt="スクリーンショット 2021-12-08 13 11 07" src="https://user-images.githubusercontent.com/85273234/145147294-8814525b-eca2-438c-875b-6e84172595d2.png">

#### Per sequence quality scores
各リードの全長のクオリティスコアの分布を示しています。 
<img width="868" alt="スクリーンショット 2021-12-08 13 11 16" src="https://user-images.githubusercontent.com/85273234/145147297-93a9b9e8-13e7-40a6-b471-1c800af9b62b.png">

#### Per base sequence content
各塩基のA, T, G, Cの含有量を示しています。 RNA-seqの場合、それぞれの含有量はほぼ25%ずつになりますが、PAR-CLIPのようにRNA結合タンパク質と結合しているRNA配列を抽出してきている場合、それぞれの含有率に偏りが見られます。
<img width="886" alt="スクリーンショット 2021-12-08 13 11 23" src="https://user-images.githubusercontent.com/85273234/145147305-7c91dfc9-83ac-474d-9612-8fec8b6d7804.png">

#### Per sequence GC content
リードのGC contentsの分布を示しています。 
<img width="865" alt="スクリーンショット 2021-12-08 13 11 30" src="https://user-images.githubusercontent.com/85273234/145147309-5eba3d04-d3aa-46c7-917b-c566df8bebbe.png">

#### Per base N content
各塩基中に含まれるNの含有率（塩基を読めなかった箇所）を示しています。 
<img width="886" alt="スクリーンショット 2021-12-08 13 11 37" src="https://user-images.githubusercontent.com/85273234/145147321-247d1c38-42ba-4965-a955-186cec9dbc52.png">

#### Sequence Length Distribution
リード長の分布を示しています。 
<img width="869" alt="スクリーンショット 2021-12-08 13 11 46" src="https://user-images.githubusercontent.com/85273234/145147326-bfaef550-03a7-4239-a344-9b06b5666dd1.png">

#### Sequence Duplication Levels
Duplidate readsの含まれている数を示しています。 
<img width="862" alt="スクリーンショット 2021-12-08 13 12 35" src="https://user-images.githubusercontent.com/85273234/145147340-515291c5-de1b-4894-a088-6e97fa8b0ef0.png">

#### Overrepresented sequences
頻出する特徴配列が示されています。リード中にアダプター配列などが混入している場合、その配列が示されます。
<img width="846" alt="スクリーンショット 2021-12-08 13 12 45" src="https://user-images.githubusercontent.com/85273234/145147348-7689cc33-8b84-4d69-9467-0b7218a7973d.png">

#### Adapter Content
各塩基ごとに見たときのリード中に含まれているアダプターの割合を示しています。 あくまで、FastQCに登録されているアダプター配列しか確認していないので、登録されていないアダプター配列を使っていた場合、そのアダプター配列がリード中に混入していても確認できないことがあります。\
<img width="862" alt="スクリーンショット 2021-12-08 13 13 07" src="https://user-images.githubusercontent.com/85273234/145147355-8b0a01f3-bbd7-4e5e-8e59-6e8745454f6c.png">

## 4 リファレンスゲノムファイル、アノテーションファイルの取得
トリミングしたリードは全ゲノム配列が記載されたリファレンスゲノムファイルにマッピングします。\
[Ensembl](https://www.ensembl.org/)は、様々な生物種のゲノム配列情報が格納されているデータベースです。\
今回は、ここからヒトのリファレンスゲノムファイルとアノテーションファイル（遺伝子ごとにコードされている染色体と染色体内でのポジションが記載されたファイル）をダウンロードします。\
<img width="1792" alt="スクリーンショット 2021-12-08 14 21 31" src="https://user-images.githubusercontent.com/85273234/145152855-ee648e1f-ae56-4e90-9a53-681ee6f0b079.png">
- まずはEnsemblトップページの『Human』からをクリックします・
- 次のページの右側『Gene annotation』内の『Download FASTA』をクリック。
- ftpサイトの『dna』をクリックすると複数のファイルが閲覧できます。このうち、『Homo_sapiens.GRCh38.dna.primary_assembly.fa.gz』を使用します。
- または、下記のコマンドでもダウンロード可能です。いずれの場合も```PBL```ディレクトリに入れてください。またいずれもダウンロードにはかなり時間がかかります。がん研究会のNASに格納しているファイルを利用して下さい。

**※かなり時間かかります。**
```
curl -OL http://ftp.ensembl.org/pub/release-105/fasta/homo_sapiens/dna/Homo_sapiens.GRCh38.dna.primary_assembly.fa.gz
```

## 5 リファレンスゲノムへのマッピング
### 5-1 リファレンスゲノムファイルのインデックス化
4でダウンロードしたリファレンスゲノムファイルをインデックス化します。一般的にゲノムデータのサイズは大きくなりがちでありそのままでは処理時間がかかってしまうため、インデックス（目次）ファイルを作成して高速にアクセスできるようにします。\
インデックス化に必要なプログラムは、多くの場合マッピングツールに含まれています。

**※かなり時間かかります。**
```
./bwa-0.7.17/bwa index ~/PBL/hs37d5.fa.gz
```

### 5-2 マッピング
いよいよマッピングを行います。１０万リードランダムサンプリングしたFASTQファイル（最初にダウンロードしたファイル）を5-1で作成したインデックス化したリファレンスゲノムにマッピングします。
```
./bwa-0.7.17/bwa mem -t 8 ./hs37d5.fa.gz ./Normal_1_100K_trim_paired.fastq.gz ./Normal_2_100K_trim_paired.fastq.gz > Normal.sam
./bwa-0.7.17/bwa mem -t 8 ./hs37d5.fa.gz ./Tumor_1_100K_trim_paired.fastq.gz ./Tumor_2_100K_trim_paired.fastq.gz > Tumor.sam
```
- -t：スレッド数（使用するPC環境に合わせて設定して下さい。）

```.sam```ファイルから```.bam```ファイルへ変換します。```.bam```ファイルはバイナリファイルのため、データサイズの縮小化と以降の解析作業の高速化させます。
```
./samtools-1.14/samtools sort -@ 8 -o ./Normal.bam ./Normal.sam
./samtools-1.14/samtools sort -@ 8 -o ./Tumor.bam ./Tumor.sam
```
- -@：スレッド数（使用するPC環境に合わせて設定して下さい。）
- -o：出力ファイル名を指定

## 6 マッピングデータの各種QC処理
マッピング結果であるBAMファイルを入力データとして、Duplicate リード (全く同じゲノム位置の同じ配列のリード) をマークします。\
```
./gatk-4.2.4.1/gatk MarkDuplicates -I ./Normal.bam -M ./metrics_Normal.txt -O ./Normal_MarkDuplicates.bam
./gatk-4.2.4.1/gatk MarkDuplicates -I ./Tumor.bam -M ./metrics_Tumor.txt -O ./Tumor_MarkDuplicates.bam
```
- -I：入力BAMファイル
- -M：Duplicatesのmetrics情報を出力
- -O：出力BAMファイル

次に参照ゲノムファイルのインデックスファイルとディクショナリーファイルを作成します。これを行わないと後の解析でファイルを作ってから実行しろと怒られます。\
インデックスファイルの作成。
```
./samtools-1.14/samtools faidx ./hs37d5.fa.gz
```
ディクショナリーファイルの作成
```
java -jar ./picard.jar CreateSequenceDictionary R=./hs37d5.fa.gz O=./hs37d5.fa.gz.dict
```
- -R：参照ゲノムファイル
- -O：出力Mファイル

Normal、Tumorそれぞれにsample名を入力します。RGSMに入力した文字列が最終的なサンプル名として扱われます。
```
java -jar picard.jar AddOrReplaceReadGroups I=Normal_MarkDuplicates.bam O=Normal_MarkDuplicates_AddOrReplaceReadGroups.bam RGLB=lib1 RGPL=ILLUMINA RGPU=unit1 RGSM=Normal RGID=Normal
java -jar picard.jar AddOrReplaceReadGroups I=Tumor_MarkDuplicates.bam O=Tumor_MarkDuplicates_AddOrReplaceReadGroups.bam RGLB=lib1 RGPL=ILLUMINA RGPU=unit1 RGSM=Tumor RGID=Tumor
```

既知変異情報をもとに、塩基スコアを再計算します。
```
./gatk-4.2.4.1/gatk BaseRecalibrator -I ./Normal_MarkDuplicates_AddOrReplaceReadGroups.bam --known-sites ./ALL.wgs.phase3_shapeit2_mvncall_integrated_v5b.20130502.sites.vcf.gz -O ./BaseRecalibrator_Normal.table -R ./hs37d5.fa.gz
./gatk-4.2.4.1/gatk BaseRecalibrator -I ./Tumor_MarkDuplicates_AddOrReplaceReadGroups.bam --known-sites ./ALL.wgs.phase3_shapeit2_mvncall_integrated_v5b.20130502.sites.vcf.gz -O ./BaseRecalibrator_Tumor.table -R ./hs37d5.fa.gz
```
- -I：入力BAMファイル
- --known-sites：既知変異情報ファイル
- -O：出力ファイル
- -R：参照ゲノムファイル

```
./gatk-4.2.4.1/gatk ApplyBQSR -I ./Normal_MarkDuplicates_AddOrReplaceReadGroups.bam -bqsr ./BaseRecalibrator_Normal.table -O ./Normal_MarkDuplicates_AddOrReplaceReadGroups_ApplyBQSR.bam
./gatk-4.2.4.1/gatk ApplyBQSR -I ./Tumor_MarkDuplicates_AddOrReplaceReadGroups.bam -bqsr ./BaseRecalibrator_Tumor.table -O ./Tumor_MarkDuplicates_AddOrReplaceReadGroups_ApplyBQSR.bam 
```

```
./gatk-4.2.4.1/gatk HaplotypeCaller -I ./Normal_MarkDuplicates_AddOrReplaceReadGroups_ApplyBQSR.bam -O ./Normal_MarkDuplicates_AddOrReplaceReadGroups_ApplyBQSR_HaplotypeCaller.bam -ERC GVCF -R ./hs37d5.fa.gz
./gatk-4.2.4.1/gatk HaplotypeCaller -I ./Tumor_MarkDuplicates_AddOrReplaceReadGroups_ApplyBQSR.bam -O ./Tumor_MarkDuplicates_AddOrReplaceReadGroups_ApplyBQSR_HaplotypeCaller.bam -ERC GVCF -R ./hs37d5.fa.gz
```
```
./gatk-4.2.4.1/gatk CombineGVCFs -R ./hs37d5.fa.gz -D ./ALL.wgs.phase3_shapeit2_mvncall_integrated_v5b.20130502.sites.vcf.gz -V ./Normal_MarkDuplicates_AddOrReplaceReadGroups_ApplyBQSR_HaplotypeCaller.g.vcf.gz -V ./Tumor_MarkDuplicates_AddOrReplaceReadGroups_ApplyBQSR_HaplotypeCaller.g.vcf.gz -O ./combine.g.vcf.gz
```
```
./gatk-4.2.4.1/gatk GenotypeGVCFs -R ./hs37d5.fa.gz -D ./ALL.wgs.phase3_shapeit2_mvncall_integrated_v5b.20130502.sites.vcf.gz -V ./combine.g.vcf.gz -O ./combine_GenotypeGVCFs.g.vcf.gz
```
```
./gatk-4.2.4.1/gatk SelectVariants -R ./hs37d5.fa.gz -V combine_GenotypeGVCFs.g.vcf.gz --select-type-to-include SNP -O combine_GenotypeGVCFs_SNPs.g.vcf.gz
```
- --select-type-to-include：```INDEL```と指定すると、INDELのみ抽出されます。

https://gatk.broadinstitute.org/hc/en-us/articles/360035531112--How-to-Filter-variants-either-with-VQSR-or-by-hard-filtering
```
./gatk-4.2.4.1/gatk VariantFiltration -R ./hs37d5.fa.gz -V ./combine_GenotypeGVCFs_SNPs.g.vcf.gz -O combine_GenotypeGVCFs_SNP_filtered.g.vcf.gz \
-filter "QD < 2.0" --filter-name "QD2" \
-filter "QUAL < 30.0" --filter-name "QUAL30" \
-filter "SOR > 3.0" --filter-name "SOR3" \
-filter "FS > 60.0" --filter-name "FS60" \
-filter "MQ < 40.0" --filter-name "MQ40" \
-filter "MQRankSum < -12.5" --filter-name "MQRankSum-12.5" \
-filter "ReadPosRankSum < -8.0" --filter-name "ReadPosRankSum-8"
```
```
./gatk-4.2.4.1/gatk SelectVariants -R ./hs37d5.fa.gz -V ./combine_GenotypeGVCFs_SNP_filtered.g.vcf.gz -O ./combine_GenotypeGVCFs_SNP_filtered_passed.g.vcf.gz -select 'vc.isNotFiltered()'
```
```
java -jar ./beagle.28Jun21.220.jar gt='combine_GenotypeGVCFs_SNP_filtered_passed.g.vcf.gz' out='combine_GenotypeGVCFs_SNP_filtered_passed_imputed_chr1' map='plink.chr1.GRCh37.map' ref='chr1.1kg.phase3.v5a.vcf.gz' chrom='1'
```

```
java -jar ./snpEff/snpEff.jar GRCh37.87 ./combine_GenotypeGVCFs_SNP_filtered_passed_imputed_chr1.vcf.gz > ./combine_GenotypeGVCFs_SNP_filtered_passed_imputed_chr1_annotated.vcf
```
```
grep ^\## -v ./combine_GenotypeGVCFs_SNP_filtered_passed_imputed_chr1_annotated.vcf | cut -f1,2,3,4,5,10,11 > c12345.txt
grep ^\## -v ./combine_GenotypeGVCFs_SNP_filtered_passed_imputed_chr1_annotated.vcf | cut -f8 | cut -d'|' -f4,8 | tr '|' '\t' > gene_region.txt
paste c12345.txt gene_region.txt > combine_GenotypeGVCFs_SNP_filtered_passed_annotated_extracted.txt
rm -rf c12345.txt gene_region.txt
```
