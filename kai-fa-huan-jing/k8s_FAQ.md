## - 如何解出app使用的secret

kubectl get secret scada-allsvc-secret -o yaml

每個namespace的多合一secret名稱不一定相同, "scada-allsvc-secret1" 不是固定的

## - 如何使用還沒進版的node module## 
## - 別人幫我佈好app了, 但我想要調整資源(例如 cpu ram), 怎麼辦?
edit deployment

## - 別人跟我要"chart 檔案" 怎辦
helm pull到lock成壓縮檔

## - 舊版的chart不見了, 怎麼辦, 可以重包嗎
可以, 

1. 先去helm chart build 專案, 找到需要的tag, 開一個branch
2. 到jenkins pipeline, 改branch, 改成剛剛拉出來的
3. 手動觸發pipeline, 要填版號
