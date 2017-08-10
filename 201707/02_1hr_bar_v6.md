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
        input:mode_1(0),mode_2(0),mode_3(0),mode_4(0);
        var:LastTradingDay(0);
        LastTradingDay=iff(DAYofMonth(Date) > 14 and DAYofMonth(Date) < 22 and DAYofWeek(Date)=3,1,0);
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
![R02V1001](.\pics\R02V1001.png)
只開mode_1:基本上和原作相同
![R02V1002](.\pics\R02V1002.png)
只開mode_2:
![R02V1003](.\pics\R02V1003.png)
mode_3只有一個方向,沒有出場,沒法回測
只開mode_4:
![R02V1004](.\pics\R02V1004.png)

---

### 研究不同的mode對mode_1的影響
![R02V1008](.\pics\R02V1008.png)

- mode_2對mode_1貢獻了70筆交易, 這在17年裡可說是非常少, 
- mode_2對mode_1的ROA提升, 貢獻很多
- 

