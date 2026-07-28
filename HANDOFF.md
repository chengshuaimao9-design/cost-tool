# 四轮成本计算 - 项目交接文档

## 项目概述

经销商内部工具，快速查询电动车（四轮车型）+ 电瓶成本 + 运费，得到总成本用于报价。客户报出收货地址后，自动匹配运费，成本一目了然，方便判断利润空间给客户减免运费。

## 技术栈

单页 HTML 应用（无框架、无构建工具）
- HTML5 + CSS3 + 原生 JavaScript
- 部署：Vercel (静态站点)

## 文件结构

`
四轮成本计算/
  index.html    # 完整单页应用（数据+样式+逻辑）
  HANDOFF.md    # 本文件
`

所有代码、样式、数据、逻辑都在 index.html 一个文件中。

## 数据结构（均在 window.__DATA__ 中）

### ehicles - 车型与电瓶成本

`json
{
  "四门": {
    "vehicle_cost": 5000,
    "batteries": [
      {"capacity": 20, "type": "铅酸", "cost": 700},
      {"capacity": 32, "type": "铅酸", "cost": 1000},
      ...
    ]
  },
  "三门": { "vehicle_cost": 3500, ... },
  "五门": { "vehicle_cost": 4000, ... }
}
`

- 三门/四门/五门是同一种（vehicle_cost 不同）
- 电瓶分铅酸和锂电两种
- 容量从 20AH 到 80AH 不等，每种容量有成本价

### shipping_short - 短途城市运费

数组，元素为 { province, cities: [{ city, price }] }
- 覆盖江苏/河北/山西/上海/天津/北京/浙江/安徽/山东/河南/湖北/陕西
- 每个城市有对应运费（元）

### shipping_long - 长途省份运费

字符串数组，如：
- "江西全境800元赣州加100元"
- "福建全境900元（福建南平1200元上岛另算）"
- "广东全境1100元（湛江.茂名1200）"
- "海南全境2200元"
- ...
- "2.9米之内价格  2.9米-3米加100..."
- "金豆 ，3.1米1.6之内车厢的车子价格在报价基础降100"

## 核心交互流程

1. 用户选择车型（三门/四门/五门）
2. 用户选择电瓶类型（铅酸/锂电）
3. 用户选择容量（AH）
4. 界面显示：电车成本 + 电瓶成本 + 成本小计
5. 用户输入城市名查运费
6. 下拉匹配结果显示后点击选中
7. 界面显示：运费 + 总成本（电车+电瓶+运费）

## 已修复的 Bug（请勿回退）

1. **长途省份无法点击选中**：原代码 onclick 在字符串拼接时断裂。改用事件委托处理所有点击
2. **closest() 选择器错误**：原使用 .search-results .item（后代选择器），但 closest() 需要单个元素匹配，改为先 closest(".item") 再检查父容器
3. **中文括号解析**：备注信息使用全角括号 （），原正则只匹配半角 ()
4. **pickProvince 死代码**：引用未定义变量 L，已删除
5. **价格属性名不一致**：长途 item 用 data-pr，事件委托读 data-price，添加了 || item.getAttribute("data-pr") 兜底

## 部署

### Vercel

- 项目名：cost-calc-deploy
- 生产域名：https://cost-calc-deploy.vercel.app/
- Vercel Token：cp_8QMJiyQVLBUwueEjfrWPoIMvbTsRgIEesIAkACaGlSkhI0rPyz27TjCp
- 部署方式：Vercel API POST /v13/deployments，文件 base64 编码

更新只需上传 index.html：
`python
import base64, json, urllib.request
html = open("index.html","rb").read()
b64 = base64.b64encode(html).decode()
payload = json.dumps({
  "name":"cost-calc-deploy",
  "project":"prj_TvrjTj5k5Ea9GtWb4qQZlvU0KY4U",
  "files":[{"file":"index.html","data":b64}],
  "target":"production"
}).encode()
req = urllib.request.Request(
  "https://api.vercel.com/v13/deployments", data=payload,
  headers={"Authorization":"Bearer VERCEL_TOKEN",
           "Content-Type":"application/json"})
urllib.request.urlopen(req, timeout=30)
`

### GitHub Pages（备用）

- 仓库：chengshuaimao9-design/cost-tool
- Pages 地址：https://chengshuaimao9-design.github.io/cost-tool/
- GitHub Token（已配置）：GITHUB_TOKEN

## 修改指南

### 修改车辆成本
直接在 HTML 中搜索 __DATA__.vehicles 下的对应车型，修改 ehicle_cost 值

### 修改电瓶成本
在对应车型的 atteries 数组中增删改条目（注意区分铅酸和锂电）

### 修改运费价格
- 短途：搜索 shipping_short，按省份-城市结构修改
- 长途：搜索 shipping_long，按字符串格式修改（保持 省份名数量元 格式）

## 本地运行

双击 index.html 即可在浏览器打开，无需本地服务器。
