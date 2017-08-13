# 紅黑棒1HR突破多空對翻策略
- 原作: http://study.godnavi.com.tw/index.php?s=85&pid=85

---- 
## 基本設置

- TXF原始資料是2001/1/1~2017/5/9, 但因為我策略屬性設置了MaxBarBack=500, 以下的績效展示區間都是從2001/5/10~2017/5/9
- 來回成本700, 即單邊設350
- K線:1小時

---
### 基本回測

- 原作使用"#return"的語法, 我的MC莫名的無法使用, 所以改寫切換參數
- 原作有四個不同的進場, 將其設定為mode_1,mode_2,mode_3,mode_4
- 個人比較看重的是 資金報酬率=Profit/(MDD+保証金), 但是MC報告裡沒有這個數值, 但是它有一個數值叫"帳戶報酬(ROA)", 雖然算出來有點差異, 基本上和資金報酬率成正比, 所以都會拉出來看
- 修改後的程式碼如下:

        input:dis(10),delay_bars(1);
        input:mode_1(0),mode_2(0),mode_3(0),mode_4(0),mp(0);
        var:LastTradingDay(0);
        LastTradingDay=iff(DAYofMonth(Date) > 14 and DAYofMonth(Date) < 22 and DAYofWeek(Date)=3,1,0);
        mp=marketposition;
        //############# mode_1 #######################
        if mode_1=1 then begin
            if  (barssinceentry>delay_bars or marketposition=0) and c-o>dis  then buy("B") next bar at H stop;
            if  (barssinceentry>delay_bars or marketposition=0) and o-c>dis  then sellshort("S") next bar at L stop;
        end;
        //############# mode_2 #######################
        if mode_2=1 then begin
            if d>d[1] then begin
                if close-open>0 and absvalue(close-open)>10 and close<close[1] then buy("B2") next bar at market;
                if close-open<0 and absvalue(close-open)>10 and close>close[1] then sellshort("S2") next bar at market;
            end;
        end;
        //############# mode_3 #######################
        if mode_3=1 then begin
            if open-close>40 then sellshort("R40") next bar at market;
        end;
        //############# mode_4 #######################
        if mode_4=1 then begin
            if barssinceentry>5 and close cross over averagefc(close,100) and close-averagefc(close,100)>10 then buy("B4") next bar at market;
            if barssinceentry>5 and close cross under averagefc(close,100) and averagefc(close,100)-close>10 then sellshort("S4") next bar at market;
        end;
        //############# setexitonclose #######################
        if LastTradingDay=1 then setexitonclose;

4個策略都打開的績效:基本上和原作相同
![R02V6001](.\pics\R02V6001.png)
只開mode_1:基本上和原作相同
![R02V6002](.\pics\R02V6002.png)
只開mode_2:
![R02V6003](.\pics\R02V6003.png)
mode_3只有一個方向,沒有出場,沒法回測
只開mode_4:
![R02V6004](.\pics\R02V6004.png)

---

### 研究不同的mode對mode_1的影響
![R02V6008](.\pics\R02V6008.png)

- mode_2對mode_1貢獻了70筆交易, 這在17年裡可說是非常少, 
- mode_2對mode_1的ROA提升, 貢獻很多
- mode_3貢獻很多交易次數, MDD沒有減少, ROA沒有增加, 只是對系統穩定性有點幫助
- mode_4就對mode_1沒什麼幫助的樣子

----

### 研究不同的mode對互相的關係
![R02V6012](.\pics\R02V6012.png)
  
- 看不出什麼所以然

---

### mode_1修改

- 由於發現有幾次在結算日倒數第二根進場,最後一根就出場,造成不必要的進出
- 所以增加濾網如下
- 結果減少了40次左右的交易,績效沒什麼影響

        condition1= LastTradingDay=1 and time<1230;
        condition2= LastTradingDay=0;
        condition3= condition1 or condition2;
        if mode_1=1 then begin
            if (barssinceentry>delay_bars or marketposition=0) and condition3 and Close-Open>dis 
                then buy("B") next bar at H stop;
            if (barssinceentry>delay_bars or marketposition=0) and condition3 and  Close-Open<-1*dis 
                then sellshort("S") next bar at L stop; 
        end;

---

### mode_2修改

- mode_2基本邏輯是在開盤第一根看有紅K打勾就去買進, 有黑K反勾就去賣出
- 把它修改成在開盤第二根也去看打勾現象
- 更名叫mode_2A

        if mode_2A=1 then begin
            if time=0900 and condition3 then begin
                if close-open>0 and absvalue(close-open)>10 and close<close[1] 
                    then buy("B2A") next bar at market;
                if close-open<0 and absvalue(close-open)>10 and close>close[1] 
                    then sellshort("S2A") next bar at market;
            end;
            if time=1000 then begin
                if close-open[1]>0 and condition3 and absvalue(close-open[1])>10 and close<close[2] 
                    then buy("B2A1") next bar at market;
                if close-open[1]<0 and absvalue(close-open[1])>10 and close>close[2] 
                    then sellshort("S2A1") next bar at market;
            end;
        end;

![R02V6014](.\pics\R02V6014.png)
- mode_2A和mode_2對比起來有明顯的績效提升
- 交易次數多了200次

![R02V6016](.\pics\R02V6016.png)
- [1+2A]也有提升, 但是MDD變大, ROA減少
- 觀察這個MDD是在2008年附近產生的連損, 2A的交易次數變多, 在當時產生的連損也比較多
- 交易次數提升100
- 最後還是判斷2A好一點

---
### mode_3修改
- mode_3暫時先不做修改, 直接加進去做比對
![R02V6017](.\pics\R02V6017.png)

---
### mode_4修改

- mode_4的精神是close和均線的差距,
- 經過一連串嚐試, 得到以下的程式碼
- 更名為mode_4A

        if mode_4A=1 then begin
             //if barssinceentry>5 and close cross over averagefc(close,100) and close-averagefc(close,100)>10 then buy("MA B") next bar at market;
             //if barssinceentry>5 and close cross under averagefc(close,100) and averagefc(close,100)-close>10 then sellshort("MA S") next bar at market; 

             value40=averagefc(c,200);
             condition40=condition3 and barssinceentry>5;
             if (Close-Open)>dis and absvalue(c-value40)>50 then begin
                 EntryTrigger4A=1;
                 //if c>=value40[0] then buy("B4") next bar at H+10 stop;
                 //buy("B4") next bar at H stop;
             end;

             if (Close-Open)<-1*dis and absvalue(c-value40)>50 then begin
                 EntryTrigger4A=-1;
                 //if c<=value40[0] then sellshort("S4") next bar at L-10 stop;
                 //sellshort("S4") next bar at L stop;
             end;

             if EntryTrigger4A=1 and EntryTrigger4A[1]=1 and condition40 then buy ("B41") next bar at value40 stop;
             if EntryTrigger4A=-1 and EntryTrigger4A[1]=-1 and condition40 then sellshort ("S41") next bar at value40 stop; 
             
        end;


![R02V6018](.\pics\R02V6018.png)
- 績效明顯提升
![R02V6019](.\pics\R02V6019.png)
- 不使用mode_3, 績效更好一點, 但是MDD變大, ROA減少, 還是使用mode_3比較好

---

# 總結

- 最後檢視發現mode_1和其他進出場有打架現象, 所以加了一個mp<>1, mp<>-1, 
- 果然濾掉一些不合理的進場後, ROA有所提升
![R02V6020](.\pics\R02V6020.png)
- 最終的程式碼如下

        input:dis(10),delay_bars(1);
        input:mode_1(0),mode_1A(0),mode_2(0),mode_2A(0),mode_3(0),mode_4(0),mode_4A(0);
        var:LastTradingDay(0),mp(0),EntryTrigger4A(0);
        LastTradingDay=iff(DAYofMonth(Date) > 14 and DAYofMonth(Date) < 22 and DAYofWeek(Date)=3,1,0);
        mp=marketposition;
        condition1= LastTradingDay=1 and time<1230;
        condition2= LastTradingDay=0;
        condition3= condition1 or condition2;
        //############# mode_1 #######################
        if mode_1=1 then begin
            if (barssinceentry>delay_bars or marketposition=0) and condition3 and Close-Open>dis 
                then buy("B") next bar at H stop;
            if (barssinceentry>delay_bars or marketposition=0) and condition3 and  Close-Open<-1*dis 
                then sellshort("S") next bar at L stop; 
        end;

        //############# mode_1 #######################
        if mode_1A=1 then begin
            if (barssinceentry>delay_bars or marketposition=0) and condition3 and Close-Open>dis and mp<>1
                then buy("B1") next bar at H stop;
            if (barssinceentry>delay_bars or marketposition=0) and condition3 and  Close-Open<-1*dis and mp<>-1
                then sellshort("S1") next bar at L stop; 
        end;
        //############# mode_2 #######################
        if mode_2=1 then begin
            if d>d[1] then begin
                if close-open>0 and absvalue(close-open)>10 and close<close[1] then buy("B2") next bar at market;
                if close-open<0 and absvalue(close-open)>10 and close>close[1] then sellshort("S2") next bar at market;
            end;
        end;
        //############# mode_2A #######################
        if mode_2A=1 then begin
            if time=0900 and condition3 then begin
                if close-open>0 and absvalue(close-open)>10 and close<close[1] 
                    then buy("B2A") next bar at market;
                if close-open<0 and absvalue(close-open)>10 and close>close[1] 
                    then sellshort("S2A") next bar at market;
            end;
            if time=1000 then begin
                if close-open[1]>0 and condition3 and absvalue(close-open[1])>10 and close<close[2] 
                    then buy("B2A1") next bar at market;
                if close-open[1]<0 and absvalue(close-open[1])>10 and close>close[2] 
                    then sellshort("S2A1") next bar at market;
            end;
        end;
        //############# mode_3 #######################
        if mode_3=1 then begin
            if open-close>40 then sellshort("S3") next bar at market;
        end;
        //############# mode_4 #######################
        if mode_4=1 then begin
            if barssinceentry>5 and close cross over averagefc(close,100) and close-averagefc(close,100)>10 then buy("B4") next bar at market;
            if barssinceentry>5 and close cross under averagefc(close,100) and averagefc(close,100)-close>10 then sellshort("S4") next bar at market;
        end;

        //############# mode_4A #######################
        if mode_4A=1 then begin

             value40=averagefc(c,200);
             condition40=condition3 and barssinceentry>5;
             if (Close-Open)>dis and absvalue(c-value40)>50 then begin
                 EntryTrigger4A=1;
                 //if c>=value40[0] then buy("B40") next bar at H+10 stop;
                 //buy("B40") next bar at H stop;
             end;

             if (Close-Open)<-1*dis and absvalue(c-value40)>50 then begin
                 EntryTrigger4A=-1;
                 //if c<=value40[0] then sellshort("S40") next bar at L-10 stop;
                 //sellshort("S40") next bar at L stop;
             end;

             if EntryTrigger4A=1 and EntryTrigger4A[1]=1 and condition40 then buy ("B41") next bar at value40 stop;
             if EntryTrigger4A=-1 and EntryTrigger4A[1]=-1 and condition40 then sellshort ("S41") next bar at value40 stop; 
             
         end; 
        //############# setexitonclose #######################
        if LastTradingDay=1 then setexitonclose;
