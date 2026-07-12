MARS-FL README
MARS-FL：基于多锚点可信参考的鲁棒联邦移动群智感知方法
说明：代码中部分文件仍保留 DSAC 或 V3Stable 命名，它们对应本文方法 MARS-FL 的早期实现名称。
1. 方法概述
MARS-FL 面向 Non-IID 数据分布和模型投毒攻击下的联邦移动群智感知鲁棒聚合问题。其核心思想是更加充分地利用服务器端少量可信感知数据。不同于只构造单一参考更新的参考引导式方法，MARS-FL 构造多个服务器端可信锚点，包括一个全局均衡锚点和若干类别组锚点，用于刻画异构良性客户端可能存在的多样化更新方向。
2. 环境安装
方式一：使用 environment.yaml 创建完整 conda 环境
cd FLPoison
conda env create -f environment.yaml
conda activate torchenv

方式二：如果已经安装 Python/PyTorch，可手动补充依赖
pip install pyyaml matplotlib scipy scikit-learn hdbscan rarfile unrar
3. 实验配置
FedAvg 相关实验配置文件位于 configs 目录下。论文实验常用配置包括：
• configs/FedAvg_MNIST_config.yaml
• configs/FedAvg_FashionMNIST_config.yaml
• configs/FedAvg_CIFAR10_config.yaml
配置项	默认设置
algorithm	FedAvg
datasets	MNIST, FashionMNIST, CIFAR10
models	MNIST/FashionMNIST 使用 lenet，CIFAR10 使用 resnet18
distribution	non-iid
dirichlet_alpha	0.5
epochs	100
num_clients	50
num_adv	0.40
batch_size	64
learning_rate	0.01
local_epochs	5
MARS-FL 主要参数：
参数	默认值
num_anchors	5
num_sample	500
dirichlet_alpha	0.5
num_adv	0.40
4. 快速运行
在 FashionMNIST + IPM 攻击下运行 MARS-FL：
python main.py --config ./configs/FedAvg_FashionMNIST_config.yaml --epochs 100 --attack IPM --defense FLTrustLayerMultiAnchorDSACV3Stable --distribution non-iid --gpu_idx 0
在 CIFAR10 + MinSum 攻击下运行 MARS-FL：
python main.py --config ./configs/FedAvg_CIFAR10_config.yaml --epochs 100 --attack MinSum --defense FLTrustLayerMultiAnchorDSACV3Stable --distribution non-iid --gpu_idx 0 --num_adv 0.4
5. 支持的攻击与防御方法
命令行中需要使用代码支持的精确大小写。
论文主实验使用的模型投毒攻击：
• Gaussian
• IPM
• MinMax
• MinSum
对比防御方法：
• Median
• TrimmedMean
• Krum
• FLTrust
• FLARE
本文方法：
• FLTrustLayerMultiAnchorDSACV3Stable
6. 主实验复现
复现实验时，应保持数据集、模型结构、数据划分、随机种子、攻击方式、恶意客户端比例和训练超参数一致，仅改变 --defense 参数进行方法对比。
项目	建议设置
数据集	MNIST, FashionMNIST, CIFAR10
攻击	Gaussian, IPM, MinMax, MinSum
对比方法	Median, TrimmedMean, Krum,  FLTrust, FLARE
本文方法	FLTrustLayerMultiAnchorDSACV3Stable
随机种子	本实验使用 2, 4, 5
统计指标	最后 5 轮测试准确率均值
7. 输出与画图
训练日志和实验结果默认保存在 logs/ 目录下，路径格式如下：
logs/{algorithm}/{dataset}_{model}/{distribution}/

日志文件中记录每轮训练的 Test Acc、Macro F1、Agg Time 和 Round Time 等信息。

可使用以下脚本解析日志并绘制准确率曲线：
tools/plot_compare_acc.py
tools/plot_mean_std_acc.py
8. 主要文件说明
文件	作用
main.py	运行单次联邦训练实验
global_args.py	解析命令行参数、读取 YAML 配置并应用覆盖项
fl/server.py	收集客户端更新、调用聚合器并更新全局模型
fl/client.py	执行客户端本地训练并生成模型更新
aggregators/fltrust_layer_multianchor_dsac_v3satable.py	MARS-FL 当前主实现，对应论文中的完整方法
tools/plot_compare_acc.py	解析日志并绘制准确率曲线
tools/plot_mean_std_acc.py	绘制多随机种子的均值和标准差曲线
