# 写在前面
在此之前我还没有尝试使用过cursor或者windsurf，但是听说vscode+cline（RooCoder）+deepseek可以作为cursor的平替，加之讯飞星火模型在25年3月10日前免费提供deepseek-v3和r1模型的免费api调用，就拿Roo Coder插件来体验一下ai当前对实际开发的提效。
# 配置
在火星Maas平台上部署Deepseek-R1，并将api-Key配置进Roo Coder里，此处选择第三方openai接口。
个人开发环境是wsl中的ubuntu。
# 实战
选择一个约在2k star以内的此前不熟悉的项目作为体验ai编程的起点。
这里选择了[kuberay](https://github.com/ray-project/kuberay)，共有1.5k star，随便找了其中的一个issue https://github.com/ray-project/kuberay/issues/3061
我们来看看ai能否帮助我们解决这个问题。
## 项目介绍
首先让ai帮我们熟悉一下这个项目。

![Image](https://github.com/user-attachments/assets/c4035054-09a2-48a1-ad91-d2943714ddb4)

直接向ai提交指令就得到了产出：
项目架构特点：
• 采用Operator模式深度集成Kubernetes
• 支持GPU资源调度（示例中的PyTorch GPU训练）
• 提供Prometheus+Grafana监控方案
• 支持多云环境（AWS/GCP/Aliyun等案例）

典型应用场景：
• 分布式机器学习训练（XGBoost、PyTorch）
• 实时推理服务（图像分类、文本摘要、Stable Diffusion）
• 批处理任务（RayJob批量推理）
• 与Kubeflow/Volcano等MLOps平台集成
企业级特性：
• 通过RBAC实现安全管控（参考apiserver/SecuringImplementation.md）
• 支持持久化存储（Volumes.md）
• 高可用集群部署方案（HACluster.md）
• 服务网格集成（ingress支持）

部署管理：
• 提供Helm Chart标准化部署
• 包含APIServer扩展（简化配置管理）
• 支持CI/CD流水线（CONTRIBUTING.md中的开发规范）

可观测性：
• 内置指标采集（config/prometheus）
• Grafana监控看板（提供多个预设仪表盘）
• 性能测试工具（benchmark目录）

社区生态：
• 与多家云厂商深度合作（Google Cloud、阿里云等）
• 丰富的集成案例（Kubeflow、Volcano、Flyte）
• 活跃的Slack支持渠道（#kuberay-questions）

KubeRay 是专为 Kubernetes 设计的 Ray 集群管理框架，主要包含以下核心能力：
RayCluster 控制器
全自动生命周期管理：秒级创建/销毁计算集群
智能弹性伸缩：基于工作负载自动调整节点数量
自愈机制：自动替换故障节点，保障服务连续性
异构资源支持：可配置CPU/GPU/Memory等资源配额
RayJob 任务引擎
任务编排：自动创建临时集群执行批处理任务
结果持久化：支持将任务输出保存至对象存储
成本优化：任务完成后自动回收计算资源
RayService 服务网格
蓝绿部署：支持无损服务版本升级
流量管理：集成Ingress实现智能路由
金丝雀发布：逐步验证新版本稳定性

## 解决issue
然后我们将issue中的问题喂给了ai。ai开始分析并给出了几个shell指令尝试。

![Image](https://github.com/user-attachments/assets/ac43bc55-cbd9-4da3-babb-fe5dcf4b1cec)

这里我的开发环境有个坑点，我的GO111MODULE有误，使用了大写的ON，这个配置是无法被正确识别的，会导致go run或者go get在内的命令无法正确运行。

![Image](https://github.com/user-attachments/assets/ef79d923-cfc1-4bae-a5bb-c3bb4ce1fa45)

但是RooCoder可以利用ai解决这一问题，帮我输出了正确的export命令。

![Image](https://github.com/user-attachments/assets/8f98fab1-1a86-4d77-a1f4-89426edfb63f)

可以看到deepseek-R1的思维链

![Image](https://github.com/user-attachments/assets/cc9262af-a340-4ec5-861f-eaa5aadbcc39)

![Image](https://github.com/user-attachments/assets/5a3cb20a-de8c-4c11-aca1-cbdb33283601)

它甚至会在shell命令行中显示timeout时帮我们配置代理

![Image](https://github.com/user-attachments/assets/47c5898c-af78-4904-bf3f-c1573ab138e3)

但是很遗憾，至少这次试用中RooCoder并没有帮助我们解决问题。（报错看起来是科大讯飞的网络问题）

![Image](https://github.com/user-attachments/assets/62524767-ad6b-4f15-b2b8-721f54a096f7)

这里我手动跑了下make，发现其实在我本地是能正常跑通的（go1.22.1），遂切换为go1.23.5，查阅之后发现golangci-lint直到1.60.1版本才支持go1.23
https://github.com/golangci/golangci-lint/issues/4837
我们将这一情报告知ai，看看ai如何为我们解决。

![Image](https://github.com/user-attachments/assets/8e5ef007-ac47-4401-9cd2-403b5caf35a3)

这里它查到的最新版本只有1.57，这里是模型本身没有联网的缘故，我们强行告诉它最新版本是1.60.5。

![Image](https://github.com/user-attachments/assets/eddf2317-b702-4ca3-9075-92bc7f277a50)

ai这里是很听话的，告诉它1.60.5，它就改成1.60.5了。
但是其实没有1.60.5这个版本，拉取这个版本会导致报错。ai拿到错误后重新开始建议使用1.57版本，这时我告诉它正确可用的最新版本1.64.5。

![Image](https://github.com/user-attachments/assets/0a250267-91f6-475a-a1f3-ffcbbb67565b)

![Image](https://github.com/user-attachments/assets/83e84616-ae83-429a-b5ed-a8801111a3be)

ai正确拉取了1.64.5版本，这时它以为自己的任务结束了，然而这个时候，其实运行make命令还是跑不通的，这里我们重新把make命令之后的报错提示喂给它。

![Image](https://github.com/user-attachments/assets/c716835d-7876-4cba-89a8-36126a333753)

它拿到报错提示之后开始编辑文件。我们可以在插件中看到它具体做出了哪些改动。

![Image](https://github.com/user-attachments/assets/5f8602e4-5b27-49e4-a59b-89857f194579)

最后这版是可以正常跑通make的。

![Image](https://github.com/user-attachments/assets/6d33d3a6-ace8-4072-9286-317c41bba7ee)

这里给出ai处理的几个lint问题：

![Image](https://github.com/user-attachments/assets/f8877287-05ef-495d-9d46-8990390a0e39)

# 结语
总体使用上很大程度可以帮助我熟悉不明白的项目或者解决一下无关紧要但是可能很烦人的配置问题，从能实际解决github上的issue看，ai和RooCoder可以帮助开源社区与开发者更加高效。