# IntelligentSystem_GroupWork
知的システムグループワーク用の開発レポジトリです。

### 関数の説明
#### ・スクレイピング
##### import_news()
| 引数 | 型 | 説明 |
|:-----:|-----|-----|
|  url  | str |  スクレイピングしたいURLを入力する。  |
```python
def import_news(url):
  # requestsを使って、ウェブページのHTMLを取得する
  response = requests.get(url)
  html = response.text
  # BeautifulSoupを使って、HTMLを解析する
  soup = BeautifulSoup(html, 'html.parser')

  # 必要な情報を抽出する
  article_title = soup.find('h1',{'class': 'sc-epOimh'}).text
  article_content = soup.find('div', {'class': 'article_body'}).text

  return article_title,article_content
```
#### ・形態素解析
##### create_prule()
| 引数 | 型 | 説明 |
|:-----:|-----|-----|
|  folder_path  | str |  スクレイピングした記事.txtが格納されているフォルダパスを指定。<br>今回は'./news' |
```python
def create_prule(folder_path):
  prule = [] # 形態素解析用

  for filename in os.listdir(folder_path):
    file_path = os.path.join(folder_path, filename)
    if os.path.isfile(file_path) and file_path.endswith(".txt"):
      with open(file_path, "r", encoding="utf-8") as file:
        text = file.read()
        name = file_path.split('/')[-1].split('.')[0]

        nouns = extract_nouns(text) # 関数呼び出し

        prule.append([name]) # タイトル
        # prule.append([urls_dic[name]]) # URL
        for word in nouns:
          prule[-1].append(word)
  return prule
```
##### extract_nouns()
| 引数 | 型 | 説明 |
|:-----:|-----|-----|
|  text  | str |  分かち書きしたい文章を指定する<br>今回は記事の本文  |
```python
def extract_nouns(text):
    mecab = MeCab.Tagger(ipadic.MECAB_ARGS)  # 単語に分割するためのTaggerを作成
    parsed_text = mecab.parse(text)  # 文章を形態素解析する

    nouns = []
    for word in parsed_text.split():
        word_parsed = mecab.parse(word).split('\t')[0]  # 単語を形態素解析して基本形を取得
        part_of_speech = mecab.parse(word).split('\t')[1].split(',')[0]  # 単語の品詞を取得
        if part_of_speech == '名詞':
            nouns.append(word_parsed)  # 名詞のみリストに追加

    tango = []
    for noun in nouns:
      if noun not in ['名詞','動詞','助詞','形容詞','助動詞','記号',r'[0-9]+']:
        tango.append(noun)
    return tango
```
#### ・エキスパートシステム
##### answer()
| 引数 | 型 | 説明 |
|:-----:|-----|-----|
|  inputline  | str |  ユーザーの入力  |
|  prule  | list |  ２次元配列であり、以下のような保存形式である。<br>[ ['記事１タイトル', '単語１', '単語２' ,,,, '単語N']<br>, ['記事２タイトル', '単語１', '単語２' ,,,, '単語N'] ]  |
```python
def answer(inputline, prule):
  no = 0
  for singlep in prule:
    if rulematch(inputline, singlep): # 関数呼び出し
      no+=1
  if no==0:
    print(f"{botname}: どうぞ、続けてください")
  else:
    limit = random.randint(0,no-1)
    no = 0
    for singlep in prule:
      if rulematch(inputline, singlep): # 関数呼び出し
        if no==limit:
          print(f"{botname}: {singlep[0]}")
          break
        no+=1
```

##### rulematch()
| 引数 | 型 | 説明 |
|:-----:|-----|-----|
|  inputline  | str |  ユーザーの入力  |
|  prule  | list |  １次元配列であり、以下のような保存形式である。<br>['記事１タイトル', '単語１', '単語２' ,,,, '単語N']  |
```python
def rulematch(inputline, singlep):
  count = 0
  for i in range(len(singlep)-1):
    if singlep[i] == "":
      count+=1
    elif singlep[i] in inputline:
      count+=1
    else:
      count+=0 # 何もしない
  if count >= 24: #24文字以上で検索
    return True
  else:
    return False
```

