
VMware Cloud Foundation（VCF）是一个由众多产品（vSphere、vSAN 以及 NSX 等）所构成的 SDDC 解决方案，这些环境中的不同组件的生命周期统一由 SDDC Manager 来进行管理，比如下载修补包、环境预检查、调度组件更新、监控运行报告等。比起传统解决方案来说，VCF 环境的生命周期管理要远远复杂的多，因为涉及到众多组件的产品互操作性以及依赖特定的升级顺序，如果没有 SDDC Manager 来统一进行编排，这对于管理来说会变得非常麻烦并且特别容易出错。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241016094503051-302102181.png)](https://github.com)


VCF 环境中具有管理工作负载域和 VI 工作负载域两种类型的工作负载域，不同工作负载域中具有一个或多个 vSphere/vSAN 集群，这些工作负载域中可能混合了不同物料清单（BOM）版本的组件，也可能使用了不同生命周期管理的方式，比如基于 Image（映像）或者 Baseline（基线）。如果环境不能连接互联网，可能需要配置代理服务器或者部署 Offline Bundle Transfer Utility (OBTU) 工具来下载或导入捆绑包；如果要进行异步补丁修补，早期可能需要使用 Async Patch Tool 工具来执行，但现在可以直接通过 SDDC Manager(5\.2\) 来完成这项工作。


本文以下内容参考 VMware 官方产品文档，有关更多细节和注意事项请访问[《VMware Cloud Foundation Lifecycle Management》](https://github.com)。


 



## 一、注意事项



VMware Cloud Foundation 环境的生命周期管理具有许多要求和[注意事项](https://github.com)，为了保证工作的顺利进行，请确保满足以下条件之后再进行后续操作。


* 验证 ESXi 主机 TPM 模块为禁用状态。
* 验证 ESXi 主机硬件是否与目标版本兼容。
* 验证 VCF 组件没有过期或即将过期的密码。
* 验证 VCF 组件没有过期或即将过期的证书。
* 验证 VCF 组件具有最新基于配置文件的备份。
* 验证 vSAN HCL 数据库以确保其为最新状态。
* 获取更新目标版本的许可证（如从 4\.5\.x 升级时）。
* 查看更新目标版本的发行说明了解升级相关的已知问题。
* 解决更新目标版本的环境预检查中所有的失败检查结果。
* 分配 vCenter Server 一个临时 IP 地址（如从 4\.5\.x 升级时）。
* 在 vCenter Server 中，确保主机或 vSphere 集群上没有活动警报。
* 在 SDDC Manager 中，确保系统没有正在运行任何错误或失败的任务。



 



## 二、更新流程



如果执行 VMware Cloud Foundation 更新工作流，需要遵循特定的[更新流程](https://github.com)。比如，要升级到 VMware Cloud Foundation 5\.2\.x，则管理域必须为 VMware Cloud Foundation 4\.5 或更高版本，如果你的环境版本低于 4\.5，则必须将管理域升级到 4\.5 或更高版本后再升级到 5\.2\.x。在 SDDC Manager 升级到版本 5\.2\.x 之前，必须先升级管理工作负载域，然后再升级 VI 工作负载域；SDDC Manager 版本为 5\.2 或更高版本后，只要工作负载域中的所有组件都兼容，就可以在升级管理域之前或之后升级 VI 工作负载域。如果是升级管理域中的组件，需要先下载相关组件的捆绑包并执行环境的预检查，然后按以下顺序执行相关组件的更新：



* SDDC Manager
* VMware Aria Suite（若有）
	+ VMware Aria Suite Lifecycle
	+ VMware Aria Suite Products
* NSX
	+ NSX Global Manager（若有）
	+ NSX Edge Cluster（若有）
	+ NSX Manager
* vSphere
	+ vCenter Server
	+ vSAN Witness（若有）
	+ ESXi


注意，SDDC Manager 可以部署 VMware Aria Suite Lifecycle 组件，但是 VMware Aria Suite 解决方案相关产品的生命周期得通过 Aria Suite Lifecycle 来进行管理，需要先在 Aria Suite Lifecycle 中更新自己然后再更新其他 Aria 产品，比如 Aria Operations 或者 Aria Automation 等。如果成功完成上面所有组件更新后，你还可以执行一些可选操作，比如更新集群中 VDS 分布式交换机以及 vSAN 磁盘的版本，执行 VCF 组件的最新配置备份等。如果 VCF 环境中还有其他解决方案，请访问 [KB 89745](https://github.com) 了解更多 VMware 产品的更新顺序。



 



## 三、配置联机库



导航到 SDDC Manager\-\>管理\-\>联机库，通过配置账号密码连接到 VMware 官方的在线仓库以获取安装和升级包。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241013155101388-1503074412.png)](https://github.com)


输入 Broadcom 支持门户的账号和密码，点击“身份验证”。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241013155251603-1435331805.png)](https://github.com)


已成功连接到 VMware 联机库。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241013155325368-1588174003.png)](https://github.com)


 



## 四、下载更新包



导航到 SDDC Manager\-\>生命周期管理\-\>发行版本，查看当前 VCF 5\.1 物料清单（BOM）版本。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241013155408130-671414338.png)](https://github.com)


计划将当前环境更新到 VCF 5\.2 物料清单（BOM）版本。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241013155438008-1461116627.png)](https://github.com)


导航到 SDDC Manager\-\>生命周期管理\-\>包管理，如果环境连接了互联网并配置了联机库，你将在这里看到所有可用的包，包含相关组件的安装包和修补/升级包。查看 [KB 96099](https://github.com) 了解更多有关 VMware Cloud Foundation 软件包的版本发布信息。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241013155517896-2106001735.png)](https://github.com)


点击某一个包查看详细信息（注意这里包的大小单位显示有误）。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241013160418638-93645978.png)](https://github.com)


计划更新到 VCF 5\.2 物理清单（BOM）版本，所以点击“立即下载”这些组件的修补/升级包，如下图所示。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241013160742609-864235141.png)](https://github.com)


点击筛选查看正在下载的包，将按顺序下载“调度下载”中的软件包。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241013160959066-542574898.png)](https://github.com)


点击“下载历史记录”查看所有已下载的软件包。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241013232033595-1077274635.png)](https://github.com)


**注意：**如果在列表中找不到 ESXi 的更新包，请在后面更新完 SDDC Manager 组件后再查看下载。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241015182939228-542377543.png)](https://github.com)


 



## 五、创建集群映像



由于 vSphere 集群的生命周期管理方式基于 Image，所以需要单独创建集群映像以用于 ESXi 主机的更新。导航到 SDDC Manager\-\>生命周期管理\-\>映像管理，需要在这里创建新的集群映像。


**注意：**请在正式执行 ESXi 组件的更新之前再执行这一步，详见“**七、执行更新过程**”步骤中的“**4）更新 ESXi 主机**”小节。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241013214425657-1209784525.png)](https://github.com)


点击“导入映像”，点击转到管理域 vCenter Server（vSphere Client）创建 vSphere Lifecycle Manager 映像，然后在创建映像期间定义 ESXi 版本，并选择添加供应商加载项、组件和固件等，最后将映像提取到 SDDC Manager 中。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241013214607259-1988458946.png)](https://github.com)


进入 vSphere Lifecycle Manager 管理视图，如果之前在 SDDC Manager 中已经下载了 ESXi 更新包，则应该会自动将 ESXi 映像导入到 vLCM 映像库中；如果没有，请在“操作”中导入本地映像。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241013214857298-2070012302.png)](https://github.com)


在数据中心级别右击新建集群，设置新集群的名称，选择使用映像管理集群，点击下一页。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241013215114142-1780662313.png)](https://github.com)


选择映像的 ESXi 版本，若有供应商加载项可选择添加，点击下一页。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241013215141106-859321959.png)](https://github.com)


完成集群创建。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241013215156354-203131700.png)](https://github.com)


导航到新创建的集群\-\>更新\-\>主机\-\>映像，你可以根据情况编辑集群的映像设备，比如添加供应商加载项、组件和固件等。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241013215225599-1810651059.png)](https://github.com)


完成集群映像设置后回到 SDDC Manager，在“导入映像”中选择“选项 1”，选择工作负载域以及刚刚创建的新集群，设置集群映像的名称，然后点击“提取集群映像”。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241013215449358-1298859818.png)](https://github.com)


映像提取成功后，在“可用的映像”中可以看到这个新的集群映像。后续可将临时创建的集群从 vCenter Server 中删除。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241013215726110-1337938213.png)](https://github.com)


 



## 六、更新前预检查



正式执行更新之前，需要先对当前环境运行预检查，确保已准备好进行更新。导航到 SDDC Manager\-\>清单\-\>工作负载域，选择要执行更新的工作负载域，比如管理工作负载域（vcf\-mgmt01），转到“更新”并点击“运行预检查”。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241015101730435-538556045.png)](https://github.com)


预检查目标版本选项选择“常规升级就绪情况”或者目标 VCF 版本，默认检查整个工作负载域，也可根据情况选择指定组件，然后点击“运行预检查”。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241013163650860-974031370.png)](https://github.com)


预检查结果。如果有警告，可以暂时忽略而不影响更新；如果有错误，请一定要全部进行解决。由于当前测试环境是通过嵌套部署的，所以下面提示 vSAN 主机磁盘控制器错误，这个错误直接“静默”即可，不影响后续更新。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241013152401034-1032234503.png)](https://github.com)


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241013152438348-1190173888.png)](https://github.com)


点击“静默预检查”。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241013154810512-301419763.png)](https://github.com)


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241013162723473-1556894277.png)](https://github.com)


 



## 七、执行更新过程



正式执行更新之前，请为所有组件创建虚拟机快照，NSX 组件无法创建虚拟机快照，基于文件的备份是 NSX 组件唯一受支持的方式。完成以上所有准备工作之后，下面正式执行更新过程。导航到 SDDC Manager\-\>清单\-\>工作负载域，选择要执行更新的工作负载域，转到“更新”并点击可用更新中的“计划升级”。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241015124444223-34231714.png)](https://github.com)


选择计划要升级的 VCF 版本，点击确认。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241015124526487-861411960.png)](https://github.com)


点击“查看包”。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241015124646378-333717913.png)](https://github.com)


以下是 VCF 组件的升级顺序。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241015124707214-1104142084.png)](https://github.com)


### **1）更新 SDDC Manager**


点击可用更新中的“立即更新”，开始 SDDC Manager 组件的更新工作流。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241015124756106-828260484.png)](https://github.com)


点击立即更新后，将跳转到一个更新页面，如下图所示。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241015124904921-899811165.png)](https://github.com)


点击“查看更新活动”了解具体更新内容。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241015124925540-376348211.png)](https://github.com)


展开更新选项卡查看更新任务状态。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241015125211187-1622806714.png)](https://github.com)


完成更新。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241015132920412-539386278.png)](https://github.com)


重新登陆 SDDC Manager UI，查看当前版本。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241015133420153-1137501323.png)](https://github.com)


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241015133313432-1072928027.png)](https://github.com):[豆荚加速器](https://yirou.org)


### **2）更新 NSX Manager**


点击可用更新中的“立即更新”，开始 NSX Manager 组件的更新工作流。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241015134047197-606326380.png)](https://github.com)


当前环境未部署 NSX Edge 集群，所以点击下一步。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241015134154919-1527478094.png)](https://github.com)


默认一次性更新工作负载域中的所有主机集群，可以勾选“允许选择主机集群”自定义勾选更新的主机集群，然后点击下一步。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241015134232012-710110697.png)](https://github.com)


默认是并行升级，可勾选主机集群是否进行顺序更新，点击下一步。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241015134307782-970801509.png)](https://github.com)


检查更新选项，点击完成并开始更新过程。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241015134323527-1136910541.png)](https://github.com)


点击任务视图可跟踪任务状态。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241015134503101-1349632377.png)](https://github.com)


登陆 NSX Manager UI（VIP）查看升级状态。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241015151732576-984237789.png)](https://github.com)


成功更新。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241015164233907-533140901.png)](https://github.com)


查看当前版本。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241015164426318-1682951681.png)](https://github.com)


### **3）更新 vCenter Server**


点击可用更新中的“立即更新”，开始 vCenter Server 组件的更新工作流。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241015164621909-918188098.png)](https://github.com)


确认已完成 vCenter Server 配置备份。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241015164651432-656293256.png)](https://github.com)


点击任务视图可跟踪任务状态。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241015164740523-192923719.png)](https://github.com)


也可以在“正在进行的更新”中查看更新状态。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241015165152224-1413780461.png)](https://github.com)


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241015165210135-888632548.png)](https://github.com)


登陆 VAMI 管理后台跟踪更新状态。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241015170458216-26539649.png)](https://github.com)


成功更新。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241015180059683-1901334436.png)](https://github.com)


查看当前版本。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241015180213545-872946394.png)](https://github.com)


### **4）更新 ESXi 主机**


点击可用更新中的“配置更新”，开始 ESXi 主机组件的更新工作流。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241015183059204-1295688943.png)](https://github.com)


查看使用集群映像更新 ESXi 主机的步骤，点击下一步。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241015183213991-913630342.png)](https://github.com)


选择要执行更新工作流的集群，点击下一步。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241015183230212-230477868.png)](https://github.com)


分配集群映像给指定集群，点击下一步。**注意：**请参阅“**五、创建集群映像**”了解集群映像创建过程。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241015183313491-277931357.png)](https://github.com)


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241015183342838-82059336.png)](https://github.com)


配置自定义升级选项，点击下一步。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241015183443732-2133585076.png)](https://github.com)


检查所有更新配置，点击完成，开始应用集群映像和兼容性检查。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241015183526314-1976223009.png)](https://github.com)


点击查看兼容性检查结果，由于是嵌套虚拟化环境，所以会有很多错误和警告，如果是这些问题则可以直接忽略。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241015191239701-1082059044.png)](https://github.com)


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241015191307983-775705646.png)](https://github.com)


如果遇到 NVMe 设备兼容性问题，请登录 vCenter Server 在 Skyline 中将其“静默”即可。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241015190545845-1763254178.png)](https://github.com)


确定一切就绪，点击预检查中的“调度更新”。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241015191509768-1533265819.png)](https://github.com)


检查更新设置和更新选项，点击下一步。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241015191546640-2078979589.png)](https://github.com)


选择“立即升级”并勾选确认兼容性检查结果，点击完成。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241015191620729-1719710495.png)](https://github.com)


查看正在更新的任务状态。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241015191801301-1327264939.png)](https://github.com)


点击任务视图可跟踪任务状态。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241015191900896-1005259122.png)](https://github.com)


登录 vCenter Server 可查看任务执行情况。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241015192323385-1161342801.png)](https://github.com)


成功更新。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241015195638419-786367009.png)](https://github.com)


查看当前版本。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241015195738177-249145046.png)](https://github.com)


现在，所有 VCF 组件都已完成更新，点击管理工作负载域查看摘要信息，确认版本已经升级到了 VCF 5\.2。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241015195943413-2144771944.png)](https://github.com)


查看 VCF 管理域更新历史记录。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241015200007285-1137639530.png)](https://github.com)


查看 SDDC Manager 中所有发行版本。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241015200051403-1040151452.png)](https://github.com)


 



## 八、后续可选操作



如果从 VCF 5\.0 之前的版本升级过来，则需要更新许可证密钥以支持 vSAN 8\.x 和 vSphere 8\.x。首先，将新的组件许可证密钥添加到 SDDC Manager，然后，您可以按工作负载域将许可证密钥应用于组件。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241015200321918-397523724.png)](https://github.com)


在 vCenter Server 中，将 vSphere Distributed Switch（VDS）交换机升级到最新版本以利用仅在更高版本中可用的功能。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241015200426786-1640824832.png)](https://github.com)


在 vCenter Server 中，升级 vSAN 集群的 vSAN 磁盘格式来获得最佳状态以及最新磁盘格式提供的 vSAN 完整功能集。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241015200514785-1764055073.png)](https://github.com)


完成所有组件更新并确保运行一切正常后，可删除组件虚拟机的快照，然后创建完整的备份以获取最新配置状态。


[![](https://img2024.cnblogs.com/blog/2313726/202410/2313726-20241015200608405-399197366.png)](https://github.com)


