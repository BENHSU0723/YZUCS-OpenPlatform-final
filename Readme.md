# Dacrd留言分析器
基於Dcard所提供的API去抓取各個文章的資料，接著經過一些資料預處理後，把每個留言用RNN、RF(RandomForest)、SVM三個預訓練好的模型去預測留言為正向或是負面，最終以網頁顯示結果


---

## Content
* Domo
* Setup
* API Application Section
* Program Section


---

## Demo
**Step1:** 執行 main.py，會跳出Dcard首頁與預測顯示網站(Dcard分析家)
![](https://i.imgur.com/ThVfckl.jpg)
![](https://i.imgur.com/G0KrtxX.jpg)


**Step2:** 選擇其中一篇文章並複製其網址，貼上於預測顯示網站(Dcard分析家)上的 **輸入網址** 之位置並按下 **送出** 鍵
![](https://i.imgur.com/iWjZnvx.jpg)


**Step3:** 接著會不停跳出一些Chrome網頁並且關閉，那是在執行程式，不用多加理會，大約等待約1分鐘結果會顯示出來
![](https://i.imgur.com/UZ1lT2G.jpg)


**Step4:** 最終網頁會顯示如圖，包含文章內文、愛心數、留言數...等等文章資訊。**重點在於留言分析分類了大部分的語意為正面或負面**
![](https://i.imgur.com/kkIPHKR.jpg)


---

## Setup
first download the folder(saved_joblib) [here](https://drive.google.com/drive/folders/1VbkIrhL8OoeBUW_r224kuxjeHwHUrM-r?usp=sharing) and put it under **OpenPlatform-Project/** , then check these  packs below are installed in your computer
* webbrowser
* flask
* jieba
* pandas
* selenium
* webdriver-manager
* bs4
* keras
* **tensorflow >= 2.3.0** (the older version would get error)
* joblib


---

## API Application Section
Dcard API使用十分簡單，只要按造格式輸入網址即可獲得包含文章各式資訊的網頁，接著用BeautifulSoup和json.loads進行處理後即可以拿取所有資訊，格式簡單介紹如下，詳細格式[見此](https://blog.jiatool.com/posts/dcard_api_v2/)

> Ex: Dcard首頁上全部文章
> https://www.dcard.tw/service/api/v2/posts
> 
> Ex: 文章留言
> https://www.dcard.tw/service/api/v2/posts/{文章ID}/comments
> 
> Ex: 文章內引用連結
> https://www.dcard.tw/service/api/v2/posts/{文章ID}/links

![](https://i.imgur.com/kWilcXD.jpg)

#### 注意!! 此API如果使用下面此方法會回傳存取遭拒絕 ❌
`r = requests.get('url')` 
#### 因此在實作上採用Chrome直接呼叫視窗去存取 ⭕
```
from selenium import webdriver
browser = webdriver.Chrome()
browser.get(url1)
```
#### 另外要用網頁直接顯示資料最好要開始無痕模式以免顯示錯誤 ⭕
![](https://i.imgur.com/DKEUjS9.jpg)


---
## Program Section

All sections I introduce below are attatch at python files.

Now that's see some critical sections in this comment analsis system. 🤪


* main.py -使用flask連結網頁與程式，讓分析完之留言正確顯示在網頁上
```
app = Flask(__name__)
@app.route('/', methods=['POST', 'GET'])

def index():
    if request.method == 'POST':
        if request.values['send'] == '送出':
            print("in")
            print(request.values['user'])
            name = request.values['user']  # name為輸入的網址
            print(name)
            ...

if __name__ == '__main__':
    webbrowser.open_new('http://127.0.0.1:5000/')
    webbrowser.open_new('https://www.dcard.tw/')
    app.run()
```


* newdcard-crawler.py - 使用chrome所提供工具開啟網頁去使用API，接著對於得到之json檔提取所需資料
```
def getdata(url):
    url1 = findid(url)  # api get
    browser = webdriver.Chrome()
    browser.get(url1)
    soup = BeautifulSoup(browser.page_source, 'html.parser')

    article = json.loads(soup.text)
    content = article['content']
    content = content.encode('cp950', 'ignore')
    content = content.decode('cp950')
    comment_count = article['commentCount']  # 留言數量
    title=article['title']
    print('文章標題: '+title)
    forumAlias=article['forumAlias']
    gender=article['gender']
    print('留言數量: '+str(comment_count))
```


* testers.py -載入預先訓練好的模型，可以直接對詞句進行判斷為正面還是負面，另外SVM和RF也是用類似方法載入使用 
```
    import tensorflow as tf

    new_model = tf.keras.models.load_model('saved_model/my_model')

    # Check its architecture
    result = new_model.predict([xtrain])
    RNNresult = []
    for i in range(0,len(result)):
        if(result[i][0]>result[i][1]):
            RNNresult.append('負面')
        else:
            RNNresult.append('正面')
```
            













