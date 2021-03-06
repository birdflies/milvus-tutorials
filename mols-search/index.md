summary: Milvus 化学式检索
id: how-to-do-reverse-molecular-search-with-milvus
categories: milvus-cn
tags: demo
status: Published
authors: Brother Long
Feedback Link: https://milvus.io

# Milvus 化学式检索

## 概述

Duration: 1

本文展示如何利用 Milvus 向量搜索引擎搭建一个化学式检索系统。

## 环境要求
Duration: 1

| 组件     | 推荐配置                                                     |
| -------- | ------------------------------------------------------------ |
| CPU      | Intel(R) Core(TM) i7-7700K CPU @ 4.20GHz                     |
| Memory   | 32GB                                                         |
| OS       | Ubuntu 18.04                                                 |
| Software | [Milvus 0.10.0](https://milvus.io/cn/docs/v0.10.0/guides/get_started/install_milvus/cpu_milvus_docker.md) <br />mols-search-webserver 0.7.0 <br />mols-search-webclient 0.3.0 |

以上配置已经通过测试，并且 Windows 系统也可以运行本次实验，以下步骤 Windows 系统通用。

## 数据准备

Duration: 3

本次实验数据来源：[ftp://ftp.ncbi.nlm.nih.gov/pubchem/Compound/CURRENT-Full/SDF](ftp://ftp.ncbi.nlm.nih.gov/pubchem/Compound/CURRENT-Full/SDF)，该数据集是压缩的 SDF 文件，需要使用工具将其转换为 SMILES 文件，我们准备了转换后的一万条 SMILES 化学式文件 [test_1w.smi](https://raw.githubusercontent.com/milvus-io/bootcamp/0.10.0/solutions/mols_search/smiles-data/test_1w.smi)，下载该文件到本地：

```bash
$ wget https://raw.githubusercontent.com/milvus-io/bootcamp/0.10.0/solutions/mols_search/smiles-data/test_1w.smi
```



## 部署流程
Duration: 10

#### 1. 启动 Milvus v0.10.0

本次实验使用 Milvus 0.10.0CPU 版，安装启动方法参考https://milvus.io/cn/docs/v0.10.0/guides/get_started/install_milvus/cpu_milvus_docker.md 。

#### 2. 启动 mols-search-webserver docker

```bash
$ docker run -d -v <DATAPATH>:/tmp/data -p 35001:5000 -e "MILVUS_HOST=192.168.1.25" -e "MILVUS_PORT=19530" milvusbootcamp/mols-search-webserver:0.7.0
```

上述启动命令相关参数说明：

| 参数                          | 说明                                                         |
| ----------------------------- | ------------------------------------------------------------ |
| -v DATAPATH:/tmp/data         | -v 表示宿主机和 image 之间的目录映射<br />请将 DATAPATH 修改为你本机存放 test_1w.smi 数据的目录。 |
| -p 35001:5000                 | -p 表示宿主机和 image 之间的端口映射                         |
| -e "MILVUS_HOST=192.168.1.25" | -e 表示宿主机和 image 之间的系统参数映射<br />请修改`192.168.1.25`为启动 Milvus docker 的服务器 IP 地址 |
| -e "MILVUS_PORT=19530"        | 请修改`19530`为启动 Milvus docker 的服务器端口号             |

#### 3. 启动 mols-search-webclient docker

```bash
$ docker run -d -p 8001:80 -e API_URL=http://192.168.1.25:35001 milvusbootcamp/mols-search-webclient:0.3.0
```

> 参数 -e API_URL=http://192.168.1.25:35001 与本节第二部分相对应，请修改`192.168.1.25`为启动 Milvus docker 的服务器 IP 地址。

#### 4. 打开浏览器

```bash
# 请根据以上步骤修改 192.168.1.25 地址和 8001 端口
http://192.168.1.25:8001
```




## 界面展示
Duration: 5

- 初始界面

![](./pic/init_status.PNG)

- 加载化学式数据
  1. 在 path/to/your/data 中输入 smi 文件位置：/tmp/data/test_1w.smi
  2. 点击 `+` 加载按钮
  3. 可以观察到化学式数量的变化：10000 Molecular Formula in this set

![](./pic/load_data.PNG)

- 化学式检索
  1. 输入待检索的**化学式**并按**回车**，如：Cc1ccc(cc1)S(=O)(=O)N
  2. 选择 TopK 值，将在右侧返回相似度最高的前 TopK 个化学式

![](./pic/search_data.PNG)

- 清除化学式数据

  点击`CLEAR ALL`按钮，将清除所有化学式数据

![](./pic/delete_data.PNG)
