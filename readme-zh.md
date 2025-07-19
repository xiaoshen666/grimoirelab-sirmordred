# SirMordred

构建状态 | 覆盖率状态 | PyPI 版本



SirMordred 是用于协调 GrimoireLab 平台执行的工具，通过两个主要配置文件（setup.cfg 和 projects.json）实现，其相应部分的说明如下。

## 目录

- Setup.cfg 配置文件
- Projects.json 配置文件
- 支持的数据源
- Micro Mordred
- 快速开始
- 问题排查
- 操作指南

## Setup.cfg 配置文件

配置文件（setup.cfg）用于存储 GrimoireLab 所有底层流程的配置信息，包含多个配置节，可定义以下内容：



- 通用设置：如需要激活的阶段（如数据收集、数据富集）、日志存储位置；
- SortingHat 和 ElasticSearch 实例的位置及凭据（用于存储原始数据和富集数据）；
- 后端配置节：设置 Perceval 访问软件开发工具所需的参数（如 GitHub 令牌、Gerrit 用户名）及数据拉取规则。



如果在配置节 [panels] 中启用了`panels`阶段，仪表盘可自动上传。“数据状态” 和 “概览” 仪表盘会包含汇总 setup.cfg 中声明的数据源信息的组件。注意：添加新数据源后，组件不会自动更新，需手动删除这两个仪表盘（路径：Stack Management > Saved Objects in Kibiter），并重启 mordred（确保`panels`选项已启用）。

### 配置节详情

#### [es_collection]

- url（字符串，默认：[http://172.17.0.1:9200](http://172.17.0.1:9200/)）：Elasticsearch 的 URL（必填）

#### [es_enrichment]

- autorefresh（布尔值，默认：True）：是否执行身份自动刷新
- autorefresh_interval（整数，默认：2）：身份自动刷新的时间间隔（天）
- url（字符串，默认：[http://172.17.0.1:9200](http://172.17.0.1:9200/)）：Elasticsearch 的 URL（必填）

#### [general]

- bulk_size（整数，默认：1000）：使用批量操作向 Elasticsearch 写入的数据条目数
- debug（布尔值，默认：True）：调试模式（主要用于日志输出）（必填）
- logs_dir（字符串，默认：logs）：sirmordred 的日志存储目录（必填）
- min_update_delay（整数，默认：60）：任务（收集、富集等）之间的最短延迟（秒）
- scroll_size（整数，默认：100）：从 Elasticsearch 滚动读取的数据条目数
- short_name（字符串，默认：Short name）：项目的短名称（必填）
- update（布尔值，默认：False）：是否循环执行任务（必填）
- aliases_file（字符串，默认：./aliases.json）：定义原始索引和富集索引别名的 JSON 文件
- menu_file（字符串，默认：./menu.yaml）：定义 Kibiter 中显示的菜单的 YAML 文件
- global_data_sources（列表，默认：bugzilla, bugzillarest, confluence, discourse, gerrit, jenkins, jira）：全局收集的数据源列表，在 projects.json 的`unknown`节中声明
- retention_time（整数，默认：None）：保留数据的最大时间（相对于当前时间，单位：分钟）
- update_hour（整数，默认：None）：忽略`min_update_delay`，强制任务（收集、富集等）执行的当天小时数

#### [panels]

- community（布尔值，默认：True）：是否在仪表盘中包含社区部分
- kibiter_default_index（字符串，默认：git）：Kibiter 的默认索引模式
- kibiter_time_from（字符串，默认：now-90d）：Kibiter 的默认时间范围
- kibiter_url（字符串）：Kibiter 的 URL（必填）
- kibiter_version（字符串，默认：None）：Kibiter 版本
- kafka（布尔值，默认：False）：是否在仪表盘中包含 KIP 部分
- github-comments（布尔值，默认：False）：启用 GitHub 评论菜单。注意：启用后，需在 setup.cfg 和 projects.json 中声明 github2:issue 和 github2:pull 配置节
- github-events（布尔值，默认：False）：启用 GitHub 事件菜单。注意：启用后，需在 setup.cfg 和 projects.json 中声明 github:event 配置节
- github-repos（布尔值，默认：False）：启用 GitHub 仓库统计菜单。注意：启用后，需在 setup.cfg 和 projects.json 中声明 github:repo 配置节
- gitlab-issues（布尔值，默认：False）：启用 GitLab 问题菜单。注意：启用后，需在 setup.cfg 和 projects.json 中声明 gitlab:issue 配置节
- gitlab-merges（布尔值，默认：False）：启用 GitLab 合并请求菜单。注意：启用后，需在 setup.cfg 和 projects.json 中声明 gitlab:merge 配置节
- mattermost（布尔值，默认：False）：启用 Mattermost 菜单
- code-license（布尔值，默认：False）：启用代码许可证菜单。注意：启用后，需在 setup.cfg 和 projects.json 中声明 colic 配置节
- code-complexity（布尔值，默认：False）：启用代码复杂度菜单。注意：启用后，需在 setup.cfg 和 projects.json 中声明 cocom 配置节
- strict（布尔值，默认：True）：启用严格的面板加载模式
- contact（字符串，默认：None）：支持仓库的 URL

#### [phases]

- collection（布尔值，默认：True）：激活数据收集阶段（必填）
- enrichment（布尔值，默认：True）：激活数据富集阶段（必填）
- identities（布尔值，默认：True）：执行身份相关任务（必填）
- panels（布尔值，默认：True）：加载面板、创建别名及其他相关任务（必填）

#### [projects]

- projects_file（字符串，默认：projects.json）：按项目分组的仓库列表文件路径
- projects_url（字符串，默认：None）：项目文件的 URL，需通过 projects_file 指定本地存储路径

#### [sortinghat]

- affiliate（布尔值，默认：True）：将身份关联到组织
- autogender（布尔值，默认：False）：为个人资料添加性别信息（执行 autogender 工具）
- autoprofile（列表，默认：['customer', 'git', 'github']）：获取身份信息以填充个人资料的顺序（必填）
- database（字符串，默认：sortinghat_db）：Sortinghat 数据库名称（必填）
- host（字符串，默认：127.0.0.1）：Sortinghat 数据库所在主机（必填）
- port（整数，默认：None）：GraphQL 服务器端口
- path（字符串，默认：None）：GraphQL 路径
- ssl（布尔值，默认：False）：GraphQL 服务器是否使用 SSL/TSL 连接
- matching（列表，默认：['email']）：Sortinghat 中身份匹配的算法（必填）
- password（字符串）：访问 Sortinghat 数据库的密码（必填）
- reset_on_load（布尔值，默认：False）：加载时取消所有身份的合并并移除关联
- sleep_for（整数，默认：3600）：身份任务执行的间隔时间（秒）（必填）
- strict_mapping（布尔值，默认：True）：严格检查身份匹配中的值（如格式正确的电子邮件地址、不重叠的加入时间段）
- unaffiliated_group（字符串，默认：Unknown）：未关联组织的身份所属的组织名称（必填）
- user（字符串，默认：root）：访问 Sortinghat 数据库的用户名（必填）

#### [backend-name:tag]（tag 为可选）

- collect（布尔值，默认：True）：启用 / 禁用数据收集阶段
- raw_index（字符串，默认：None）：存储原始数据的索引名称（必填）
- enriched_index（字符串，默认：None）：存储富集数据的索引名称（必填）
- studies（列表，默认：[]）：需要执行的研究列表
- anonymize（布尔值，默认：False）：启用 / 禁用用户个人信息匿名化
- backend-param-1: ...（后端参数 1）
- backend-param-2: ...（后端参数 2）
- ...



以上是后端配置节的模板。Perceval 后端参数的详细信息可参考：



- 参数详情：https://perceval.readthedocs.io/en/latest/perceval/perceval-backends.html
- 示例：https://github.com/chaoss/grimoirelab-sirmordred/blob/main/tests/test_studies.cfg



注意：部分后端配置节支持指定特定的富集选项，如下：



- [jenkins]
  node_regex：用于从 builtOn 字段提取节点名称的正则表达式，必须包含至少一个分组，第一个分组用于提取节点名称，其他分组不用于提取信息。

#### [studies-name:tag]（tag 为可选）

- study-param-1: ...（研究参数 1）
- study-param-2: ...（研究参数 2）
- ...



以上是研究配置节的模板。研究参数的完整列表可参考：
https://github.com/chaoss/grimoirelab-sirmordred/blob/main/tests/test_studies.cfg

## Projects.json 配置文件

projects.json 用于描述按项目分组的仓库，这些仓库将在仪表盘中展示。



通过该文件，用户可列出需要分析的软件开发工具实例，例如本地和远程 Git 仓库、GitHub 和 GitLab 的问题跟踪器 URL、Slack 频道名称等。此外，还可将这些实例组织为嵌套组，其结构会反映在可视化成果（文档和仪表盘）中。组可用于表示：公司内的项目、大型项目（如 Linux、Eclipse）的子项目、协作项目中的组织等。

### 结构层级

- 一级：项目名称
- 二级：数据源和元数据
- 三级：数据源 URL

### 特殊过滤器、标签和配置节

- 过滤器`--filter-no-collection=true`：用于在仪表盘中展示已不存在于上游的仓库的历史富集数据。
- 过滤器`--filter-raw`和`unknown`配置节：数据源仅在`unknown`节中收集，但可在不同节中通过`--filter-raw`进行富集。
- 标签`--labels=[example]`：数据源会被标记为`example`，可用于创建特定数据集的可视化。
- `unknown`配置节：若数据源仅存在于该节中，会作为 “主项目” 进行富集。

### 示例

json











```json
{
    "Chaoss": {
        "gerrit": [
            "gerrit.chaoss.org --filter-raw=data.projects:CHAOSS"
        ],
        "git": [
            "https://github.com/chaoss/grimoirelab-perceval",
            "https://<username>:<api-token>@github.com/chaoss/grimoirelab-sirmordred"
        ],
        "github": [
            "https://github.com/chaoss/grimoirelab-perceval --filter-no-collection=true",
            "https://github.com/chaoss/grimoirelab-sirmordred  --labels=[example]"
        ]
    },
    "GrimoireLab": {
        "gerrit": [
            "gerrit.chaoss.org --filter-raw=data.projects:GrimoireLab"
        ],
        "meta": {
            "title": "GrimoireLab",
            "type": "Dev",
            "program" : "Bitergia",
            "state": "Operating"
        }
    },
    "unknown": {
        "gerrit": [
            "gerrit.chaoss.org"
        ],
        "confluence": [
            "https://wiki.chaoss.org"
        ]
    }
}
```

### 示例说明

- 仓库`gerrit.chaoss.org`因在`unknown`节中列出，会被完整收集，但仅作为 GrimoireLab 项目的一部分进行富集（如 GrimoireLab 节中声明）。
- Chaoss 节的 github 数据源中，仓库`grimoirelab-perceval`不会被收集到原始索引，但会被富集到富集索引。
- GrimoireLab 节的元数据会作为额外字段显示在富集索引中。
- `unknown`节的 confluence 数据源会作为 “主项目” 进行富集。

## 支持的数据源

GrimoireLab 支持的数据源包括：askbot、bugzilla、bugzillarest、cocom、colic、confluence、crates、discourse、dockerhub、dockerdeps、dockersmells、functest、gerrit、git、github、github2、gitlab、gitter、google_hits、groupsio、hyperkitty、jenkins、jira、kitsune、mattermost、mbox、mediawiki、meetup、mozillaclub、nntp、pagure、phabricator、pipermail、puppetforge、redmine、remo、rocketchat、rss、slack、stackexchange、supybot、telegram、twitter

### askbot

来自 Askbot 网站的问答内容



- projects.json 示例



json











```json
{
    "Chaoss": {
        "askbot": [
            "https://askbot.org/"
        ]
    }
}
```



- setup.cfg 示例



ini











```ini
[askbot]
raw_index = askbot_raw
enriched_index = askbot_enriched
```

### bugzilla

来自 Bugzilla 的缺陷数据



- projects.json 示例



json











```json
{
    "Chaoss": {
        "bugzilla": [
            "https://bugs.eclipse.org/bugs/"
        ]
    }
}
```



- setup.cfg 示例



ini











```ini
[bugzilla]
raw_index = bugzilla_raw
enriched_index = bugzilla_enriched
backend-user = yyyy（可选）
backend-password = xxxx（可选）
no-archive = true（建议启用）
```

### bugzillarest

来自 Bugzilla 服务器（>=5.0）的缺陷数据（通过 REST API）



- projects.json 示例



json











```json
{
    "Chaoss": {
        "bugzillarest": [
            "https://bugzilla.mozilla.org"
        ]
    }
}
```



- setup.cfg 示例



ini











```ini
[bugzillarest]
raw_index = bugzillarest_raw
enriched_index = bugzillarest_enriched
backend-user = yyyy（可选）
backend-password = xxxx（可选）
no-archive = true（建议启用）
```

### cocom

代码复杂度集成，需依赖 graal 的部分工具（如 cloc），参考：https://github.com/chaoss/grimoirelab-graal#how-to-installcreate-the-executables



- projects.json 示例



json











```json
{
    "Chaoss":{
        "cocom": [
            "https://github.com/chaoss/grimoirelab-toolkit"
        ]
    }
}
```



- setup.cfg 示例



ini



```ini
[cocom]
raw_index = cocom_chaoss
enriched_index = cocom_chaoss_enrich
category = code_complexity_lizard_file
studies = [enrich_cocom_analysis]
branches = master
worktree-path = /tmp/cocom/
```

### （其他数据源配置类似，此处省略，可参考官方文档）

## Micro Mordred

Micro Mordred 是简化版的 Mordred，省略了调度器，每次执行仅支持运行单个 Mordred 任务（如原始数据收集、数据富集）。



Micro Mordred 位于仓库的`sirmordred/utils`目录下，可通过命令行执行，参数如下：



- --debug：以调试模式执行
- --raw：激活原始数据任务
- --enrich：激活数据富集任务
- --identities：激活身份合并任务
- --panels：激活面板任务
- --cfg：配置文件路径
- --backends：执行任务的配置节列表
- --logs-dir：日志存储目录

### 执行示例

bash

```bash
cd .../grimoirelab-sirmordred/sirmordred/utils/
# 为Git配置节执行原始数据和富集任务
micro.py --raw --enrich --cfg ./setup.cfg --backends git
# 执行面板任务以加载Sigils面板到Kibiter
micro.py --panels
# 为groupsio配置节执行原始数据和富集任务（调试模式，日志存储在logs目录）
micro.py --raw --enrich --debug --cfg ./setup.cfg --backends groupsio --logs-dir logs
```
