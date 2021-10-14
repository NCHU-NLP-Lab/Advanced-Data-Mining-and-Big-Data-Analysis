## 高等資料探勘與巨量資料分析 作業一：文字共現關聯分析 (Keyword Correlation Analysis)
### 做資料探勘有三件事情需要留意：
- 第一：資料從哪來？
- 第二：資料從哪來？
- 第三：資料從哪來？

### 文字資料為最容易取得之資料，因此我們用文字資料當出發點
- 文字(Text Data)中富含許多知識(Knowledge)，但是如何將Knowledge從文字資料中萃取而出是一件困難的任務。
- 此次作業讓我們來分析文字關聯。
- 基本想法為兩個字詞(say Baseball 與 Pitcher)，如果常常一起出現在一段句子中，那麼這兩個字語意上(Semantically)應該有高度關聯。
- 若我們針對大量句子進行字詞間之共現關聯分析，將有機會萃取出文字間關聯知識。

### 以 S_1 = [歐巴馬出生於夏威夷，擔任美國總統]為例
- 我們可以列舉出現下面關聯 S_1
- [(歐巴馬, 夏威夷), (歐巴馬, 美國), (歐巴馬, 總統), (夏威夷, 美國), (夏威夷, 總統), (美國, 總統)]
- 假設我們有S_1, ..., S_10000 一萬個句子 
- 我們就可統計出字詞間之共現關係 -.-

### 資料在哪邊？
#### wiki資料下載
提供50000筆維基百科條目的JSON檔，請使用此資料來完成作業。

```
pip install gdown
gdown --id 1rQnbaOiqoN40AzHVq_IrRW4ki-rFPRxZ
```


## KCM作業繳交說明

#### 需求: 給定一字詞 W，請回傳在上述資料集(gdown --id 1rQnbaOiqoN40AzHVq_IrRW4ki-rFPRxZ)中與W共現次數最高的前10個字
- 網站已上線提供測試- 評分網站: <https://admbda.nlpnchu.org/scoring/>
- 下周上課會公布正式20題，一題5分，回傳的前十名與標準答案有5個以上重疊即拿到分數。
- 題目範例:['臺灣','美國']
- 答案範例 : [["日本", "香港", "中國大陸", "分佈", "中國", "中華民國", "日治", "臺北市", "名稱", 
"臺北"],["非建制地區", "城市", "人口普查", "加拿大", "英國", "地區", "加利福尼亞州", "國家", "伊利諾伊州", "公司"]]

---

- 評分網站: <https://admbda.nlpnchu.org/scoring/>
- 測試用題目:['臺灣', '美國', '大學', '肺炎','天安門','歌手','中國','蔡英文','立法院','颱風']
- 參考答案: testCase_answer.json <https://github.com/NCHU-NLP-Lab/110_Advanced-Data-Mining-and-Big-Data-Analysis/blob/main/KCM/testCase_answer.json>

#### 繳交答案網站使用說明
- 第一格填入課程代號+ilearing上的分組(Ex:6648_5,7748_1)，第二格填入你程式跑出的答案
- **答案請使用規定之JSON格式，例如:題目兩題各回傳關聯度最高的前兩名，格式應為 [["第一題第一名關鍵字", "第一題第二名關鍵字"],["第二題第一名關鍵字","第二題第二名關鍵字"]]**



#### 可參考資源
1.https://colab.research.google.com/github/astg606/py_materials/blob/master/useful_modules/introduction_json.ipynb (JSON格式介紹)

2.https://github.com/fxsjy/jieba (結巴斷詞)

3.範例框架 https://github.com/NCHU-NLP-Lab/110_Advanced-Data-Mining-and-Big-Data-Analysis/blob/main/KCM/KCM_Keyword_Correlation_Models_from_Open_Corpus.ipynb

4.Jieba斷詞範例程式碼 https://colab.research.google.com/drive/1KD_U2BOhhWKW9UdhKh_z3lp_EOfh4_I7?usp=sharing 

