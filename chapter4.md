# 安裝環境

## 安裝 Python 3

#### Windows ：

這邊只有 Windows 環境的安裝教學 。
- [Python 官網](https://www.python.org/)
- 在 Downloads 標籤裡選擇 3.6
- 主標籤中預設為 32 位元 ， 也可以進入下載列表選擇 64 位元下載
- 安裝時記得把 Add Python to PATH 選項勾起來 ， 如不勾之後可以自行添加 Python 至 PATH 中
- 記得一併安裝 pip

<hr>

## 安裝 pip

#### Windows ：
安裝 python 請一併安裝 pip

<hr>

## Pip 說明

pip 是 python 的套件管理程式 ， 以下為 pip 常用指令
<br>
其他作業系統的 pip 通常預設指到 python2<br>
所以其他系統可能要將 pip 指令改成 pip3指令
```bash
$ pip install [packagename] # 安裝套件
$ pip install --upgrade pip # 更新 pip ， 需管理員權限
$ pip list # 套件列表
$ pip freeze # 將套件列表以 requirements 格式印出
$ pip freeze > requirements.txt # 套件列表用特定格式輸出到 requirements.txt
$ pip -r requirements.txt # 安裝所有 requirements.txt 中的套件
$ pip install -t lib # 將套件安裝到 lib 資料夾 
```
pip 安裝套件時預設會裝在整個主環境底下<br>
如有許多專案 ， 可能導致套件雜亂<br>
因此將利用 virtualenv (虛擬環境)來管理專案套件<br>

<hr>

## Pipenv

安裝 pipenv
```bash
$ pip install pipenv
```

pipenv 是 python 官方推薦的套件管理工具 ， 可以用指定的 python 版本處理專案 ， pipenv 同時也包含 virtualenv<br>
詳情參考[pipenv](https://pipenv.readthedocs.io/en/latest/)與[指令列表](https://github.com/pypa/pipenv#-usage)

<hr>

## 建立 virtualenv

安裝好 pipenv 之後就可以建立虛擬環境，每個虛擬環境彼此間不受干擾
```bash
| global # nothing
| 
| _ _ virtualenv01 # package01
|
|
| _ _ virtualenv02 # package01 、 package02
|
|
| _ _ virtualenv03 # package01 、 package03
# 啟動該虛擬環境就能用該虛擬環境底下的套件
```

#### 利用 virtualenv 建立虛擬環境(基本)
在你想要建立環境的資料夾裡輸入：
```bash
$ virtualenv ./venv # 在當前資料夾底下建立虛擬環境(venv 為虛擬環境資料夾名)
```
開起虛擬環境：
```bash
$ venv\scripts\activate # windows
$ source venv/bin/activate # ubuntu
```
開啟後用 pip 安裝的東西都會裝在虛擬環境裡 ， 要關閉虛擬環境只要輸入 ：
```bash
$ deactivate
```

#### 利用 pipenv 建立虛擬環境(推薦)

常用指令列表 ：
```bash
$ pipenv --venv # 印出當前專案所使用的虛擬環境的路徑
$ pipenv --rm # 刪除當前專案所使用的虛擬環境
$ pipenv install # (若無則)建立虛擬環境，有Pipfile檔則一併安裝裡面的套件
$ pipenv install [package-name] # 安裝套件在虛擬環境裡 、 創建 Pipfile與Pipfile.lock 、 (若無則)建立虛擬環境
$ pipenv shell # 啟動當前專案的虛擬環境
$ exit # 離開當前虛擬環境
```
- 若要指定 python 版本，該版本必須已安裝
- 創建的虛擬環境會統一放在一個預設的資料夾底下
[全部指令](https://github.com/pypa/pipenv#-usage)

pipenv會依據Pipfile檔來判斷是否是一個python專案
```bash
| 
| _ _  A/(Pipfile)
     |
     | _ _ B/
```
- A 為獨立專案
- B 屬於 A 的子資料夾

<hr>

利用 pipenv 建立虛擬環境
```bash
$ cd your-project
$ pipenv install
```
會建立一個虛擬環境 ， 同時會建立 Pipfile 與 Pipfile.lock 兩個檔案<br>
這兩個檔案是用來管理套件的 ， 用 pipenv 安裝的東西都會自動列在 Pipfile 裡<br><br>
啟動虛擬環境 ：
```bash
$ pipenv shell
```
如在 windows 系統出現 ... prevent_path_error() SyntaxError: invalid syntax 的問題 ， 輸入下面指令後重新啟動虛擬環境(官方可能已更新，所以不會出現問題或是解法沒有效果)：
```bash
$ pip install -e git+https://github.com/pypa/pipenv.git@master#egg=pipenv
```
輸入 exit 或 Ctrl + D 離開虛擬環境
```
$ exit
```

<hr>

## 安裝爬蟲所需套件

#### beautifulsoup4 用

安裝 requests
```bash
$ pipenv install requests
```

安裝 beautifulsoup4
```bash
$ pipenv install beautifulsoup4
```

#### Scrapy框架用

安裝 Scrapy
```bash
$ pipenv install scrapy
```

安裝 pypiwin32
```bash
$ pipenv install pypiwin32
```

安裝 Pillow(Scrapy 下載圖片用)
```bash
$ pipenv install Pillow
```