- 如何解出app使用的secret

kubectl get secret scada-allsvc-secret -o yaml

每個namespace的多合一secret名稱不一定相同, "scada-allsvc-secret1" 不是固定的

- 如何使用還沒進版的node module
- 別人幫我佈好app了, 但我想要調整資源(例如 cpu ram), 怎麼辦?