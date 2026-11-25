
# Project Name

## 1. Overview
このコードはESA MAAPからEarthCAREデータをダウンロードするためのコードです。
ご自身のアカウントよりToken を発行し、所定のファイル内に記載することでデータを取得し、Google Driveまたはローカルに格納できます。
コード自体はGoogle ColabGoogle Driveを想定していますが、通信が許可されているならローカルへのダウンロードも可能です。パスのみ記載したら、あとはコードがColabか以外かを判定し、必要な追加処理を実行します。
ESA MAAP上のデータについてはダウンロードなしの処理も可能ですので、ニーズに応じた利用を検討されてください。

## 2. Features
- MAAPからのデータ取得
- 期間指定（※長期間の設定も可能です、ただし、ダウンロード時間にはご注意ください）とプロダクトID+バージョンのシンプルな指定
- 長期間の指定が可能。
- ダウンロードディスクの容量確認機能
- 並列取得の設定
- Colab上動作時の自動設定（Google Driveのマウントも自動で実施してくれます。）

## 3. Preparations
- 事前にEOアカウントを発行:  
https://portal.maap.eo.esa.int/earthcare/join-collaborative-environment/ から"Join a Collaborative Environment", Log in with ESA Accountより"Create Account"にてアカウントを作成。
- ダウンロード用のTokenの発行

[Google Colabからの場合] Googe Colab, Google Driveのアカウント

## 4. Requirements
#### ローカルの場合   
- pystac_client  
Python >= 3.10, 但し、pystac_clientが動作したら基本的には動作する模様です。
OS: Linux / macOS / Windows  
※ Windowsについてはパスを適切に設定したら動作します。  

#### Colabの場合
Colab環境の場合、自動でpipでpystac_clientを取得します。

## Installation

リポジトリを取得のみです。
基本的にはpythonとpystac_clientの環境で、あとは単にコードとtokenを設定したcredentials.txtがあれば動作します。

```bash
git clone https://github.com/kalessinlord/EC_MAAP.git
cd EC_MAAP
```

## Usage

基本的な実行の流れ  
※事前にEOアカウントを発行  
1. tokenを発行( https://portal.maap.eo.esa.int/ini/services/auth/token/ )
2. credentials.txtのOFFLINE_TOKEN=の直後に1で取得したtokenをペースト  
3. 冒頭に期間とプロダクトID(10文字)などを設定(Configuration参照)  
4. 実行  

## Configuration
- 認証情報ファイル: credentials.txt
- 入力パラメータ: [主要設定項目]
基本的にはIn[1]、In[3]の指定のみです。最低限太字をまずは指定してください。

### In[1]   
#### 開始と終了日
**START_date**="2025-08-01T00:00:00Z"  
**END_date**  ="2025-08-05T23:59:59Z"  

####  取得プロダクトとバージョン
※最新のバージョンを取得の場合は、FLG_VAR_LATESTを1にしたらよいです。
**target_prd** ="	CPR_CLD_2A" #10 char  
target_var="DA" #次のFLG_VAR_LATESTが0かつ、バージョンを固定したいなら指定。
 see at https://earth.esa.int/eogateway/missions/earthcare/data
**FLG_VAR_LATEST**=1 # 最新のバージョンを取得の場合は1 

####  カタログの指定
L1, ESA L2, JAXA L2でカタログを個別に指定。
EC_COLLECTION = ['EarthCAREL1Validated_MAAP']
['EarthCAREL1Validated_MAAP'] : ESA/JAXA L1 (???_???_1?)
['EarthCAREL2Validated_MAAP'] : ESA L2 (JAXA L2以外)
['JAXAL2Validated_MAAP']      : JAXA L2 (???_CLP_2?)
JAXA L2 : CPR_ECO,CPR_CLP, MSI_CLP, ATL_CLA, AC__CLP, ACM_CLP, ALL_RAD 
ECO/CLP or CLA/ALL_RAD

#### ディレクトリの指定
Colabの場合 /content/drive/MyDrive以下を指定する。
WORK_DIRは指定しなくてもよく、削除し、直接DL_DIR, DL_URLを指定も可能
WORK_DIR="/content/drive/MyDrive" #**ColabからGoogle Driveの場合、MyDrive以下にディレクトリを格納**
DL_DIR=WORK_DIR+"/EC_DL" ** download directory**  
DL_URL=WORK_DIR+"/EC_URL" ** url text出力用,後のGURL_FLG=0なら作成されないので指定不要 **

MAAP_CRED=WORK_DIR+"/MAAP/credentials.txt" #tokenを張り付けるファイル

### In[3]
並列取得数などを設定。  最初はデフォルトでよい。
avail_cpus=8 #**並列取得数Colab Proまでなら8程度が妥当。**  
drive_limit=85 #**google driveの停止水準(%)**  

NOGET_FLG=0 #1なら処理を停止。urlファイル取得のみ  
FLG_HDR=0 #1ならHDRファイルまで取得  
GURL_FLG=0 #1なら取得したurlを残す。 0なら削除  


## Project Structure

```text
.
├── ec_maap_dlpar.py
├── credentials.txt(※OFFLINE_TOKENを設定のこと)
├── README.md(※本ファイル)
├── LICENSE
```

## Limitations
1. [既知の制約]
本プログラムは日数が多いアクセスを可能にしている一方で、日付単位のアクセスになる点注意(START_dateとEND_dateは日付のみ有効)。  
バージョン指定について、FLG_VAR_LATEST=0の場合、メジャーバージョン(vDaの場合D)のみ固定の指定は未実装  

2. [未対応の機能]
7セル目で、以下の部分を修正することで、フレーム指定や緯度経度の指定、取得数の上限の設定なども可能。
```python  
search = catalog.search(
    collections=EC_COLLECTION,
    filter=f"productType = '{target_prd}'",
    #datetime=[d.isoformat(), next_d.isoformat()],
    datetime=[d.isoformat(), d.isoformat()],
    method="GET",
    limit=1200      # 124*4以上でも可
)
```
3. [動作確認済みの環境]
- Google Colab  
- MacOS(26.7.3)+python(3.12.7)+pystac-client(0.9.0)  

## License
This project is released under the MIT License.

You are free to:
use
modify
redistribute
use commercially
without prior permission.

Copyright remains with the original author.
See the LICENSE file for details.

## Author
- Kohtaro R. Araragi (JAXA/SAOC)

## Acknowledgments / References
### [参考文献・利用ライブラリ等] ###
- ESA MAAP 
https://portal.maap.eo.esa.int/earthcare/
- ESA MAAP Documentation
https://portal.maap.eo.esa.int/earthcare/documentation/
- PySTAC Client (pystac_client) Documentation
https://pystac-client.readthedocs.io/en/stable/
- ESA sample code
https://catalog.maap.eo.esa.int/doc/data-access.html

