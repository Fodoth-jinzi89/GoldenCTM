# GoldenCTM

从两张 32×32 的手绘贴图生成 Create 模组所需的全部连接纹理（CTM）图：

- `*_base.png` —— 底纹，格子相连时露出的重复图案；
- `*_frame.png` —— 外框，决定每种连接状态下边缘怎么画。

脚本把外框按 `6 / 20 / 6` 九宫格切开，组合出 16 种四邻接状态、角块，以及 Create 使用的 47 格八方向（OMNIDIRECTIONAL）总表；再由此无损/有损地派生出其余七种布局。

## 依赖

- Python 3.8+
- [Pillow](https://python-pillow.org/)

```bash
pip install pillow
```

## 目录

```
ctm.py                 全部命令都在这里
templates/             可直接使用的 32×32 底纹 + 外框示例
examples/              由 templates/ 生成的产物，供对照
  shell_pe/
  shell_pp/
  template/
    <name>_base_large.png   底纹重复成 4×4
    <name>_ct.png           4×4 连接状态表（中间透明，让底纹透出）
    <name>_r.png            角块
    <name>_ctm.png          底纹 + 连接状态合成图
    <name>_omni.png         8×8、47 格八方向总表
    layouts/                从 omni 派生出的八种布局
```

## 用法

### 一条龙：底纹 + 外框 → 所有图

```bash
python ctm.py pipeline templates/shell_pe_base.png templates/shell_pe_frame.png out/shell_pe
```

输出 `out/shell_pe_{base_large,ct,r,ctm,omni}.png`。

可选参数：

- `--no-flip-marks`：角标不旋转 180° 后再贴。

### 单独使用各步骤

```bash
python ctm.py corners <frame.png> <out.png>          # 外框 → 角块
python ctm.py omni <ctm.png> <corners.png> <out.png> # 4×4 表 + 角块 → 8×8 八方向表
python ctm.py layout <omni.png> <layout> <out.png>   # 8×8 表 → 指定布局
python ctm.py layouts                                # 列出八种布局及尺寸
```

`omni` 额外支持：

- `--recolour RRGGBB`：把角标统一替换成某个颜色，只保留透明度；
- `--no-flip-marks`：角标不翻转。

### 检查

```bash
python ctm.py verify <sheet.png> [layout]   # 逐个格子核对边缘是否符合连接状态
python ctm.py seams  <omni.png> [w] [h]     # 拼成方块网格，检查内部接缝是否有杂线
python ctm.py show   <sheet.png> <index> [grid]   # 以字符画出某一格
python ctm.py preview <omni.png> <w> <h> [out.png] # 按游戏取图规则预览一整片墙
```

`verify` 能在不打开游戏的情况下抓出索引表或拼接错误；`seams` 用 2×2 网格暴露“多画了角标”的问题。

## 支持的布局

| 布局 | 格子 | 尺寸 |
| --- | --- | --- |
| `HORIZONTAL` | 2×2 | 64×64 |
| `HORIZONTAL_KRYPPERS` | 2×2 | 64×64 |
| `VERTICAL` | 2×2 | 64×64 |
| `OMNIDIRECTIONAL` | 8×8 | 256×256 |
| `ROOF` | 4×4 | 128×128 |
| `ROOF_STAIR` | 4×4 | 128×128 |
| `CROSS` | 4×4 | 128×128 |
| `RECTANGLE` | 4×4 | 128×128 |

`OMNIDIRECTIONAL` 区分 47 种状态，其余布局读取的邻居更少，多个状态会合并成一格。派生时按“表面绕过拐角继续延伸”来选图，因此这一步不可逆。

## 新增一套材质

1. 画两张 32×32 的 PNG：`xxx_base.png`、`xxx_frame.png`；
2. 运行 `pipeline`，检查 `xxx_ctm.png` 与 `xxx_omni.png`；
3. 需要别的布局就继续跑 `layout`；
4. 用 `verify` / `seams` 复核后，把对应图放进资源包。

## 说明

- 外框的切分点会按图测量：实心区足够厚（≥4px）时角块就是实心区宽度，否则按 6px 网格切。
- 四块贴向内部的窗（C/H 横向、D/E 纵向）不是原样复制，而是取中间带向外镜像，让边框的渐变在接缝处连续。
- 角标贴到内角前会旋转 180°，使描边朝向内侧；位置本身不移动。

源码内的注释记录了各条规则的来历，修改索引或拼接逻辑前建议先读一遍。
