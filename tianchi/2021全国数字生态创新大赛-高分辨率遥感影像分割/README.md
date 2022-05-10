# [2021全国数字生态创新大赛-高分辨率遥感影像分割](https://tianchi.aliyun.com/competition/entrance/531860/introduction)

## 2/7更新
很多朋友复现不出38+效果，这里做一个原因说明:
- 因为epoch 30还不够， 原始代码已更正，对于T_0=3, T_mult=2,的cosinscheduler,在44的时候学习率到达最低点，验证也会到达最高。

- 推荐一个涨点神器，参考SWA Object Detection。操作很简单，十来行代码，自行思考，暂不开源。

还有很多参数可调节，我并没有精调参，请大家自行调参，譬如将优化器换成sgd，然后T_0=2, T_mult=2,max_epoch=70，模型会在64/65左右达到最优。(效果应该会好于adamW)

## 当前实验结果
- backnone: efficientb6
- model: unet++
- 0.8/0.2 训练/验证数据线上37+(线下51)
- 全数据**38.51**(无TTA)
- dice-loss+softCrossEntropy联合Loss
- optimize: adamW,SGD均可
- warmUpConsineScheduler
- 8G显存的单卡即可训练:256输入,时间<5h

由于前段时间误操作导致测试集里面混入了一些错误的图片，导致自己分数一直在25左右徘徊，发现bug后直接涨10+个点。这个题赛程太长了，开源大家一起学习吧。不像mmseg封装得不好修改，这份代码利于自己魔改。这份代码也是参考自前段时间华为云卫星图像分割的选手分享，文末有给出。
efficientUnet++ 线上38.51方案开源(单模，无TTA)
[代码地址](https://github.com/DLLXW/data-science-competition)

**后续可上分操作**：
- 换更大backbone
- 换模型(个人认为unet还是永远滴神)
- 数据增强
- 类别不均衡处理
- TTA
- 多尺度训练/测试
- 模型融合

所以在此repo基础上上40应该不难。这么多trick没用。
## 1.赛题描述
    本赛题基于不同地形地貌的高分辨率遥感影像资料，希望参赛者能够利用遥感影像智能解译技术识别提取土地覆盖和利用类型，实现生态资产盘点、土地利用动态监测、水环境监测与评估、耕地数量与监测等应用。结合现有的地物分类实际需求，参照地理国情监测、“三调”等既有地物分类标准，设计陆域土地覆盖与利用类目体系，包括：林地、草地、耕地、水域、道路、城镇建设用地、农村建设用地，工业用地、构筑物、裸地
**类别不均衡的语义分割问题!!!**

## 2.数据处理
原始标注:

{
  1: "耕地",
  2: "林地",
  3: "草地",
  4: "道路",
  5: "城镇建设用地",
  6: "农村建设用地",
  7: "工业用地",
  8: "构筑物"
  9: "水域"
  10: "裸地"
 }

背景类不需要(同时标注无背景类)，所以训练的时候，最好当作10类来训，就是每一个原始标注都要减1，所以对应0-9的标注，预测的时候再+1即可。由于原始输入是.tif，这里可以选择四通道输入，但我这里只用到了RGB通道。

对于本开源代码，为了方便跑通,建议:将原始mask数据整体-1保存，或者在dataloader里面-1，对于.tif数据也事先转化为.jpg保存。根据如下格式进行组织数据
```
├── satellite_data
│   ├── ann_dir
│   └── img_dir
├── satellite_jpg
│   ├── ann_dir
│   └── img_dir
```
## 代码运行说明
### 环境:
- torch>1.6,(因为使用了自动混合精度训练。如果<1.6，自行将混合精度训练那部分代码注释掉即可)
- segmentation_models_pytorch
- pytorch_toolbelt
### 运行
数据处理(可自行灵活处理)
```
python tif_jpg.py
python make_datasets.py
```
训练
```shell
python train.py
```
预测
```shell
python infer.py
```
infer里面use_demo=True,可以可视化一张预测图片，若为False,则为生成提交结果

[代码参考](https://github.com/InchSoup/HWCC2020_RS_segmentati)
