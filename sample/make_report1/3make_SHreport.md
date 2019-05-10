会議用資料
================
2019-05-09

# SH会議用の出力

``` r
source("../../rvpa1.9.2.r")
source("../../future2.1.r")
source("../../utilities.r",encoding="UTF-8") # ggplotを使ったグラフ作成用の関数

options(scipen=100) # 桁数表示の調整(1E+9とかを抑制する)

library(tidyverse)

theme_SH <- function(){
    theme_bw(base_size=12) +
    theme(panel.grid = element_blank(),
          axis.text.x=element_text(size=11,color="black"),
          axis.text.y=element_text(size=11,color="black"),
          axis.line.x=element_line(size= 0.3528),
          axis.line.y=element_line(size= 0.3528),
          legend.position="none")
}

ggsave_SH <- function(...){
    ggsave(width=150,height=85,dpi=600,units="mm",...)
}

ggsave_SH_large <- function(...){
    ggsave(width=150,height=120,dpi=600,units="mm",...)
}

## 再生産関係のプロット(x,yの単位やスケール,ラベルを入れたい年を指定してください)
(g1_SRplot <- SRplot_gg(SRmodel.base,
                        xscale=1000,xlabel="千トン",
                        yscale=1,   ylabel="尾",
                        labeling.year=c(1990,2000,2010,2017),
                        add.info=FALSE) + theme_SH())
```

![](3make_SHreport_files/figure-gfm/unnamed-chunk-2-1.png)<!-- -->

``` r
ggsave_SH("g1_SRplot.png",g1_SRplot)

## yield curve
# プロットする管理基準値だけ取り出す
refs.plot <- dplyr::filter(refs.base,RP.definition%in%c("Btarget0","Blimit0","Bban0"))
```

![](3make_SHreport_files/figure-gfm/unnamed-chunk-2-2.png)<!-- -->

``` r
# プロットする
(g2_yield_curve <- plot_yield(MSY.base$trace,
                              refs.plot,
                              refs.label=c("目標管理基準値","限界管理基準値","禁漁水準"),
                              future=list(future.default),
                              past=res.pma,labeling=FALSE,
                              refs.color=c(1,1,1), # 印刷が出ないので縦線の色は黒にすることに
                              biomass.unit=1000,#資源量の単位
                              AR_select=FALSE,
                              xlim.scale=0.4,ylim.scale=1.3 # x, y軸の最大値の何倍までプロットするか。ラベルやyield curveの形を見ながら適宜調整してください
                              ) + theme_SH())
```

![](3make_SHreport_files/figure-gfm/unnamed-chunk-2-3.png)<!-- -->

``` r
ggsave_SH("g2_yield_curve.png",g2_yield_curve)

## kobe plot

# 縦軸はFとする
# MSYを達成するときのSPRを計算する
(SPR.msy <- calc_MSY_spr(MSY.base,max.age=Inf,Fmax=10))
```

    [1] 25.43032

``` r
# SPR.msyを目標としたとき、それぞれのF at age by yearを何倍すればSPR.msyを達成できるか計算
SPR.history <- get.SPR(res.pma,target.SPR=round(SPR.msy),max.age=Inf,Fmax=10)$ysdata
Fratio <- SPR.history$"F/Ftarget"

# SPRの時系列と目標SPR
plot(SPR.history$perSPR,ylim=c(0,max(c(SPR.history$perSPR,SPR.msy))),type="b")
abline(h=SPR.msy,lty=2)
```

![](3make_SHreport_files/figure-gfm/unnamed-chunk-2-4.png)<!-- -->

``` r
(g3_kobe4_F <- plot_kobe_gg(res.pma,refs.base,roll_mean=1,category=4,
                            Blow="Btarget0", Btarget="Btarget0",
                            beta=0.8, # 推奨されるβに変える
                            refs.color=c(1,1,1),
                            yscale=1.2, # y軸を最大値の何倍まで表示するか。ラベルの重なり具合を見ながら調整してください
                            HCR.label.position=c(1,1),# HCRの説明を書くラベルの位置。相対値なので位置を見ながら調整してください。
                            ylab.type="F",Fratio=Fratio)+theme_SH())
```

![](3make_SHreport_files/figure-gfm/unnamed-chunk-2-5.png)<!-- -->

``` r
ggsave_SH("g3_kobe4_F.png",g3_kobe4_F)

## 将来予測の図
# 親魚資源量と漁獲量の時系列の図示
(g4_future <- plot_futures(res.pma, #vpaの結果
                   list(future.Fcurrent,future.default), # 将来予測結果
                   future.name=c("s1","s2"),
                   CI_range=c(0.1,0.9),
                   maxyear=2045,
                   ncol=1, # 図の出力の列数。3行x1列ならncol=1
                   what.plot=c("biomass","SSB","catch"),
                   Btarget=derive_RP_value(refs.base,"Btarget0")$SSB,
                   Blimit=derive_RP_value(refs.base,"Blimit0")$SSB,
                   Bban=derive_RP_value(refs.base,"Bban0")$SSB,
                   RP_name=c("目標管理基準値","限界管理基準値","禁漁水準"),
                   biomass.unit=1000,  # バイオマスの単位(100, 1000, or 10000トン)
                   n_example=5,seed=2)+ # どのシミュレーションをピックアップするかはseedの値を変えて調整してください
    theme_SH()+
    theme(legend.position="right")+
    scale_color_hue(labels=c(VPA="過去の推定値",s1="現状の漁獲圧",s2="漁獲管理規則\n(β=0.8)"))+
    scale_fill_hue(labels=c(VPA="過去の推定値",s1="現状の漁獲圧",s2="漁獲管理規則\n(β=0.8)"))+
    scale_linetype_discrete(guide=FALSE)
)
```

![](3make_SHreport_files/figure-gfm/unnamed-chunk-2-6.png)<!-- -->

``` r
ggsave_SH_large("g4_future.png",g4_future)
```
