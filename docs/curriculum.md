# 玥玥练习：计算范围和技能列表

本文说明当前实现会出哪些计算题、技能之间的前置关系、题目生成条件和输入方式。游戏整体规则见 `docs/SPEC.md`，技能定义见 `app/js/skills.js`，出题处理见 `app/js/problems.js`。

## 1. 对象和结构

本项目用 58 个技能覆盖小学 1～6 年级的数字输入型计算。范围包括四则运算、竖式、小数、分数、概数、约数和倍数、计算顺序、百分比、相等的比以及字母 `x` 的值。不包含图形、长度/容量/时间等测量、图表、应用题、汉字数字输入或数位表输入。

| 年级 | 技能数 | 主要内容 |
| --- | ---: | --- |
| 1 | 8 | 10 的分解、进位/退位、三个数、简单两位数加减 |
| 2 | 13 | 两位数和简单三位数加减竖式、乘法口诀、整十数乘一位数、1/2 和 1/4 |
| 3 | 14 | 三/四位数加减、整数乘法竖式、除法和余数、十分位小数加减、同分母分数 |
| 4 | 10 | 整数除法竖式、计算顺序、四舍五入、百分位小数加减、小数和整数乘除、带分数 |
| 5 | 8 | 小数乘除、最大公约数、最小公倍数、约分、异分母分数、分数和整数、百分比 |
| 6 | 5 | 分数乘除、小数和分数计算、相等的比、求 x |

技能树分为“加减法”“乘除法”“小数和分数”“其他”四个类别。屏幕上每个类别会拆成两列显示；视觉上相邻的技能不一定有前置关系。

## 2. 技能列表

以下表格来自当前实现。ID、生成器名称和参数是程序值，不应翻译或随意修改。

| ID | 名称 | 年级 | 类别 | 前置技能 | 生成器和参数 |
| --- | --- | --- | --- | --- | --- |
| `g1-compose10` | 凑成10 | 1年级 | 加减法 | 无 | `compose` `{"total":10}` |
| `g1-add-nc` | 一位数加法 | 1年级 | 加减法 | 无 | `hadd` `{"a":[1,9],"b":[1,9],"carry":"none"}` |
| `g1-sub-nb` | 10以内减法 | 1年级 | 加减法 | `g1-add-nc` | `hsub` `{"a":[2,10],"b":[1,9],"borrow":"none"}` |
| `g1-add-c` | 进位加法 | 1年级 | 加减法 | `g1-compose10`、`g1-add-nc` | `hadd` `{"a":[2,9],"b":[2,9],"carry":"yes"}` |
| `g1-sub-b` | 退位减法 | 1年级 | 加减法 | `g1-add-c`、`g1-sub-nb` | `hsub` `{"a":[11,18],"b":[2,9],"borrow":"yes"}` |
| `g1-add3` | 三个数的计算 | 1年级 | 加减法 | `g1-sub-b` | `add3` `{}` |
| `g1-add-2d1` | 两位数＋一位数 | 1年级 | 加减法 | `g1-add-c` | `hadd` `{"a":[11,89],"b":[1,9],"carry":"none","tensToo":true}` |
| `g1-sub-2d1` | 两位数−一位数 | 1年级 | 加减法 | `g1-sub-b`、`g1-add-2d1` | `hsub` `{"a":[11,99],"b":[1,9],"borrow":"none","tensToo":true}` |
| `g2-vadd2-nc` | 两位数加法竖式 | 2年级 | 加减法 | `g1-add-2d1` | `vadd` `{"da":2,"db":2,"carry":"none","maxDigits":2}` |
| `g2-vadd2-c` | 进位竖式加法 | 2年级 | 加减法 | `g2-vadd2-nc`、`g1-add-c` | `vadd` `{"da":2,"db":[1,2],"carry":"some","maxDigits":2}` |
| `g2-vsub2-nb` | 两位数减法竖式 | 2年级 | 加减法 | `g1-sub-2d1` | `vsub` `{"da":2,"db":2,"borrow":"none"}` |
| `g2-vsub2-b` | 退位竖式减法 | 2年级 | 加减法 | `g2-vsub2-nb`、`g1-sub-b` | `vsub` `{"da":2,"db":[1,2],"borrow":"some"}` |
| `g2-vadd3s` | 超过100的加法 | 2年级 | 加减法 | `g2-vadd2-c` | `vadd` `{"da":2,"db":2,"carry":"many","maxDigits":3}` |
| `g2-vsub3s` | 从100多里减 | 2年级 | 加减法 | `g2-vsub2-b`、`g2-vadd3s` | `vsub` `{"da":3,"db":2,"borrow":"some","aMax":199}` |
| `g2-kuku25` | 乘法口诀 5和2 | 2年级 | 乘除法 | `g1-add-c` | `kuku` `{"dans":[5,2]}` |
| `g2-kuku34` | 乘法口诀 3和4 | 2年级 | 乘除法 | `g2-kuku25` | `kuku` `{"dans":[3,4]}` |
| `g2-kuku67` | 乘法口诀 6和7 | 2年级 | 乘除法 | `g2-kuku34` | `kuku` `{"dans":[6,7]}` |
| `g2-kuku891` | 乘法口诀 8、9、1 | 2年级 | 乘除法 | `g2-kuku67` | `kuku` `{"dans":[8,9,1]}` |
| `g2-kuku-mix` | 乘法口诀混合 | 2年级 | 乘除法 | `g2-kuku891` | `kuku` `{"dans":[1,2,3,4,5,6,7,8,9]}` |
| `g2-mul-tens` | 整十数×一位数 | 2年级 | 乘除法 | `g2-kuku-mix` | `mulTens` `{}` |
| `g2-frac-of` | 1/2和1/4 | 2年级 | 小数和分数 | `g2-kuku25` | `fracOf` `{"dens":[2,4]}` |
| `g3-vadd3` | 三位数加法 | 3年级 | 加减法 | `g2-vadd3s` | `vadd` `{"da":3,"db":3,"carry":"some","maxDigits":3}` |
| `g3-vsub3` | 三位数减法 | 3年级 | 加减法 | `g2-vsub3s` | `vsub` `{"da":3,"db":[2,3],"borrow":"some"}` |
| `g3-vadd4` | 四位数加法 | 3年级 | 加减法 | `g3-vadd3` | `vadd` `{"da":4,"db":[3,4],"carry":"many","maxDigits":4}` |
| `g3-vsub4` | 四位数减法 | 3年级 | 加减法 | `g3-vsub3` | `vsub` `{"da":4,"db":[3,4],"borrow":"zero"}` |
| `g3-div-basic` | 除法 | 3年级 | 乘除法 | `g2-kuku-mix` | `div` `{"exact":true}` |
| `g3-div-rem` | 有余数除法 | 3年级 | 乘除法 | `g3-div-basic` | `divRem` `{}` |
| `g3-div-tens` | 整十数÷一位数 | 3年级 | 乘除法 | `g3-div-basic` | `divTens` `{}` |
| `g3-vmul-2x1` | 两位数×一位数竖式 | 3年级 | 乘除法 | `g2-mul-tens` | `vmul` `{"da":2,"db":1}` |
| `g3-vmul-3x1` | 三位数×一位数 | 3年级 | 乘除法 | `g3-vmul-2x1` | `vmul` `{"da":3,"db":1}` |
| `g3-vmul-2x2` | 两位数×两位数 | 3年级 | 乘除法 | `g3-vmul-2x1` | `vmul` `{"da":2,"db":2}` |
| `g3-vmul-3x2` | 三位数×两位数 | 3年级 | 乘除法 | `g3-vmul-2x2`、`g3-vmul-3x1` | `vmul` `{"da":3,"db":2}` |
| `g3-dec-add1` | 小数加法 | 3年级 | 小数和分数 | `g2-vadd2-c` | `vdec` `{"op":"add","places":1}` |
| `g3-dec-sub1` | 小数减法 | 3年级 | 小数和分数 | `g3-dec-add1`、`g2-vsub2-b` | `vdec` `{"op":"sub","places":1}` |
| `g3-frac-same` | 分数加减法 | 3年级 | 小数和分数 | `g2-frac-of` | `frac` `{"op":"addsub","same":true,"maxOne":true}` |
| `g4-vdiv-2d1` | 两位数÷一位数竖式 | 4年级 | 乘除法 | `g3-div-rem`、`g3-div-tens` | `vdiv` `{"dd":2,"ds":1}` |
| `g4-vdiv-3d1` | 三位数÷一位数 | 4年级 | 乘除法 | `g4-vdiv-2d1` | `vdiv` `{"dd":3,"ds":1}` |
| `g4-vdiv-2d2` | 两位数÷两位数 | 4年级 | 乘除法 | `g4-vdiv-2d1`、`g3-vmul-2x1` | `vdiv` `{"dd":2,"ds":2}` |
| `g4-vdiv-3d2` | 三位数÷两位数 | 4年级 | 乘除法 | `g4-vdiv-2d2`、`g4-vdiv-3d1` | `vdiv` `{"dd":3,"ds":2}` |
| `g4-order` | 计算顺序 | 4年级 | 其他 | `g2-kuku-mix`、`g2-vsub2-b` | `order` `{}` |
| `g4-round` | 概数（四舍五入） | 4年级 | 其他 | `g3-vadd4` | `round` `{}` |
| `g4-dec-add2` | 百分位小数加减法 | 4年级 | 小数和分数 | `g3-dec-sub1` | `vdec` `{"op":"addsub","places":2}` |
| `g4-dec-mul` | 小数×整数 | 4年级 | 小数和分数 | `g4-dec-add2`、`g3-vmul-2x1` | `vmul` `{"da":2,"db":1,"pa":1}` |
| `g4-dec-div` | 小数÷整数 | 4年级 | 小数和分数 | `g4-dec-mul`、`g4-vdiv-2d1` | `decDivInt` `{}` |
| `g4-frac-mixed` | 带分数加减法 | 4年级 | 小数和分数 | `g3-frac-same` | `frac` `{"op":"addsub","same":true,"mixed":true}` |
| `g5-dec-mul` | 小数×小数 | 5年级 | 小数和分数 | `g4-dec-mul` | `vmul` `{"da":2,"db":2,"pa":1,"pb":1}` |
| `g5-dec-div` | 小数÷小数 | 5年级 | 小数和分数 | `g4-dec-div`、`g5-dec-mul` | `decDivDec` `{}` |
| `g5-gcd` | 最大公约数 | 5年级 | 其他 | `g3-div-basic` | `gcdlcm` `{"kind":"gcd"}` |
| `g5-lcm` | 最小公倍数 | 5年级 | 其他 | `g5-gcd` | `gcdlcm` `{"kind":"lcm"}` |
| `g5-frac-reduce` | 约分 | 5年级 | 小数和分数 | `g5-gcd`、`g4-frac-mixed` | `frac` `{"op":"reduce"}` |
| `g5-frac-diff` | 异分母分数 | 5年级 | 小数和分数 | `g5-frac-reduce`、`g5-lcm` | `frac` `{"op":"addsub","same":false}` |
| `g5-frac-int` | 分数×÷整数 | 5年级 | 小数和分数 | `g5-frac-reduce` | `frac` `{"op":"muldivInt"}` |
| `g5-percent` | 百分率 | 5年级 | 其他 | `g4-dec-mul` | `percent` `{}` |
| `g6-frac-mul` | 分数×分数 | 6年级 | 小数和分数 | `g5-frac-int` | `frac` `{"op":"mul"}` |
| `g6-frac-div` | 分数÷分数 | 6年级 | 小数和分数 | `g6-frac-mul` | `frac` `{"op":"div"}` |
| `g6-frac-dec` | 小数和分数的计算 | 6年级 | 小数和分数 | `g6-frac-div`、`g5-dec-div` | `frac` `{"op":"decimal"}` |
| `g6-ratio` | 相等的比 | 6年级 | 其他 | `g5-lcm` | `ratio` `{}` |
| `g6-letter` | 求x | 6年级 | 其他 | `g4-order` | `letter` `{}` |

## 3. 题目生成条件

### 横式整数计算

`compose` 把 10 分成 1～9 和剩余数。`hadd` 的 `carry: none` 表示无进位，`yes` 表示至少一次进位。`hsub` 总是保证被减数大于减数；`borrow: none` 表示无退位，`yes` 表示至少一次退位。

`tensToo: true` 会在普通“两位数和一位数”之外，按一定概率生成整十数之间的加减。`add3` 使用 1～9 的三个整数，前一步和最终结果都限制在适合一年级的范围。`kuku` 使用指定乘法口诀段，`mulTens` 使用整十数乘一位数。

### 竖式

加法竖式的 `carry: some` 表示至少一次进位，`many` 表示两次以上进位，并可用 `maxDigits` 限制结果位数。减法竖式保证结果为正；`borrow: some` 表示至少一次退位，`zero` 还会优先生成跨 0 退位的题。

乘法竖式按 `da`、`db` 指定被乘数和乘数位数。小数乘法用 `pa`、`pb` 指定小数位数。除法竖式用 `dd`、`ds` 指定被除数和除数位数，既可能有余数，也可能整除。

### 小数和分数

`vdec` 会把小数整数化后生成加减竖式。`decDivInt` 和 `decDivDec` 生成横式小数除法，目前答案为一位小数或整数，不做除法竖式中间步骤输入。

分数题会使用固定答案格式判定。三年级同分母分数保留原分母，不自动约分。其他分数计算使用生成时准备好的既约分数或带分数格式。玩家不能输入任意等价表达式。

### 其他计算

`order` 生成含括号或乘加减混合的计算顺序题。`round` 生成十位、百位、千位四舍五入。`gcdlcm` 生成最大公约数和最小公倍数。`percent` 使用常见百分比和基数，保证答案为正整数。`ratio` 生成等比缺项，`letter` 生成 `x×a=b`、`x＋a=b`、`x−a=b` 三类求 x 题。

## 4. 输入和提示

横式答案从左到右输入。加减和乘法竖式从最低位开始；除法竖式从商的上方开始，随后输入减完的余数和下一个商。小数点、进位、退位、部分乘积、下落数字等会在适当时自动显示。

分数按分母、分子输入；带分数先输入整数部分。同一格多次答错时，系统会高亮参考数字并显示提示，提示可能包含中间式、乘法表、通分分母等。

## 5. 复习和时间胶囊

题目的标题和式子组成签名，用于避免短期重复。技能的最近题目和当前游玩中的题目都会参与去重；如果多次尝试仍无法避开重复，则使用最后生成的候选。复习题和时间胶囊题会直接复用保存的题目。
