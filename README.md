# Sparkify 项目

## 内容清单

1.[所需环境](#installation)

2.[项目简介](#introduction)

3.[项目文件](#files)

## 所需环境 <a name="installation"></a>
本项目运行的环境需要安装以下程序：
* `python3`:运行python3依赖的lib库文件
* `pandas`:python pandas依赖的lib库文件
* `numpy`:python numpy依赖的lib库文件
* `pyspark`:python 运行spark依赖库，如果您在个人电脑上运行spark，还需要安装spark相关库文件
* `Jupyter Notebook`: 用于运行项目包含的.ipynb文件
* `mini_sparkify_event_data.json`:项目使用的实验数据集

## 项目简介 <a name="introduction"></a>
此项目是优达学城纳米学位的毕业项目，仅用于学习使用，不应用于商业用途，以下描述的具体实现可以在项目文件`Sparkify-zh.ipynb`中找到

### 概述
数字音乐服务公司（如网易云音乐、QQ音乐）通过网页、桌面应用和移动应用为用户提供在线音乐服务。用户使用服务过程中，会产生诸如听歌、添加歌单、交友、点击广告、升级VIP和注销等庞大的行为日志。此项目旨在通过抽取日志相关性特征来预测客户流失情况。

完整的数据集大小为 12GB，通过分析完整数据集的子集（小数据集），使用Spark集群的时候，把成果迁移到大数据集上。

### 问题概述
1.把数据集加载到 Spark 上，并使用 Spark SQL 和 Spark 数据框来操作数据

2.对数据进行整理

3.对数据进行探索性分析

4.提取相关性特征

5.在 Spark ML 中使用机器学习 API 来搭建和调整模型

### 衡量指标
流失用户相对于整体用户的基数非常小，使用F1分数作为优化指标。实现精度和召回率平衡的模型


## 加载和清洗数据

### 评估
数据集共有286500条记录，包含的特征如下：

- `artist`:创作歌曲的艺术家名字
- `auth`:记录登录状态
- `firstName`:用户名
- `gender`:性别
- `itemInSession`:当前会话中id
- `lastName`:用户姓
- `length`:歌曲长度
- `level`:区分用户付费还是免费
- `location`:用户地理位置
- `method`:数据上行还是下行
- `page`:用户访问的页面
- `registration`:注册时间
- `sessionId`:会话id
- `song`:歌曲名
- `status`:用户访问页面的http状态码
- `ts`:日志流水时间
- `userAgent`用户访问使用的浏览器信息
- `userId`用户id

### 清洗
对数据集进行整理

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

2.提取与客户流失有较强相关的特征绘制图表观察

3.从图表分析结果中最终确定用于机器学习的特征

### 探索数据
对数据进行与用户流失相关的探索性分析


## 特征工程
最终选定做为机器学习的特征

## 建模

**最终需要解决的问题是一个二分类问题**

模型选择决定采用逻辑回归和决策树先进行尝试，如果预测效果不好，再选择其他模型

### 建模结论
通过对比模型性能得出最终结论

## 项目文件 <a name="files"></a>
* `Sparkify-zh.ipynb` 项目可执行文件
* `blog.md` 项目分析的博客 [这里](https://github.com/chenqin-km/nanodegrees/blob/master/blog.md)
