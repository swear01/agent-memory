---
title: 1005 有限域學習進度與易混淆概念
scope: projects/error-correcting-codes / 1005 finite fields tutoring
status: reviewed-by-user-not-assessed
updated: 2026-10-20
---

# 學習進度

- 來源為本次逐步問答與老師的 `Symbol1/Correct2026/1005.ipynb`。使用者於 2026-10-20 表示「我讀的差不多了」，要求更新 QMD。
- 已討論 field 定義、prime subfield、prime-power cardinality、basis 與係數、quotient ring、extension field、splitting field、root bound、Frobenius、GF(4) 符號與矩陣表示，以及 irreducible / primitive polynomial。
- 此狀態表示使用者自述已大致讀過本次有限域內容；尚未以獨立作答或測驗確認熟練度，不表示整門課、後續講義或作業已完成。
- 下次從使用者的新問題接續；需要複習時先用具體算例確認，不預設必須把 field 定義從頭重講。

# 定義與推理的接續點

- 此課的 field 譯為「域／體」。非零元素必須有留在同一集合的乘法反元素；整數沒有 2 的整數反元素，所以不是 field。
- 有限域 F 的 characteristic 必為質數 p；由 1 反覆相加生成 prime subfield F_p，F 是其上的有限維向量空間，因此 |F|=p^n。這排除任何 6-element field，不只是排除整數 mod 6 的運算。
- F_p 的 p 是質數，F_(p^n) 的總元素數則是質數冪；n=1 是有限 prime field 的情形。一般的 prime field 也包括 characteristic 0 的 Q，不能把「prime field 只能有限」當成定義。
- pZ 是 p 的所有整數倍。Z/pZ 的元素是餘數類，斜線表示取商，不是普通除法。
- Field 公理不要求所有書寫方式唯一。固定 basis 後，每個元素的係數座標才唯一：spanning 保證存在，linear independence 保證唯一。
- Quotient ring R/I 將差屬於 ideal I 的元素視為同一類，以 [a]+[b]=[a+b]、[a][b]=[ab] 運算；取商後不一定是 field。
- Extension field 必須包含原域並保留其運算。Splitting field 是讓指定多項式完全分解成一次因子、由原域和全部根生成的最小擴張域；只加入一個根不一定足夠。
- Root bound：整域上的非零 d 次多項式至多有 d 個不同根，不要求恰好 d 個。證明直接依賴沒有零因子；Z 雖非 field 仍適用。反例為 Z/8Z 中 x^2-1 有根 1,3,5,7。有限整域則必為 field。

# GF(4) 算術與兩種表示

- GF(4)=F_4=F_2[x]/(x^2+x+1)，與 Z/4Z 的運算不同。令 alpha=[x]，則 alpha^2=alpha+1；alpha 不是普通整數 2。
- 元素為 0,1,alpha,1+alpha，各自唯一寫成 a+b alpha，其中 a,b 屬於 F_2。係數就是乘在 basis 元素 1、alpha 前面的數。
- 加減法為係數 mod 2；乘法展開後用 alpha^2=alpha+1 化簡；alpha(alpha+1)=1，所以 alpha^(-1)=alpha+1。alpha+1/alpha=1，但 (alpha+1)/alpha=alpha；先確認括號與變數是不是 alpha。
- 老師採 row-vector 慣例。對應矩陣 A=[[0,1],[1,1]]，所有 entries 在 F_2 運算，A^2=A+I；a+b alpha 對應 aI+bA=[[a,b],[b,a+b]]。
- 只有 0,I,A,I+A 這四個矩陣代表這個 F_4，不是所有 2x2 矩陣。符號表示與矩陣表示保持同樣的加法、乘法。

# Frobenius 與 x^q-x 的根

- 在 characteristic p 中，p 為質數，二項式展開中間的 binomial coefficients 都是 p 的倍數，因此 (a+b)^p=a^p+b^p。反覆 m 次，令 q=p^m，得到 (a+b)^q=a^q+b^q。
- 這是 Frobenius map a -> a^p 保持加法的性質，不是實數中的一般乘方規則。
- 證明根集合封閉時有兩個不同依據：Frobenius 給出 (a+b)^q=a^q+b^q；前提 g(a)=g(b)=0、g(x)=x^q-x 才給出 a^q=a、b^q=b。因此 g(a+b)=0。
- 不可把 a^q=a 當作任意 splitting-field 元素的規則。對 x^q-x，先在 splitting field 中完全分解，再以 formal derivative -1 非零證明無重根，才得到恰好 q 個不同的根；確認根集合的 field 封閉性後得到大小 q 的有限域。

# Irreducible 與 primitive polynomial

- 不可約性要指定係數域。F_p 上 n 次不可約 f 使 F_p[x]/(f) 成為大小 p^n 的 field；代表元素可唯一寫成次數小於 n 的多項式。
- Primitive polynomial 在此有限域語境中要求不可約，且其根 alpha 的 multiplicative order 為 p^n-1；alpha 的冪才能遍歷全部非零元素。不可約保證可建構 field，但不保證選取的根本身是乘法群生成元。
- x^2+x+1 在 F_2 上不可約且 primitive：alpha 的階為 3；到了 F_4 則分解成 (x-alpha)(x-(alpha+1))。
- x^2+1 在 F_3 上不可約，能建構 F_9；其根 beta 滿足 beta^2=2、beta^4=1，階為 4，小於 8，所以不是 primitive polynomial。
- x^2+1 在 F_2 上卻等於 (x+1)^2，是可約的；其商環有非零 alpha+1 滿足 (alpha+1)^2=0，不能稱為 F_4。

# 教學偏好

- 使用繁體中文、保留標準英文術語；從具體元素與算例銜接抽象定義，拆開每個等號依據，尤其是 Frobenius 與根的代換。
- 不把同一域的不同表示混成不同結構，不把數學 field 轉去解釋物理場。疑似語音誤辨的「舉證」先標示可能指矩陣。
- 詳細教學與精簡作業是不同交付：這份記錄只保存學習脈絡，不改動既有作業、期中小抄或講義。
