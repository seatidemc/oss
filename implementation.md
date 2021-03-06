# 抢占式实例 Minecraft 服务器开服实践

本文介绍使用阿里云 ECS 抢占式实例开启 Minecraft 服务器的基本实现方法以及相关工具的简单实现。

## 介绍

我们通常以包月方式购买阿里云 ECS，对于轻度用户来说，这也许有些过于昂贵且没必要。因为很多时候，这买来的一个月里并没有多少玩家参与游戏，导致资源被大幅度浪费，甚至最后「人财两空」。如果既想要较为稳定方便的服务商，又不希望浪费资源，则可以考虑使用阿里云 ECS 提供的**抢占式实例**。抢占式实例具有以下基本特点：

### 较为优惠的计费模式

抢占式实例是按秒计费的。在初次购买时，价格会显示一个小时的价格。在流量方面，既可以选择按量计费，也可以按带宽计费。实例持有阶段将持续计费，实例释放后立刻停止计费。因此，这能够有效帮助我们节省成本，任何时候只要不需要开服（人太少等因素造成），都可以直接释放实例，防止损失。同时，流量方面也可以在人少的时候减少支出（选择按量计费）。

粗略来看，较低配置（但足够开 Minecraft 服务器）的抢占式实例每小时的价格是等规格按量付费实例价格的 10%（一折后价格），配置越高此折扣越低。

### 较为特殊的生命周期

抢占式实例在创建时可选择保护期：无确定保护期和一个小时保护期。对于一个小时保护期，在创建以后的一个小时里，你的实例不会被释放。而在这一个小时以后，每五分钟将根据该规格的库存以及市场价等因素来决定是否释放你的实例。实例被释放后，所有的数据将无法恢复。若选无确定保护期，则在创建以后就直接进入这种状态。

有关更多抢占式实例的描述信息，可以参考[官方文档](https://help.aliyun.com/document_detail/52088.html)。

## 开服的基本实现

要使用抢占式实例开服，首先要确保的是服务器的数据安全。根据阿里云官方描述，抢占式实例适用于无状态应用程序，因为会被释放并摧毁数据。如果你需要高配机来一次性地试着跑一些东西，用抢占式实例再好不过（就相当于是掏了不到一块钱或者一块钱左右把高配机租来用一下）。而对于 Minecraft 服务器这种有状态应用程序，似乎就不太适合。但是我们选择抢占式实例最主要的原因之一是，它的释放率并不高，一般在 1% ~ 3% 左右。如果地区选择得当，出价方式正确，基本上不会被释放。而且，我们将使用脚本来实现定时备份，即使被释放，数据也是安全的。

我们将用到以下阿里云产品：

- 阿里云 ECS 的抢占式实例
- 阿里云 OSS

其中 ECS 用来开服，OSS 用来存储我们的数据。这样计算下来，成本远低于包月付费的 ECS。ECS 的费用，以 `ecs.g5.xlarge` 规格（4 核 16G 通用型）为例，据不完全估计，每个月的花费在 ￥234 左右。而同规格在包月条件下，花费为 ￥530。OSS 的费用可参考[这里](https://cn.aliyun.com/price/detail/oss)，并不会太高，且内、外网流入和内网流出流量均为免费。*数据时间：`2021/06/19`*

下面将粗略描述整个过程。

### 创建 ECS 和 OSS

此过程将不多赘述。创建 ECS 时，选择抢占式实例，根据需要选择相应的区域、配置即可。

![](https://i.loli.net/2021/06/19/vlStxYXFwysAkVg.png)

在 ECS 表格中值得参考的数据是释放率和历史折扣率。

创建 OSS 时，**必须确保 OSS 的地域与 ECS 完全相同，否则传输速率慢且需要额外付费。** 例如 ECS 选择的是华北 2 上海，则 OSS 也必须选择华北 2 上海。

### ECS 上的准备工作

此步骤见 [README](./README.md)。

## 免责声明

抢占式实例被自动释放后，所有数据均无法找回。因此，请尽可能保证账户余额充足、出价策略正确且选择到了市场价较为稳定的区域和规格。本项目配置以后，如果不能正常运作，请及时修复并对数据进行备份。任何问题都可以在 Issue 中反馈。本项目不对任何损失负责。