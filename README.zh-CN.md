# Type-IV Vessel Builder (Liner & Boss) V1.0

**简体中文** | [English](README.md)

> **状态：代码与安装包暂时撤回。** 部分功能正在更新，后续会完全开放。想提前体验或交流，请联系 **Z25150070@s.upc.edu.cn**。

> **开发说明**：本软件在开发过程中借助了 vibe coding（AI 辅助编程）。
>
> **问题反馈**：使用中遇到任何问题，欢迎在 [GitHub Issues](https://github.com/IY724/Type-IV-Vessel-Builder/issues) 中反馈，或发送邮件至 **Z25150070@s.upc.edu.cn**。

## 1 插件介绍

Type-IV Vessel Builder (Liner & Boss) V1.0 是运行在 Abaqus/CAE 2025 中的Ⅳ型储氢瓶参数化几何建模插件，用于生成瓶体内衬（Liner）与金属极孔接头（Boss）两个核心部件，消除Ⅳ型储氢瓶几何建模中的繁琐重复操作。使用者在插件图形界面中选择建模空间与封头类型、填入几何尺寸参数，软件即可自动完成参数合法性校验、解析与数值封头曲线求算以及旋转体构建，在当前 CAE 会话中生成对应模型与部件，全程无需人工交互式绘制草图。

软件支持二维轴对称（2D Axisymmetric）与三维实体（3D Solid）两种建模空间：二维轴对称直接生成子午面壳体截面，适合快速评估与初选尺寸；三维实体支持任意角度回转扇区或完整三维实体构建，适合精细化局部评估或全三维仿真分析。两种建模空间在界面中采用并列标签页形式呈现，几何参数实时双向同步。

封头曲面提供半球（Hemispherical）、半椭球（Semi-ellipsoidal）和等张力（Isotensoid）三种典型轮廓：半椭球由长短轴比 k 驱动，等张力由赤道开孔角 α0 与数值测地线积分（RK4）驱动，半球由外半径解析确定。分界半径处的封头深度 l2 由软件依据封头类型精确求算并在界面中只读回显。勾选布尔合并后，软件在装配中将内衬与接头求并为单一组合部件 Vessel_Base，并分别创建 SET_LINER 与 SET_BOSS 几何集合以保留各自分区边界。

功能边界：本插件聚焦参数化几何前处理，不创建材料与截面、不划分网格、不施加载荷与边界条件、不提交求解作业；材料指派与网格划分在几何构建完成后于 Abaqus/CAE 模块中接续进行。

完整的使用说明（几何参数表、校验体系与常见问题）见仓库内 [《Type-IV Vessel Builder V1.0-软件设计说明书》](Type-IV%20Vessel%20Builder%20V1.0-软件设计说明书.pdf)（PDF）与 [Word 版](Type-IV%20Vessel%20Builder%20V1.0-软件设计说明书.docx)。

## 2 插件功能

从几何参数直接生成 IV 型瓶的内衬（Liner）与金属接头（Boss），支持 2D 轴对称与 3D 实体两种建模空间、三种封头轮廓。

**操作界面**：左侧参数分组，右侧参考图随封头类型联动（布局示意）。

<table>
<tr><td align="center" width="100%"><img src="docs/images/window-semiellipsoidal.png" width="62%"/><br/>半椭球封头参数窗口</td></tr>
</table>


**生成的模型**：Abaqus/CAE 视口。

<table>
<tr><td align="center" width="25.0%"><img src="docs/images/model-axi-isotensoid.png" width="95%"/><br/>2D 轴对称，等张力</td><td align="center" width="25.0%"><img src="docs/images/model-axi-hemispherical.png" width="95%"/><br/>2D 轴对称，半球</td><td align="center" width="25.0%"><img src="docs/images/model-solid-isotensoid-10deg.png" width="95%"/><br/>3D 实体 10° 扇区，等张力</td><td align="center" width="25.0%"><img src="docs/images/model-solid-semiellipsoidal-10deg.png" width="95%"/><br/>3D 实体 10° 扇区，半椭球</td></tr>
</table>


**参数命名与核验**

<table>
<tr><td align="center" width="50.0%"><img src="type4_vessel_builder/type4_schematic.png" width="95%"/><br/>几何参数示意</td><td align="center" width="50.0%"><img src="type4_vessel_builder/type4_verified_schematic_isotensoid.png" width="95%"/><br/>等张力封头核验图</td></tr>
</table>


**几何与校验**

| 项目 | 说明 |
|---|---|
| 封头轮廓 | 半球：外半径解析，无形状参数；半椭球：长短轴比 k 驱动；等张力：赤道开孔角 α0 加四阶龙格-库塔数值测地线积分 |
| 封头深度 | 分界半径处的 l2 自动精确求算并只读回显，校验「接头顶部高度 < l2 < 局部包络高度 − 1 mm」 |
| 部件生成 | Liner 为单连通部件（直筒段、肩部折角、封头段与内壁包络）；Boss 为对称两件（阀座通孔、颈部壁厚、界面密封环、外侧封头曲线） |
| 布尔合并 | 勾选后在装配中求并为 Vessel_Base，保留交界面特征，自动重建 SET_LINER 与 SET_BOSS |
| 参数校验 | 半径单调次序链 `0 < r0 < r_n < r_f < r_c < r_sp < R < R_out` 与多级流水线校验，未通过即拦截并给调整建议 |
| 输出与自检 | 创建后自动做几何拓扑自检；消息区打印以 `TYPE4_BUILD_OK` 开头的 JSON 回显；重复运行自动递增模型名后缀 |
| 单位与范围 | 长度 mm、角度 deg，不做隐藏换算；只做参数化几何，不创建材料、不划网格、不提交计算 |

## 3 运行环境与安装

**最省事**：到 [Releases](https://github.com/IY724/Type-IV-Vessel-Builder/releases/latest) 下载 `Type-IV-Vessel-Builder-V1.0.zip`，解压后按下面第 2 步放置即可，不必逐个文件下载。
运行环境：Abaqus/CAE 2025；插件完全依赖 Abaqus 内置的标准 Python 运行环境，无需单独配置第三方解释器或额外扩展包，也不改动 Abaqus 原生配置文件。

安装步骤：

1. 关闭全部 Abaqus/CAE 窗口；
2. 将包含完整源程序的 `type4_vessel_builder` 文件夹整体拷贝至 Abaqus 插件目录中，保持单层文件夹组织结构完整（三份 Python 程序文件与封头静态参考图必须统一置于该文件夹第一层目录内）；
3. 重启 Abaqus/CAE，在 Plug-ins 菜单中点击 Type-IV Vessel Builder 项即可弹出交互窗口（在 Part、Property、Assembly、Mesh 等前处理模块下均可调用）。

排查提示：若菜单项未出现，常见原因为插件搜索路径偏差、目录层级错乱或同名备份引发模块冲突；加载异常日志记录于插件目录下的 `type4_load.log` 与 `type4_gui_load.log`。

## 4 界面与建模方式

对话框采用左右布局：左侧为 2D Axisymmetric 与 3D Solid 两个并列标签页（含 Head type 选择与各几何参数分组框），右侧为随封头类型联动的尺寸定义参考图；底端常驻 Apply、OK、Cancel 三个动作按钮。

- **2D Axisymmetric**：生成子午面壳体截面（变形体、轴对称），Liner 为 1 个连续面，Boss 为两端各 1 个对称截面；
- **3D Solid**：激活 Sector angle 输入框（0°–360°），调用回转算子生成扇区或整周实体，360° 生成完整瓶体；
- **参数双向同步**：在任一标签页修改参数，切换后自动同步；切换标签页时建模空间随之匹配；
- **布尔合并**：勾选 Boolean merge into Vessel_Base 后，在装配中实例化并求并，保留内衬与接头交界面特征，合并部件自动重建 SET_LINER 与 SET_BOSS 集合。

单位约定：长度以 mm 输入，角度以 deg 输入（Abaqus 无量纲强制转换，软件不做隐藏换算）。

## 5 几何参数

| 参数分组 | 参数 | 默认值（mm） | 含义与填写说明 |
| --- | --- | --- | --- |
| 封头形状 | k | 2 | 仅半椭球封头：长短轴比（k>0，常用 1.5–3.0） |
| 封头形状 | α0 | 10 | 仅等张力封头：赤道开孔角（0°<α0<54.74°，常用 8°–15°） |
| 封头形状 | 无 | — | 半球封头无需形状参数 |
| 筒身 | R | 135 | 筒身内半径，须大于 r_sp；外半径 R_out = R + t1 自动派生 |
| 筒身 | l1 | 1282 | 直筒段长度 |
| 筒身 | t1 | 5 | 直筒段内衬厚度 |
| 筒身 | L | 1520 | 两端极孔外端面总长（半球模式建议 1600，切换时软件自动联动调整） |
| 内衬肩部 | r_so / r_si | 30 / 28 | 肩部外/内圆角半径，须满足 r_si < r_so |
| 内衬肩部 | r_c | 50 | 内衬回折壁圆角半径，须满足 r_f < r_c < r_sp |
| 内衬肩部 | l_tip | 22 | 环端面相对切点的轴向偏移 |
| 极孔配合面 | r_sp | 90 | 封头段与接头段分界半径，须满足 r_c < r_sp < R |
| 极孔配合面 | r_f | 43 | 内衬与阀座配合界面环半径 |
| 极孔配合面 | r_if | 35 | 界面圆角半径 |
| 金属接头 | r0 | 16 | 阀座中心通孔半径（实际内径为 2r0） |
| 金属接头 | t4 | 20 | 阀座颈部壁厚，颈部外半径 r_n = r0 + t4 自动派生 |
| 金属接头 | r_nf | 12 | 颈部与封头外曲线过渡圆角半径（保留至少 3 mm 颈部直段） |
| 金属接头 | r_bf | 6 | 通孔内底倒角半径，须小于 t4 |
| 输出 | Sector angle | 10° | 仅 3D：回转角度，360° 为整周 |
| 输出 | Boolean merge | 不勾选 | 是否合并生成 Vessel_Base |

半径次序链与派生量：软件在内核中强制施行单调递增判据 `0 < r0 < r_n < r_f < r_c < r_sp < R < R_out`；封头深度 l2 必须落在合理物理区间（不小于接头顶部高度，且不大于局部包络高度 H＝(L−l1)/2 减 1 mm，且 H≥10 mm）。

## 6 校验与输出

软件在调用绘图算子前执行严格的参数流水线校验，任何一级未通过均安全拦截并弹出明确说明，主要包括：

1. R、l1、t1、L、Sector angle 均为正数；
2. 半径单调次序链 0 < r0 < r_n < r_f < r_c < r_sp < R < R_out；
3. 肩部直段裕度与内外圆角关系（r_si < r_so 且 R − r_so − r_c − r_si > 0）；
4. l_tip 偏移界限 l_tip > −(r_so − r_si)；
5. 底倒角约束 r_bf < t4；
6. 端面局部高度 H ≥ 10 mm；
7. 等张力极限 0 < α0 < 54.74° 且轨迹常数 r_cl 满足相容性；
8. 封头深度界限：接头顶部高度 < l2 < H − 1 mm；
9. 颈圆角相切二分求解成功且保留 ≥ 3 mm 直段。

输出行为：

- 二维轴对称创建模型 `TypeIV_Vessel_Axi`、三维实体创建模型 `TypeIV_Vessel`（同名自动加序号后缀）；
- 未勾选合并时输出 Liner 与 Boss 两个独立部件（含 SET_LINER / SET_BOSS 几何集合）；勾选合并时额外输出 Vessel_Base；
- 所有部件创建后自动执行高等级几何拓扑自检；
- 构建完成后在 CAE 消息区打印以 `TYPE4_BUILD_OK` 开头的 JSON 回显（各部件体积与截面面积）；
- 插件不自动保存 CAE 数据库，请使用 File → Save / Save As 手动保存。

## 7 插件文件组成

| 文件 | 职责 |
| --- | --- |
| type4_plugin.py | 向 Abaqus 图形环境注册主菜单按钮与动作监听 |
| type4_builder_db.py | 基于 AFXForm / AFXDataDialog 构建双标签页交互视窗，参数联动与关键字采集 |
| type4_builder_kernel.py | 核心几何运算库：参数校验、子午面解析段与数值积分段生成、实体回转构建 |

## 8 常见问题

- **菜单栏找不到插件入口**：确认 `type4_vessel_builder` 文件夹层级完整置于插件目录，并清理历史残留版本后重启 CAE；
- **切换至半球模式后总长自动变为 1600 mm**：内置几何防干涉保护动作，属正常设计行为；
- **k 或 α0 无法输入**：该参数在当前封头类型下处于禁用状态，先切换封头类型；
- **模型树中未显示装配实例**：未勾选布尔合并时仅生成独立 Part，需在 Assembly 模块手动创建实例；
- **退出软件后模型丢失**：插件不自动写盘，请务必通过 File 菜单手动保存。

## 9 使用与联系

- 非商业用途可直接使用。功能正在升级，想提前体验的请联系 **Z25150070@s.upc.edu.cn**。
- 问题反馈：欢迎提交 [GitHub Issue](https://github.com/IY724/Type-IV-Vessel-Builder/issues)，或发送邮件至 **Z25150070@s.upc.edu.cn**。
