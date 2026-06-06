# 示例 Consult：窗外猫触发攻击

这是一个虚构、缩短版示例，只展示回答结构。它不是针对真实动物的兽医建议。

## Intake 追问

完整评估前，skill 应该先问：

1. 最近是否做过体检、疼痛检查、牙科检查和泌尿问题排查？
2. 咬伤是否需要就医？事件后是否已经安全隔离？
3. 攻击前具体身体语言是什么：僵住、甩尾、炸毛、低吼、瞳孔放大、盯视或 stalking？
4. 现在能否暂时遮挡或封锁这扇窗？
5. 最近有没有食欲、饮水、排泄、活动、梳毛或皮肤变化？

## 结论

这个模式最像由窗外猫触发的 redirected aggression，但疼痛或医学原因导致的易激惹仍然是重要鉴别方向。因为咬伤破皮且已经复发，优先级是防止再次受伤、阻断触发源，并安排兽医检查排除医学问题。

## 处理方案

- 人被猫咬破皮要认真处理，必要时就医。
- 暂时阻断窗边触发：遮住低处窗面、拉上百叶窗、贴可移除窗膜，或在户外猫活跃时关闭该区域。
- 事件后冷静隔离，不要抱起、惩罚、吼叫或追赶猫。
- 在远离触发窗的位置增加室内 enrichment：食物玩具、固定玩耍时间、高处休息点和稳定日程。
- 安静一段时间后，再做低阈值 desensitization / counterconditioning。一旦出现凝视、身体僵硬、低吼或接近攻击阈值就停止。
- 预约兽医检查，排查疼痛、泌尿问题、皮肤病、牙痛、神经变化和药物/补充剂影响。
- 如果攻击频率或强度上升、伤害继续发生、恢复时间变长，或另一只猫不安全，应升级到兽医行为专科。

## 需要检索的科学证据

本地检索可以使用：

```bash
python3 scripts/search_corpus.py "owner-directed aggression in cats redirected aggression" -n 8
python3 scripts/search_corpus.py "human-directed aggression cats differential diagnosis management" -n 8
python3 scripts/search_corpus.py "stress in owned cats behavioural changes welfare" -n 8
```

相关公开 provenance metadata 包括：

- Amat et al. (2019), *Common feline problem behaviours: Owner-directed aggression*, Journal of Feline Medicine and Surgery, PMID:30798644.
- Frank and Dehasse (2003), *Differential diagnosis and management of human-directed aggression in cats*, Veterinary Clinics of North America: Small Animal Practice, PMID:12701512.
- Amat et al. (2016), *Stress in owned cats: behavioural changes and welfare implications*, Journal of Feline Medicine and Surgery, PMID:26101238.
- Stelow (2018), *Diagnosing Behavior Problems: A Guide for Practitioners*, Veterinary Clinics of North America: Small Animal Practice, PMID:29429600.

## 真实案例对照

有 web access 时，可以搜索：

```text
cat redirected aggression outdoor cat window owner bitten
cat aggression after seeing outdoor cat through window resolved
```

这些公开真实案例只能作为 anecdotal implementation patterns，不能当作机制或疗效证据。
