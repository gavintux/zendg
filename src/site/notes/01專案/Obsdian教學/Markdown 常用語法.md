---
{"dg-publish":true,"permalink":"/01/obsdian/markdown/","dgPassFrontmatter":true,"created":"2025-05-19T09:28:47.303+08:00","updated":"2025-08-27T14:00:06.697+08:00"}
---

# 標題
# 標題 H1
## 標題 H2
### 標題 H3
#### 標題 H4
##### 標題 H5
###### 標題 H6
```
# 標題 H1  
## 標題 H2  
### 標題 H3  
#### 標題 H4  
##### 標題 H5  
###### 標題 H6
```
---
> ` 符號。option+shift+~


# 文字格式

- ## 粗體字
    
    **粗體字**
 
 ```
   **粗體字**  或 __粗體字__
```
  
- ## 斜體字
    
    _斜體字_
    
```
    *斜體字* 
	_斜體字_
```
    
- ## 底線
<u>底線</u>
```
<u>底線</u>
```
    
- ## 刪除線
    ~~刪除線~~
```
~~刪除線~~
```  
- ## 文字凸顯
`文字凸顯`
```
`文字凸顯`
```

---

# 引言

> 引言，若要換行，結尾要加 `兩個空白` 如 _ _  
> 這是新的一行
```
> 引言，若要換行，結尾要加 `兩個空白`    > 這是新的一行
```

# 連結

- ## 網址連結
<https://www.naes.ilc.edu.tw>

在網址前、後加上 “<” 和 “>” 字元
```
<https://www.google.com>
```

- ## 文字連結

    [(宜蘭縣南澳國小首頁)](https://www.naes.ilc.edu.tw/)

```
    |1|[連結文字](https://www.google.com)|
```
---

# 圖片
## 網路圖片
![2024芬蘭海豚杯](https://image.taiwannews.com.tw/2024%2F08%2F07%2F9727b7bf1b6a4aadb739d22eb5c24d96.jpg "Finland's Delphin Basket Tournament")

```
![圖片 Alt]( "圖片網址" "圖片 Title") 
或  
![]("圖片網址")
```
## 圖片檔案
![incoming-D479F884-67E7-4E0A-AFB8-B3B8CA597BEF.png](/img/user/Attachment/incoming-D479F884-67E7-4E0A-AFB8-B3B8CA597BEF.png)
```
![["圖片檔名"]]
```

# 分隔線
--- 
```
---
```


---

# 數學公式 **_(GitHub 不支援)_**

> 語法參考： [mhchem for MathJax mhchem for KaTeX](https://mhchem.github.io/MathJax-mhchem)

$𝑓(𝑥)=10$
```
$f(x)=10$
```

$K = \frac{[\ce{Hg^2+}][\ce{Hg}]}{[\ce{Hg2^2+}]}$

```
$K = \frac{[\ce{Hg^2+}][\ce{Hg}]}{[\ce{Hg2^2+}]}$
```

---

# 支援 HTML 語法

<div>  範例內容<br>使用 HTML 連結語法 <a href="http://www.google.com" target="_blank">Google</a> <br>含有<span style="color:#ff00ff"> 顏色 </span>的段落內容  <br></div>
```
<div>  範例內容<br>使用 HTML 連結語法 <a href="http://www.google.com" target="_blank">Google</a> <br>含有<span style="color:#ff00ff"> 顏色 </span>的段落內容  <br></div>
```

---

# 程式碼區塊

- ### 程式碼
    
    ```csharp
    // C# 程式碼區塊
    public class MyClass
    {
      public string String1 { get; set; }
      public int Int1 { get; set; }
    
      public MyClass()
      {
          Console.WriteLine("建構子");
      }
    }
    ```
>   ```csharp
    // C# 程式碼區塊
    public class MyClass
    {
      public string String1 { get; set; }
      public int Int1 { get; set; }
    
      public MyClass()
      {
          Console.WriteLine("建構子");
      }
    }
    ```

- ## Javascript 程式碼
    
    ```javascript
    // Javascript 程式碼區塊
    function MyFunc()
    {
      alert("Hello World !");
    }
    ```
>
    ```javascript
    // Javascript 程式碼區塊
    function MyFunc()
    {
      alert("Hello World !");
    }
    ```
---

# 表格、對齊方式、文字樣式
|A|B|C|D|
|:--|---|:-:|--:|
|XXXXXXXXXX|XXXXXXXXXX|XXXXXXXXXX|XXXXXXXXXX|
|靠左|預設|置中|靠右|
|A1|B1|C3|D1|
|`凸顯字`|B2|C2|D2|
|_斜體字_|**粗體字**|<u>底線</u>|~~刪除線~~|

```
|A|B|C|D|
|:--|---|:-:|--:|
|XXXXXXXXXX|XXXXXXXXXX|XXXXXXXXXX|XXXXXXXXXX|
|靠左|預設|置中|靠右|
|A1|B1|C3|D1|
|`凸顯字`|B2|C2|D2|
|_斜體字_|**粗體字**|<u>底線</u>|~~刪除線~~|
```
第一列是標題 ( 輸出結果以粗體顯示 )  
第二列是設置對齊方式，不會顯示 ( 至少三個 `-` 符號表示內容位置，`:` 表示對齊位置 )  
第三列之後是表格資料內容  
表格的 Markdown 可用 `空白鍵` 排版增加可讀性 ( 不會影響輸出結果 )  
表格內可使用文字格式 Markdown 語法

---

# 列表
1. test
2. t2
	1. testtest
	2. linkuangchang  tset__
	   test
- ## 有序列表
    
    有序列表開頭為 `數字` + `.` + `空白鍵`
    
1. 有序列表 1
2. 有序列表 2
    1. 有序列表 1-1
    2. 有序列表 1-2
        1. 有序列表為 `數字` + `.` + `空白鍵`
            
            > 2. 可搭配引言符號 `>`
             * 可搭配無序列表符號 *
            + 可搭配無序列表符號 +
             - 可搭配無序列表符號 -
            
    3. 有序列表 1-3 結尾加 `兩個空白鍵` + `換行` __  
        這行文字就會對齊
    4. 有序列表 1-4
3. 有序列表 3

- ## 無序列表
    
    無序列表開頭為 `*` 或 `+` 或 `-`
    
- 無序列表 1
- 無序列表 2
    - 無序列表 1-1
    - 無序列表 1-2
        - 無序列表為 `*` 或 `+` 或 `-`
            > - 可搭配引言符號 `>`
            > 1. 可搭配有序列表 `數字` + `.` + `空白鍵`
            > 2. 可搭配有序列表 `數字` + `.` + `空白鍵`
            > 3. 可搭配有序列表 `數字` + `.` + `空白鍵`
            
    - 無序列表 1-3 結尾加 `兩個空白鍵` + `換行` __  
        這行文字就會對齊
    - 無序列表 1-4
- 無序列表 3

---

# 待辦事項

- [ ] 寫月報表
- [x] 慢跑
- [x] 靜坐 
```
- [ ] 寫月報表
- [x] 慢跑
- [x] 靜坐 
```

---

# 表情符號 (emoji)

> 請參考 [https://www.webfx.com/tools/emoji-cheat-sheet](https://www.webfx.com/tools/emoji-cheat-sheet)

---

