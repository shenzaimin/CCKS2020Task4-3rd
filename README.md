# CCKS-2020-Task4
面向金融领域的小样本跨类迁移事件抽取 第三名 方案及代码
> ## 我们在 CCKS2020 大会分享的 技术方案PPT分享[于此](https://pan.baidu.com/s/1ulJXAMVZua7lcHy8K57ZMQ)提取码：bfd1 

## 环境安装
> 依赖文件路径code/conda.txt  和  code/pip.txt
### 1.conda创建python==3.8环境以及依赖包:  conda env create -f conda.txt
### 2.Pip安装依赖包： pip install -r pip.txt
### 重要说明：经过试验表明，环境的匹配要完全一致，否则会影响结果的复现

## 硬件条件：GPU显存11G及以上（eg. Nivdia 2080Ti）
耗时（eg. Nvdia 2080Ti）：
分类模块:10-15 min
事件抽取模型（esemble）: 6-7 hour
## 1.执行分类模型：
训练集数据路径：code/CCKS-Cls/dataset/trans_train.json
测试集数据路径：code/CCKS-Cls/dataset/trans_test.json
> cd /code/CCKS-Cls/
> sh classification.sh
得到分类结果文件:/code/CCKS-Cls/test_output/cls_out_single.csv

> 说明：chinese_roberta_wwm_large_ext_pytorch 预训练模型文件路径 code/CCKS-Cls/pretrained_model/Bert-wwm-ext/
下载链接：
http://pan.iflytek.com/#/link/9B46A0ABA70C568AAAFCD004B9A2C773
提取密码：43eH


## 2.执行事件抽取
训练集数据路径：code/data/train/trans_train.json
以及code/data/train/train_base.json(这个是A榜的训练集，需要加入进来，作为预训练模型的”权重学习资料”)
测试集数据路径：code/data/dev/trans_test.json
> cd /code/
> sh aug.sh
根目录得到结果文件 /code/result.json 
### 事件抽取执行步骤
### 1.chinese_roberta_wwm_large_ext_pytorch 预训练模型文件路径 code/chinese_roberta_wwm_ext_pytorch/
> 下载链接：
http://pan.iflytek.com/#/link/9B46A0ABA70C568AAAFCD004B9A2C773
提取密码：43eH


### 2.迁移权重学习：首先transfer_train_roberta_model_aug.py会利用trans_train.json和train_base.json中所有数据进行预训练模型（roberta）的权重学习，并保存预训练模型的权重参数，作为后续模型训练的基础预训练模型（roberta）权重，参与到后续的学习

### 3.基础子模型训练：基于第二步学到的预训练权重，transfer_train_roberta_model_ensemble.py依据每个事件抽取框架会生成10个基本模型，存放路径为
> code/saved_model_roberta_db_1_1/esemble
> code/saved_model_roberta_pj_1_1/esemble
> code/saved_model_roberta_qsht_1_1/esemble
> code/saved_model_roberta_sg_1_1/esemble
> code/saved_model_roberta_zb_1_1/esemble

### 4.投票预测：采用投票基于上述esemble模型进行每个事件的集成预测，生成结果文件result_tmp.json(存放路径为/code/result_tmp.json)

### 5.统计学修正：code/config/reader.py 中的fix_trigger()方法会使用规则进行各事件的修正（修正原则为统计各事件中trigger在训练集（A榜和B榜）里的集合，对测试集的模型推断结果进行”查漏补缺”）最终生成result.json为提交的测试集最终结果文件。
