# Sparkify 项目

## 项目概览


数字音乐服务公司（如网易云音乐、QQ音乐）通过网页、桌面应用和移动应用为用户提供在线音乐服务。用户使用服务过程中，会产生诸如听歌、添加歌单、交友、点击广告、升级VIP和注销等庞大的行为日志。此项目旨在通过抽取日志相关性特征来预测客户流失情况。

我认为在线音乐服务的产品逻辑：
- 用户需要注册一个账号成为普通用户，可以点播平台提供的免费歌曲，平台向用户展示广告，赚取收入用于平台运营
- 使用初期平台为用户推荐平台热门歌曲和用户，用户可以把歌曲加入歌单、评价歌曲和交友。通过社交行为增加用户粘性和提升平台活跃度
- 使用一段时间以后，平台为用户推荐有相似听歌行为的用户和分类歌曲
- 用户建立了使用喜好和社交圈后，平台向用户展示需要升级成为VIP用户才可以点播的歌曲（如明星艺术家和无损音乐）和特权（社交虚拟道具），吸引用户成为付费用户
- 出于多种原因，付费用户可能从付费降级为免费，也有可能从免费升级为付费，最糟糕的就是注销，那么用户将从平台流失，这是平台要极力避免的

预测客户流失率是数据科学家和分析师在面向消费者的一类公司中经常遇到的一项具有挑战性的问题。还有，能用 Spark 高效处理大数据集是数据领域职位急需的一种能力。

完整的数据集大小为 12GB，通过分析完整数据集的子集（小数据集），使用Spark 集群的时候，把成果迁移到大数据集上。

## 问题概述
1.把数据集加载到 Spark 上，并使用 Spark SQL 和 Spark 数据框来操作数据

2.对数据进行整理

3.对数据进行探索性分析

4.提取相关性特征

5.在 Spark ML 中使用机器学习 API 来搭建和调整模型

## 衡量指标
流失用户相对于整体用户的基数非常小，使用F1分数作为优化指标。实现精度和召回率平衡的模型


## 加载和清洗数据

### 评估
数据集共有286500条记录，包含的特征如下：

- `artist`:创作歌曲的艺术家名字，有**228108**条记录
- `auth`:记录登录状态,数据完整无缺失
- `firstName`:用户名,有**2278154**条记录
- `gender`:性别,有**2278154**条记录
- `itemInSession`:当前会话中id,数据完整无缺失
- `lastName`:用户姓,有**2278154**条记录
- `length`:歌曲长度,有**2228108**条记录
- `level`:区分用户付费还是免费，数据完整无缺失
- `location`:用户地理位置,有**2278154**条记录
- `method`:数据上行还是下行,数据完整无缺失
- `page`:用户访问的页面,数据完整无缺失
- `registration`:注册时间,有**2278154**条记录
- `sessionId`:会话id,数据完整无缺失
- `song`:歌曲名,有**2228108**条记录
- `status`:用户访问页面的http状态码,数据完整无缺失
- `ts`:日志流水时间,数据完整无缺失
- `userAgent`用户访问使用的浏览器信息,数据完整无缺失
- `userId`用户id,数据完整无缺失

从观测值来看，数据分为两个维度：

- 从获取到的用户信息来看，有2278154条记录，而userId数据又是完整的，意味着有userId是无效的
- 从获取到的歌曲信息来看，有228108条记录，这个是可以接受，毕竟用户不是每一次行为都是听歌

### 质量
- `userId`列没有空值，但存在**空字符串**,为无效的用户ID
- `auth`、`level`、`method`、`page`、`itemInSession`、`sessionId`、`status`、`ts`列没有空值，数据完整
- `userAgent`,`registration`,`location`,`firstName`,`lastName`,`gender`列数据存在缺失值，条数都为278154条
- `artist`、`song`、`length`存在缺失值，条数都为228108

### 清洗
- 去除`userId`为空字符串的数据：清洗完`userId`列为空值的数据后，数据条数与`userAgent`,`registration`,`location`,`firstName`,`lastName`,`gender`列一致了

## 探索性数据分析
用户访问页面的主要行为有：
- `About`:点击应用的说明
- `Add Friend`:添加好友
- `Add to Playlist`: 添加播放歌单
- `Cancel`:查看注销页
- `Cancellation Confirmation`:确定注销
- `Downgrade`:查看降级页
- `Error`:访问错误
- `Help`:查看帮助页
- `Home`:查看主页
- `Logout`:登出
- `NextSong`:播放下一首歌曲
- `Roll Advert`:滚动广告
- `Save Settings`:保存设置
- `Settings`:访问设置
- `Submit Downgrade`:确定降级
- `Submit Upgrade`:确定升级
- `Thumbs Down`:撤下当前头像
- `Thumbs Up`:上传头像
- `Upgrade`:查看升级页

### 探索思路：
1.使用 Cancellation Confirmation 行为来定义客户流失

2.提取我认为的与客户流失有较强相关的特征绘制图表观察

3.从图表分析结果中最终确定用于机器学习的特征

### 探索数据
从观察数据集来看，每一条日志记录了用户的一个动作，我决定采用page特征做为最主要的探索对象，将page中的重要动作提取出来进行独热编码

编码完成后，数据集增加了12个特征：

1.`friend`:用户的交友行为

2.`addplaylist`:用户添加歌单的行为

3.`cancel`:用户想要注销的行为

4.`downgrade`:用户想要降级的行为

5.`error`:用户访问发生错误的行为

6.`help`:用户访问帮助页面的行为

7.`logout`:用户登出的行为

8.`upgrade`:用户想要升级的行为

9.`upgraded`:用户确定升级的事件

10.`downgraded`:用户确定降级的事件

11.`advert`:用户点击广告的行为

12.`lv`:标识用户付费还是免费

按选取的特征按用户分组，汇总所有的行为和事件


![image](https://github.com/chenqin-km/nanodegrees/blob/master/images/statistic_data.png)


在此过程中，我发现听歌的事件在数量级上远大于其他事件，决定废弃听歌事件，以免对其他特征干扰太大

#### 探索免费用户的行为


![image](https://github.com/chenqin-km/nanodegrees/blob/master/images/free_users.png)
![image](https://github.com/chenqin-km/nanodegrees/blob/master/images/free_cancel_users.png)


从免费用户的统计图表来看：
- 广告点击事件`advert`的占比远高于其他事件
- 流失客户交友`friend`的占比低于存留用户
- 流失客户想要注销`cancel`的占比高于存留客户


#### 探索付费用户的行为


![image](https://github.com/chenqin-km/nanodegrees/blob/master/images/paid_users.png)
![image](https://github.com/chenqin-km/nanodegrees/blob/master/images/paid_churn_users.png)

从付费用户的统计图表来看：
- 广告`advert`对用户的干扰占比很小，添加歌曲`addplaylist`和交友`friend`占比变高起来

![image](https://github.com/chenqin-km/nanodegrees/blob/master/images/paid_down_users.png)


从付费用户发生降级行为的统计图表来看：
- 点击降级页面的占比高于整体付费用户水平
- 最终发生降低的占比高于整体付费用户水平

#### 探索用户升级的行为

![image](https://github.com/chenqin-km/nanodegrees/blob/master/images/upgraded_users.png)


- 从最终升级为付费用户的免费用户来看，想要升级的动作占比比较高

### 结论
- 广告是影响免费用户使用的一个关键特征
- 用户想要发生某种事件时，点击相关页面的占比比较高
- 添加歌单的行为在整体行为中占比高，我决定不使用其作为机器学习模型的特征


## 特征工程
最终选定做为机器学习特征如下：

1.`friend`:用户的交友行为

2.`cancel`:用户想要注销的行为

3.`downgrade`:用户想要降级的行为

4.`error`:用户访问发生错误的行为

5.`help`:用户访问帮助页面的行为

6.`upgrade`:用户想要升级的行为

7.`upgraded`:用户确定升级的事件

8.`advert`:用户点击广告的行为

9.`lv`:标识用户付费还是免费

除了`lv`特征保留1和0（1标识用户付费 0标识用户免费），其余特征按用户分组后，按时间顺序进行相加，做为关键特征的值

想要达到的效果如：2018年1月1日用户点击广告的日志，`adverts`特征计数为1，2018年1月2日用户点击广告的日志，`adverts`特征计数为2。其他特征也如此

处理完成后，对特征进行归一化，最终创建一个叫做`scaledfeatures2`的向量，用于机器学习

![image](https://github.com/chenqin-km/nanodegrees/blob/master/images/model_data.png)


## 建模

**最终需要解决的问题是一个二分类问题**

模型选择决定采用逻辑回归和决策树先进行尝试，如果预测效果不好，再选择其他模型

- 逻辑回归模型:调整的参数先尝试`regParam`，使用0和0.1进行网格搜索

- 决策树模型：调整的参数先尝试`maxDepth`，使用5和10进行网格搜索

流失用户相对于整体用户的基数非常小，使用F1分数作为优化指标。实现精度和召回率平衡的模型

操作步骤如下：

1.将完整数据集分成训练集、测试集和验证集

2.使用逻辑回归模型进行测试，得到f1分数和学习时间

3.使用决策树模型进行测试，得到f1分数和学习时间

4.比较两种模型的准确率，如果两种模型的f1分数低于0.9，尝试选择其他模型；如果效果较好，比较两个模型的学习效率

![image](https://github.com/chenqin-km/nanodegrees/blob/master/images/lr.png)

![image](https://github.com/chenqin-km/nanodegrees/blob/master/images/dt.png)

### 建模结论
1.逻辑回归模型的f1分数为0.9997，验证集预测时间用时50.8秒

2.决策树模型的f1分数为0.9996，验证集预测时间用时50.1秒

3.两个模型都非常棒，几乎没有差别，因此我决定使用逻辑回归做为本项目使用的最终模型
