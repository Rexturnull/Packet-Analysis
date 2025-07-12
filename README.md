Packet Analysis
===
🔙 [HOME](../../HOME.md)


# Introduciton
為什麼要學習封包分析
- 弱點掃描是一個非常差勁政策，弱點掃描的本質是在**找尋已知漏洞，**滲透測試的本質是在**找尋未知漏洞**
- [遠東銀行事件惡意程式分析](https://blogs.longwin.com.tw/wordpress/201710-TWCERTCC-MIFR.pdf) (關鍵字 : 2017、FEIB、駭客、金額、PDF)
  - TWCERT在台灣算半官方的單位，所以該份報告具有一定的可信度
  - 早在104年開始金管會就規定金融單位，每季應進行弱點掃描，並針對其掃描或測試結果進行風險評估，同時也有密碼相關的規定([電子支付機構資訊系統標準及安全控管作業基準辦法](https://law.fsc.gov.tw/LawContent.aspx?id=GL001541))
  - 也就是說所有花錢能安裝的能買的，IPS、FW、防毒軟體，全部都已被銀行部屬，但事實是**駭客成功入侵銀行並且潛伏長達一年，這一年銀行進行過至少四次的弱掃**，卻沒有發現問題
  - 那為什麼還會被入侵，分析有以下兩種可能
    1. 各單位作的弱點掃描都只是虛應了事，如果是這樣的調查結果，那相關的遠東、馬偕、彰基的入侵事件，各單位都應該被受罰，但是事實是並沒有
    2. 剩下的可能就是，**<font color="red">現行的弱點掃描檢測對於資安是沒有用的</font>**
- 學習病毒封包分析的三大精神
  - 不要怕駭客 (再厲害都是人都是來上CEH，只是誰比較熟而已)
  - 不要怕病毒 (再厲害的病毒、勒索軟體，說到底都只是程式碼)
  - 不要怕資安事件 (冷靜應對步步為營，不要害怕做決定，最怕人猶豫不決)

# Malware Sample
- [MalwareBazaar](https://bazaar.abuse.ch/)
  - 瑞士資安組織 abuse.ch 所維護的一個全球性惡意軟體樣本分享平台
  - 每筆樣本附有惡意軟體家族分類、上傳時間與來源資訊
  - 支援 API 與自動化分析整合
- [VirusTotal](https://www.virustotal.com/)
  - Google 擁有的一個免費線上惡意檔案與網址掃描平台
  - 支援超過 70 種防毒引擎的掃描結果（如 Kaspersky、Symantec、TrendMicro 等）
  - 支援網址與 IP 檢測、YARA 規則搜尋與 API 整合
- 從 MalwareBazaar 複製某個惡意樣本的 Hash（例如 MD5、SHA256），然後貼到 VirusTotal 上查詢，就能看到該檔案目前被哪些防毒軟體偵測為惡意，哪些沒有偵測出來
  - 分析哪些新興惡意程式尚未被傳統防毒辨識
  - 分析各家防毒引擎的更新與偵測能力差異
  - 可以觀察到很多病毒其實防毒軟體都找不出來
- 也可以將樣本產出Hash之上傳到Virus Total進行分析
    ```bash
    certutil -hashfile dwm.exe MD5
    ```

# Wireshark
- [Basic](./Wireshark/Basic.md)
  - Configuration
  - Packet Color
  - Endpoint
  - Filter
  - HTTP


# Network Attack
1. 惡意程式(Malware、木馬、蠕蟲、加密勒索)，對外通訊的可能有以下三種
  - 標準通訊協定 FTP、HTTP、HTTPS、SMTP...
  - 偽冒通訊協定，EX.走TCP-80但並非HTTP、走TCP-443但並非HTTPS
  - 自己定義的通訊協定，必定Port大於 1024
2. 其他
  - Password Attack
  - Scan,Port Scan,網頁程式CGI Scan
  - DDoS
  - SQL-Injection  

# Analysis Network Attack
分析辦公室對**外部連線**
1. 忽略TCP-80、TCP-443，先看其他通訊
2. 看TCP-80 (HTTP在wireshark中是綠色的，先看因為至少是明文)
   ```bash
   HTTP用途 : 瀏覽網頁
   - 瀏覽器的User-Agent必定為Mozilla，而Mozilla對欄位有一定的規範
   - 欄位正常必有四項(P30,31,32)
    1. Host       : 正常不會是IP
    2. User-Agent : Mozilla
    3. Referer    : 參考來源，沒有Referer的狀況只有
                    URL直接搜尋、書籤Bookmark、網站首頁、**弱點掃描惡意程式**
    4. Accept-Language : 在台灣必定是zh-TW(繁體中文)
                         中國大陸zh-cn(簡體中文)
                         香港zh-hk(港式中文)
   ---
   HTTP用途 : 系統用途
   - 系統用途的 User-Agent 通常會有相關的內容在裡面
   ```
3. TCP-443
    雖然是加密但必定有 Client-Hello 與 Server-Hello 這兩個是明碼的

# Malware Network Analysis
惡意程式分析的事前準備工作
1. 檔案總管 > ⋯ > 選項 > 檢視 
   - 勾選 : 顯示隱藏的檔案、資料夾及磁碟機
   - 取消 : 隱藏保護的作業系統檔案(建議選項)
2. 工作管理員
   - 效能 > ⋯ > 開啟資源監視器
   - 詳細資料 > [右鍵] > 選取欄位 > 開啟 [ 映像路徑名稱 ] 、 [ 命令列 ]
3. CMD
    ```bash
    # Windows
    netstat -anop tcp

    # Linux
    sudo netstat -tupan
    ```
4. Wireshark 開始錄封包
5. 注意事項
   - 大多數的電腦中毒都會有**對外網路通訊**
   - 中毒不會導致你的CPU 100% 使用
- Analysis
  - [Lab 1 - VallyRAT.exe [ 對外傳輸 ]](./Malware%20Analysis/Lab%201%20-%20VallyRAT.exe%20[%20對外傳輸%20].md)
  - [Lab 2 - XWorm.exe [ USB感染 ]](./Malware%20Analysis/Lab%202%20-%20XWorm.exe%20[%20USB感染%20].md)

