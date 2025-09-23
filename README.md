                                                          自动转换 Clash 规则为 MRS 格式
这是一个使用 GitHub Actions 自动将 Clash/Clash.Meta 规则文件 (.yaml, .txt, .list) 转换为 Mihomo / Clash.Meta 专属的 .mrs 高效格式的项目。

✨ 功能定时自动更新：每周定时从上游地址拉取最新的规则文件，确保规则时效性。智能格式转换：自动处理 .yaml, .txt, .list 格式的源文件。

智能拆分混合型规则（如一个文件内同时包含 DOMAIN 和 IP-CIDR），分别转换为独立的 .mrs 文件。将不支持 .mrs 转换的规则（如 PROCESS-NAME）自动归档为 classical 类型的 .list 文件。

稳定链接 & 版本归档：latest 目录：根目录下的 latest 文件夹内始终存放着最新一次成功生成的所有规则文件。

你可以直接引用此目录下的链接，地址永远不会改变。versioned_rules 目录：每次运行的产物都会以日期版本归档在此目录下，方便追溯历史版本。

两种模式：自定义源：通过在 rule_sources.txt 文件中提供链接，转换您指定的规则。

默认源：如果 rule_sources.txt 为空或不存在，项目将自动拉取并解析 GeneralClashRule.ini 文件。

自动提交：转换后的文件会自动提交到本仓库，无需人工干预。
🚀 如何使用Fork 本仓库 到你自己的 GitHub 账号。

进入你 Fork 后的仓库页面，点击 Actions 选项卡，如果有提示，请点击 "I understand my workflows, go ahead and enable them" 来启用 GitHub Actions。

配置转换源 (二选一):方法一 (推荐): 编辑仓库中的 rule_sources.txt 文件，将你需要转换的规则链接粘贴进去，每行一个。

方法二: 删除 rule_sources.txt 文件，Action 将自动使用默认的 .ini 规则源。手动触发或等待执行:你可以进入 Actions -> Auto Convert Clash Rules to MRS -> 点击 Run workflow 来立即触发一次任务。或者等待每周的定时任务自动执行。

获取结果:推荐：直接使用根目录下 latest 文件夹中的规则文件。这里的链接是固定的。根目录下的 mrs_rules.md 文件会始终指向 latest 目录的规则链接，方便你直接复制使用。如果需要查找历史版本，可以进入 versioned_rules 目录。

📂 文件结构.
├── .github/workflows/convert_rules.yml   # GitHub Action 核心配置文件
├── latest/                               # 存放最新版本规则的目录 (链接稳定)
│   ├── BilibiliHMT_domain.mrs
│   ├── BilibiliHMT_ipcidr.mrs
│   ├── Inside_domain.mrs
│   ├── Inside_ipcidr.mrs
│   ├── Inside_other.list
│   └── ...
├── versioned_rules/                      # 存放所有历史版本规则的父目录
│   └── 2025-09-23/                       # 以日期命名的单次运行结果
│       ├── rules/                        # 包含本次运行的源文件和 .mrs 文件
│       └── provider_config.yaml          # 本次运行的 Clash rule-provider 配置
├── mrs_rules.md                          # 指向 latest 目录规则的汇总列表 (推荐使用)
├── rule_sources.txt                      # (可选) 自定义规则源列表
└── README.md                             # 本说明文件
📝 Clash.Meta 配置文件示例方案一：直接使用 RULE-SET (最简单)你可以直接复制根目录下 mrs_rules.md 的内容来使用最新的规则集。# 在你的 config.yaml 的 rules 部分:
rules:
  # 示例 (具体内容请复制 mrs_rules.md):
  - RULE-SET,🎯 国内流量 (Domain),DIRECT
  - RULE-SET,🎯 国内流量 (IP),DIRECT
  - RULE-SET,🎯 国内流量 (Unsupported),DIRECT
  - RULE-SET,🤖 Google (Domain),PROXY
  # ... 更多规则
方案二：使用 rule-providers (更灵活)如果你想对规则进行更精细的控制（例如独立的更新间隔），可以参考任意版本文件夹下的 provider_config.yaml 文件。注意：请将文件中的 url 地址手动指向 latest 目录以获得稳定链接。# 在你的 config.yaml 中:

rule-providers:
  # 示例: 将版本化路径手动修改为 latest 路径
  # 原始名称: 🎯 国内流量 (Domain)
  Inside_domain:
    type: http
    behavior: domain
    # 将 versioned_rules/2025-09-23/rules/ 改为 latest/
    url: "[https://raw.githubusercontent.com/你的用户名/你的仓库/main/latest/Inside_domain.mrs](https://raw.githubusercontent.com/你的用户名/你的仓库/main/latest/Inside_domain.mrs)"
    path: "./rules/Inside_domain.mrs"
    interval: 86400
  
  # 原始名称: 🎯 国内流量 (IP)
  Inside_ipcidr:
    type: http
    behavior: ipcidr
    url: "[https://raw.githubusercontent.com/你的用户名/你的仓库/main/latest/Inside_ipcidr.mrs](https://raw.githubusercontent.com/你的用户名/你的仓库/main/latest/Inside_ipcidr.mrs)"
    path: "./rules/Inside_ipcidr.mrs"
    interval: 86400

  # 原始名称: 🎯 国内流量 (Unsupported)
  Inside_other:
    type: http
    behavior: classical
    url: "[https://raw.githubusercontent.com/你的用户名/你的仓库/main/latest/Inside_other.list](https://raw.githubusercontent.com/你的用户名/你的仓库/main/latest/Inside_other.list)"
    path: "./rules/Inside_other.list"
    interval: 86400

rules:
  - RULE-SET,Inside_domain,DIRECT
  - RULE-SET,Inside_ipcidr,DIRECT
  - RULE-SET,Inside_other,DIRECT
  # ... 更多规则
