﻿暂时这样凑合着看，之后有时间再补充文字说明。[微笑]

@[TOC]

# 爬取文本数据集
`download_text_data.py`文件：
```python
import os
import random
import requests
import json
import time
```

```python
# 分类新闻参数
news_classify = [
    [0, '民生', 'news_story'],
    [1, '文化', 'news_culture'],
    [2, '娱乐', 'news_entertainment'],
    [3, '体育', 'news_sports'],
    [4, '财经', 'news_finance'],
    [5, '房产', 'news_house'],
    [6, '汽车', 'news_car'],
    [7, '教育', 'news_edu'],
    [8, '科技', 'news_tech'],
    [9, '军事', 'news_military'],
    [10, '旅游', 'news_travel'],
    [11, '国际', 'news_world'],
    [12, '证券', 'stock'],
    [13, '农业', 'news_agriculture'],
    [14, '游戏', 'news_game']
]
```


```python
# 已经下载的新闻标题的ID
downloaded_data_id = []
# 已经下载新闻标题的数量
downloaded_sum = 0


def get_data(tup, data_path):
    global downloaded_data_id
    global downloaded_sum
    print('============%s============' % tup[1])
    url = "http://it.snssdk.com/api/news/feed/v63/"
    # 分类新闻的访问参数，模仿正常网络访问
    t = int(time.time() / 10000)
    t = random.randint(6 * t, 10 * t)
    querystring = {"category": tup[2], "max_behot_time": t, "last_refresh_sub_entrance_interval": "1524907088",
                   "loc_mode": "5",
                   "tt_from": "pre_load_more", "cp": "51a5ee4f38c50q1", "plugin_enable": "0", "iid": "31047425023",
                   "device_id": "51425358841", "ac": "wifi", "channel": "tengxun", "aid": "13",
                   "app_name": "news_article", "version_code": "631", "version_name": "6.3.1",
                   "device_platform": "android",
                   "ab_version": "333116,297979,317498,336556,295827,325046,239097,324283,170988,335432,332098,325198,336443,330632,297058,276203,286212,313219,328615,332041,329358,322321,327537,335710,333883,335102,334828,328670,324007,317077,334305,280773,335671,319960,333985,331719,336452,214069,31643,332881,333968,318434,207253,266310,321519,247847,281298,328218,335998,325618,333327,336199,323429,287591,288418,260650,326188,324614,335477,271178,326588,326524,326532",
                   "ab_client": "a1,c4,e1,f2,g2,f7", "ab_feature": "94563,102749", "abflag": "3", "ssmix": "a",
                   "device_type": "MuMu", "device_brand": "Android", "language": "zh", "os_api": "19",
                   "os_version": "4.4.4", "uuid": "008796762094657", "openudid": "b7215ea70ca32066",
                   "manifest_version_code": "631", "resolution": "1280*720", "dpi": "240",
                   "update_version_code": "6310", "_rticket": "1524907088018", "plugin": "256"}

    headers = {
        'cache-control': "no-cache",
        'postman-token': "26530547-e697-1e8b-fd82-7c6014b3ee86",
        'User-Agent': 'Dalvik/1.6.0 (Linux; U; Android 4.4.4; MuMu Build/V417IR) NewsArticle/6.3.1 okhttp/3.7.0.2'
    }

    # 进行网络请求
    response = requests.request("GET", url, headers=headers, params=querystring)
    # 获取返回的数据
    new_data = json.loads(response.text)
    with open(data_path, 'a', encoding='utf-8') as fp:
        for item in new_data['data']:
            item = item['content']
            item = item.replace('\"', '"')
            item = json.loads(item)
            # 判断数据中是否包含id和新闻标题
            if 'item_id' in item.keys() and 'title' in item.keys():
                item_id = item['item_id']
                print(downloaded_sum, tup[0], tup[1], item['item_id'], item['title'])
                # 通过新闻id判断是否已经下载过
                if item_id not in downloaded_data_id:
                    downloaded_data_id.append(item_id)
                    # 安装固定格式追加写入文件中
                    line = u"{}_!_{}_!_{}_!_{}".format(item['item_id'], tup[0], tup[1], item['title'])
                    line = line.replace('\n', '').replace('\r', '')
                    line = line + '\n'
                    fp.write(line)
                    downloaded_sum += 1
```


```python
def get_routine(data_path):
    global downloaded_sum
    # 从文件中读取已经有的数据，避免数据重复
    if os.path.exists(data_path):
        with open(data_path, 'r', encoding='utf-8') as fp:
            lines = fp.readlines()
            downloaded_sum = len(lines)
            for line in lines:
                item_id = int(line.split('_!_')[0])
                downloaded_data_id.append(item_id)
            print('在文件中已经读起了%d条数据' % downloaded_sum)
    else:
        os.makedirs(os.path.dirname(data_path))

    while 1:
        # 　开始下载数据
        time.sleep(10)
        for classify in news_classify:
            get_data(classify, data_path)
        # 当下载量超过300000就停止下载
        if downloaded_sum >= 300000:
            break
```


```python
if __name__ == '__main__':
    data_path = 'datasets/news_classify_data.txt'
    dict_path = "datasets/dict_txt.txt"
    # 下载数据集
    get_routine(data_path)
```

输出信息：
```
============文化============
17 1 文化 6646565189942510093 世界第一豪宅，坐落于北京，一根柱子27个亿，世界首富都买不起！
18 1 文化 6658382232383652104 俗语讲：“男怕初一，女怕十五”，这话什么意思？有道理吗？
19 1 文化 6636596124998173192 浙江一员工请假条火了，内容令人狂笑不止，字迹却让人念念不忘
20 1 文化 6658848073562718734 难怪悟空被赶下山后菩提神秘消失，你看看方寸山门口对联写了啥？
21 1 文化 6658952207871771140 他把183件国宝无偿捐给美国，捐回中国却收了450万美元
```


# 制作训练数据
`create_data.py`文件：
```python
import os

def create_data_list(data_root_path):
    with open(data_root_path + 'test_list.txt', 'w') as f:
        pass
    with open(data_root_path + 'train_list.txt', 'w') as f:
        pass

    with open(os.path.join(data_root_path, 'dict_txt.txt'), 'r', encoding='utf-8') as f_data:
        dict_txt = eval(f_data.readlines()[0])

    with open(os.path.join(data_root_path, 'news_classify_data.txt'), 'r', encoding='utf-8') as f_data:
        lines = f_data.readlines()
    i = 0
    for line in lines:
        title = line.split('_!_')[-1].replace('\n', '')
        l = line.split('_!_')[1]
        labs = ""
        if i % 10 == 0:
            with open(os.path.join(data_root_path, 'test_list.txt'), 'a', encoding='utf-8') as f_test:
                for s in title:
                    lab = str(dict_txt[s])
                    labs = labs + lab + ','
                labs = labs[:-1]
                labs = labs + '\t' + l + '\n'
                f_test.write(labs)
        else:
            with open(os.path.join(data_root_path, 'train_list.txt'), 'a', encoding='utf-8') as f_train:
                for s in title:
                    lab = str(dict_txt[s])
                    labs = labs + lab + ','
                labs = labs[:-1]
                labs = labs + '\t' + l + '\n'
                f_train.write(labs)
        i += 1
    print("数据列表生成完成！")
```


```python
# 把下载得数据生成一个字典
def create_dict(data_path, dict_path):
    dict_set = set()
    # 读取已经下载得数据
    with open(data_path, 'r', encoding='utf-8') as f:
        lines = f.readlines()
    # 把数据生成一个元组
    for line in lines:
        title = line.split('_!_')[-1].replace('\n', '')
        for s in title:
            dict_set.add(s)
    # 把元组转换成字典，一个字对应一个数字
    dict_list = []
    i = 0
    for s in dict_set:
        dict_list.append([s, i])
        i += 1
    # 添加未知字符
    dict_txt = dict(dict_list)
    end_dict = {"<unk>": i}
    dict_txt.update(end_dict)
    # 把这些字典保存到本地中
    with open(dict_path, 'w', encoding='utf-8') as f:
        f.write(str(dict_txt))

    print("数据字典生成完成！")
```


```python
# 获取字典的长度
def get_dict_len(dict_path):
    with open(dict_path, 'r', encoding='utf-8') as f:
        line = eval(f.readlines()[0])

    return len(line.keys())
```


```python
if __name__ == '__main__':
    # 把生产的数据列表都放在自己的总类别文件夹中
    data_root_path = "datasets/"
    data_path = os.path.join(data_root_path, 'news_classify_data.txt')
    dict_path = os.path.join(data_root_path, "dict_txt.txt")
    # 创建数据字典
    create_dict(data_path, dict_path)
    # 创建数据列表
    create_data_list(data_root_path)
```

输出信息：
```
数据字典生成完成！
数据列表生成完成！
```

# 定义模型
`bilstm_net.py`文件：
```python
import paddle.fluid as fluid

def bilstm_net(data, dict_dim, class_dim, emb_dim=128, hid_dim=128, hid_dim2=96, emb_lr=30.0):
    # embedding layer
    emb = fluid.layers.embedding(input=data,
                                 size=[dict_dim, emb_dim],
                                 param_attr=fluid.ParamAttr(learning_rate=emb_lr))

    # bi-lstm layer
    fc0 = fluid.layers.fc(input=emb, size=hid_dim * 4)

    rfc0 = fluid.layers.fc(input=emb, size=hid_dim * 4)

    lstm_h, c = fluid.layers.dynamic_lstm(input=fc0, size=hid_dim * 4, is_reverse=False)

    rlstm_h, c = fluid.layers.dynamic_lstm(input=rfc0, size=hid_dim * 4, is_reverse=True)

    # extract last layer
    lstm_last = fluid.layers.sequence_last_step(input=lstm_h)
    rlstm_last = fluid.layers.sequence_last_step(input=rlstm_h)

    # concat layer
    lstm_concat = fluid.layers.concat(input=[lstm_last, rlstm_last], axis=1)

    # full connect layer
    fc1 = fluid.layers.fc(input=lstm_concat, size=hid_dim2, act='tanh')
    # softmax layer
    prediction = fluid.layers.fc(input=fc1, size=class_dim, act='softmax')
    return prediction
```

# 定义数据读取
`text_reader.py`文件：
```python
from multiprocessing import cpu_count
import numpy as np
import paddle
```


```python
# 训练数据的预处理
def train_mapper(sample):
    data, label = sample
    data = [int(data) for data in data.split(',')]
    return data, int(label)
```


```python
# 训练数据的reader
def train_reader(train_list_path):

    def reader():
        with open(train_list_path, 'r') as f:
            lines = f.readlines()
            # 打乱数据
            np.random.shuffle(lines)
            # 开始获取每张图像和标签
            for line in lines:
                data, label = line.split('\t')
                yield data, label

    return paddle.reader.xmap_readers(train_mapper, reader, cpu_count(), 1024)
```


```python
# 测试数据的预处理
def test_mapper(sample):
    data, label = sample
    data = [int(data) for data in data.split(',')]
    return data, int(label)
```


```python
# 测试数据的reader
def test_reader(test_list_path):

    def reader():
        with open(test_list_path, 'r') as f:
            lines = f.readlines()
            for line in lines:
                data, label = line.split('\t')
                yield data, label
```

# 训练模型
`train.py`文件：
```python
import os
import shutil
import paddle
import paddle.fluid as fluid
import create_data
import text_reader
import bilstm_net
```


```python
# 定义输入数据， lod_level不为0指定输入数据为序列数据
words = fluid.layers.data(name='words', shape=[1], dtype='int64', lod_level=1)
label = fluid.layers.data(name='label', shape=[1], dtype='int64')
```


```python
# 获取数据字典长度
dict_dim = create_data.get_dict_len('datasets/dict_txt.txt')
# 获取长短期记忆网络
model = bilstm_net.bilstm_net(words, dict_dim, 15)
```


```python
# 获取损失函数和准确率
cost = fluid.layers.cross_entropy(input=model, label=label)
avg_cost = fluid.layers.mean(cost)
acc = fluid.layers.accuracy(input=model, label=label)

# 获取预测程序
test_program = fluid.default_main_program().clone(for_test=True)

# 定义优化方法
optimizer = fluid.optimizer.AdagradOptimizer(learning_rate=0.002)
opt = optimizer.minimize(avg_cost)

# 创建一个执行器，CPU训练速度比较慢
# place = fluid.CPUPlace()
place = fluid.CUDAPlace(0)
exe = fluid.Executor(place)
# 进行参数初始化
exe.run(fluid.default_startup_program())
```


```python
# 获取训练和预测数据
train_reader = paddle.batch(reader=text_reader.train_reader('datasets/train_list.txt'), batch_size=128)
test_reader = paddle.batch(reader=text_reader.test_reader('datasets/test_list.txt'), batch_size=128)
```


```python
# 定义输入数据的维度
feeder = fluid.DataFeeder(place=place, feed_list=[words, label])

# 开始训练
for pass_id in range(10):
    # 进行训练
    for batch_id, data in enumerate(train_reader()):
        train_cost, train_acc = exe.run(program=fluid.default_main_program(),
                             feed=feeder.feed(data),
                             fetch_list=[avg_cost, acc])

        if batch_id % 40 == 0:
            print('Pass:%d, Batch:%d, Cost:%0.5f, Acc:%0.5f' % (pass_id, batch_id, train_cost[0], train_acc[0]))
            # 进行测试
            test_costs = []
            test_accs = []
            for batch_id, data in enumerate(test_reader()):
                test_cost, test_acc = exe.run(program=test_program,
                                              feed=feeder.feed(data),
                                              fetch_list=[avg_cost, acc])
                test_costs.append(test_cost[0])
                test_accs.append(test_acc[0])
            # 计算平均预测损失在和准确率
            test_cost = (sum(test_costs) / len(test_costs))
            test_acc = (sum(test_accs) / len(test_accs))
            print('Test:%d, Cost:%0.5f, ACC:%0.5f' % (pass_id, test_cost, test_acc))
```


```python
    # 保存预测模型
    save_path = 'infer_model/'
    # 删除旧的模型文件
    shutil.rmtree(save_path, ignore_errors=True)
    # 创建保持模型文件目录
    os.makedirs(save_path)
    # 保存预测模型
    fluid.io.save_inference_model(save_path, feeded_var_names=[words.name], target_vars=[model], executor=exe)
```

# 预测文本
`infer.py`文件：
```python
import numpy as np
import paddle.fluid as fluid
```


```python
# 创建执行器
place = fluid.CPUPlace()
exe = fluid.Executor(place)
exe.run(fluid.default_startup_program())

# 保存预测模型路径
save_path = 'infer_model/'
# 从模型中获取预测程序、输入数据名称列表、分类器
[infer_program, feeded_var_names, target_var] = fluid.io.load_inference_model(dirname=save_path, executor=exe)
```


```python
# 获取数据
def get_data(sentence):
    # 读取数据字典
    with open('datasets/dict_txt.txt', 'r', encoding='utf-8') as f_data:
        dict_txt = eval(f_data.readlines()[0])
    dict_txt = dict(dict_txt)
    # 把字符串数据转换成列表数据
    keys = dict_txt.keys()
    data = []
    for s in sentence:
        # 判断是否存在未知字符
        if not s in keys:
            s = '<unk>'
        data.append(int(dict_txt[s]))
    return data
```


```python
data = []
# 获取图片数据
data1 = get_data('京城最值得你来场文化之旅的博物馆')
data2 = get_data('谢娜为李浩菲澄清网络谣言，之后她的两个行为给自己加分')
data.append(data1)
data.append(data2)
```


```python
# 获取每句话的单词数量
base_shape = [[len(c) for c in data]]

# 生成预测数据
tensor_words = fluid.create_lod_tensor(data, base_shape, place)
```


```python
# 执行预测
result = exe.run(program=infer_program,
                 feed={feeded_var_names[0]: tensor_words},
                 fetch_list=target_var)
```


```python
# 分类名称
names = ['民生', '文化', '娱乐', '体育', '财经',
         '房产', '汽车', '教育', '科技', '军事',
         '旅游', '国际', '证券', '农业', '游戏']

# 获取结果概率最大的label
for i in range(len(data)):
    lab = np.argsort(result)[0][i][-1]
    print('预测结果标签为：%d， 名称为：%s， 概率为：%f' % (lab, names[lab], result[0][i][lab]))
```
