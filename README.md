# stitching
about img stitching

#总体思路

## 前提
---
1. 图片拍摄是以一个点为中心，延一条线逐步向外扩散进行。
2. 拍摄结果是一些列同心圆环，且环上的图片分布是均匀的。
3. 摄像头拍摄的图片本身没有形变，不会因摄像头形变造成拼接误差。
4. 图像最终是圆中取一个正方形，因此会有许多图片丢弃。这意味着拍摄图片要足够大。图像裁剪暂没有考虑。
## 整体流程
---
### 思路
以中心点为圆心，逐步进行拼接。考虑到当前拼接主要是左右拼接，因此，会根据不同位置，进行旋转后再拼接。
### 流程图
#### 整体
暂时没找到实现换图的语法。
具体看[这里](https://share.weiyun.com/5ySsiso)吧。
``` flow
st=&gt;start: 总流程
wenjian=&gt;subroutine: 获取文件数量并检查
zhongxin=&gt;operation: 获取中心点(0,0)
zhongxinhuxian=&gt;subroutine: 生成竖线
you=&gt;subroutine: 生成中心线右侧图片
zuo=&gt;subroutine: 生成中心线左侧图片
quantu=&gt;subroutine: 组合生成全图
fencengtu=&gt;subroutine: 生成分层图片
youC=&gt;condition: 完成&#63;
zuoC=&gt;condition: 完成&#63;
e=&gt;end
st-&gt;wenjian-&gt;zhongxin-&gt;zhongxinhuxian-&gt;you-&gt;youC
youC(yes)-&gt;zuo-&gt;zuoC
youC(no)-&gt;you
zuoC(yes)-&gt;quantu-&gt;fencengtu-&gt;e
zuoC(no)-&gt;zuo
```
#### 竖线
``` flow
st=&gt;start: 竖线流程
zhongxin=&gt;operation: 获取中心点(0,0)
zhongxinXuanZhuan=&gt;operation: 中心点顺时针旋转90度
you=&gt;subroutine: 向右（上）拼接图片
zuo=&gt;subroutine: 向左（下）拼接图片
youC=&gt;condition: 完成&#63;
zuoC=&gt;condition: 完成&#63;
zhongxinQuan=&gt;operation: 拼接中心图片
zhongxinQuanZhuan=&gt;operation: 中心点顺时针旋转270度(逆90)
e=&gt;end
st-&gt;zhongxin-&gt;zhongxinXuanZhuan-&gt;you-&gt;youC
youC(yes)-&gt;zuo-&gt;zuoC
youC(no)-&gt;you
zuoC(yes)-&gt;zhongxinQuan-&gt;zhongxinQuanZhuan-&gt;e
zuoC(no)-&gt;zuo
```
#### 生成一个点的图片（x,y）

``` flow
st=&gt;start: 生成一个逻辑点的图片
cankao=&gt;operation: 获取参考图片(x+1orx-1,y-1到y+1)
根据当前是往左生成还是往右判断，
考虑图片完整性，y也多取上下各一张
tupianQuyu=&gt;operation: 根据逻辑位置x,y获取图片范围
即左上到右下的范围
yuanshtuliebiao=&gt;operation: 根据图片的范围，获取相关的原始图片列表
也就是有可能覆盖的取值范围的图片。
为了防止空白，取值范围会适当放大（分别增加n像素）
重新拼接的话，范围会进一步扩大
xuanzhuan=&gt;operation: 原始图片旋转
pinjie=&gt;subroutine: 从列表中抽取一张进行拼接
pinjieTu=&gt;operation: 生成新的拼接图
pinjieC=&gt;condition: 完成？

jiequ=&gt;operation: 截取理论位置(x,y)的图片
jiequC=&gt;condition: 完整无空缺？
wancheng=&gt;operation: 保存理论位置(x,y)的图片
e=&gt;end
st-&gt;cankao-&gt;tupianQuyu-&gt;yuanshtuliebiao-&gt;xuanzhuan-&gt;pinjie-&gt;pinjieTu-&gt;pinjieC
pinjieC(yes)-&gt;jiequ-&gt;jiequC
pinjieC(no)-&gt;pinjie
jiequC(yes)-&gt;wancheng-&gt;e
jiequC(no)-&gt;tupianQuyu
```

## 遗留问题
---
1. 形变误差积累放到问题。因为从开始进行全景拼接就会造成误差，且误差会因为拼接次数过多而变得不可接受。例如，假设每次拍摄变形时会随机造成正负0.1的误差，那么在极端情况下，第二次就会是0.2，第三次是0.3，后续就会形变到不能接受的地步。
2. 图片整合。目前拼接的思路是先存成每个点的小图片，最后在同一整合成为一张大的图片。图片最终是什么类型、多大仍没有确定。
3. 图片展示。已有了集中方案，仍没有确认使用何种形式。
