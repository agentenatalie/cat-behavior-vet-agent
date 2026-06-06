# 示例 Consult：就诊压力

这是一个虚构、缩短版示例，只展示回答结构。它不是针对真实动物的兽医建议。

## 结论

这个 case 需要在下次常规就诊前建立低压力 handling 方案；是否使用就诊前药物应由兽医决定。猫在车里喘和尿在猫包里提示 distress 很高，所以方案重点应放在减少可预测触发，而不是到诊所后强行保定。

## 处理方案

- 把猫包长期放在家里，当作家具的一部分；里面放熟悉垫子和零食。
- 先在猫包附近喂高价值零食，再逐步移到猫包内，之后才短暂关门。
- 做短时、非就诊性质的乘车练习：猫包进车、不开引擎；开引擎；短距离行驶；回家。
- 提前和诊所沟通 cat-friendly 安排：安静候诊、遮盖猫包、减少犬类暴露、温和 handling，压力升级时暂停。
- 如果训练不足以支持必要就诊，或就诊很紧急，和兽医讨论 pre-visit medication。
- 避免 flooding、强行拖出猫、惩罚，或在猫恐慌时反复长途练车。

## 需要检索的科学证据

本地检索可以使用：

```bash
python3 scripts/search_corpus.py "cat stress veterinary visit handling transport" -n 8
python3 scripts/search_corpus.py "gabapentin cats veterinary examination stress" -n 8
python3 scripts/search_corpus.py "preappointment medications reduce fear anxiety cats veterinary visits" -n 8
```

相关公开 provenance metadata 包括：

- van Haaften et al. (2017), *Effects of a single preappointment dose of gabapentin on signs of stress in cats during transportation and veterinary examination*, Journal of the American Veterinary Medical Association, PMID:29099247.
- Rodan (2010), *Understanding feline behavior and application for appropriate handling and management*, Topics in Companion Animal Medicine, PMID:21147470.
- Erickson et al. (2021), *A review of pre-appointment medications to reduce fear and anxiety in dogs and cats at veterinary visits*, Canadian Veterinary Journal, PMID:34475580.
- Mariti et al. (2022), *Happy cats: stress in cats and their carers associated with outpatient visits to the clinic*, Journal of Feline Medicine and Surgery, PMID:36322402.

## 真实案例对照

有 web access 时，可以搜索：

```text
cat carrier training vet visit stress owner report
cat gabapentin vet visit stress case report
```

真实案例只总结实践模式，和科学证据分开呈现。
