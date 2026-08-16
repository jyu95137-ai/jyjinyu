# 三维模型接入说明

## 主模型文件

- 文件名：`digestive-system.glb`
- 放置位置：`public/models/digestive-system.glb`
- 当前文件：由本机“人体八大系统”项目的 BodyParts3D、Z-Anatomy 咽腔与 NIH 3D / Human Reference Atlas 大肠模型组合生成。
- 当前大小：25,102,700 字节（约 25.1MB），共 116 个真实解剖网格（103 个 BodyParts3D 元素 + 3 个 Z-Anatomy 咽腔网格 + 10 个 HRA 大肠网格）。
- HRA 大肠：来自 NIH 3D `3DPX-021005` 的 `3D Reference Organ for Large intestine, Male v1.2`，包含盲肠、回盲瓣、阑尾、升/横/降结肠、肝曲、脾曲、乙状结肠和直肠。
- 传输优化：装配时以 0.000001 单位容差为属性一致的重复顶点建立索引；不删除三角面、不量化坐标。HRA 大肠只使用统一缩放、旋转和平移与现有人体对齐，不做非均匀拉伸。
- 来源和署名：见项目根目录 `CREDITS.md`。
- 重新生成：运行 `npm run models:assemble`；可用 `BODYPARTS3D_MODELS`、`BODYPARTS3D_ISA_MODELS`、`BODYPARTS3D_PARTOF_MODELS`、`BODYPARTS3D_PARTOF_DEFINITIONS`、`Z_ANATOMY_VISCERAL_MODEL` 和 `HRA_LARGE_INTESTINE_MODEL` 环境变量覆盖源路径。

## 推荐节点名称

主模型建议包含独立、可点击的节点：

- `BodyShell`
- `Mouth`
- `SalivaryGlands`
- `Pharynx`
- `PharynxMuscles`
- `Esophagus`
- `Stomach`
- `Liver`
- `Gallbladder`
- `Pancreas`
- `SmallIntestine`
- `LargeIntestine`
- `Rectum`
- `ExternalAnalSphincter`

HRA 大肠父节点为 `HraLargeIntestineAlignment`，其中保留 `VH_M_caecum`、`VH_M_ileocecal_valve`、`VH_M_sigmoid_colon`、`VH_M_rectum` 等原始分段名称；点击识别仍通过 `LargeIntestine` 与 `Rectum` 标准父节点完成。

程序同时识别常见下划线写法、部分英文别名和中文名称。推荐仍使用上面的标准英文名称，避免后续维护歧义。

## 当前覆盖边界

- 已覆盖：BodyParts3D `FJ2810` 真实皮肤网格裁取的半透明 `BodyShell`、完整口腔、唾液腺、鼻咽/口咽/喉咽三段完整咽腔、咽部肌群参照、食管、胃、完整肝脏、胆囊、胰腺、十二指肠、空肠、回肠、小肠系膜；HRA 提供连续的盲肠、回盲瓣、阑尾、升结肠、肝曲、横结肠、脾曲、降结肠、乙状结肠和直肠；另保留 BodyParts3D 肛门外括约肌参照。
- 待补齐：完整肛管；外括约肌局部参照不会标成完整肛管。
- `BodyShell` 只保留原始坐标 Z=650–1642mm、|X|≤340mm 范围内的真实三角面，初始透明度 0.08；它用于人体位置参照，不参与器官点击和器官镜头归一化。
- HRA 大肠以直肠中心和回盲部中心两个同名解剖地标对齐，替换旧的 BodyParts3D 不完整大肠组合；相邻结肠段共享边界或接缝小于 0.2mm。它仍不包含完整肛管，因此网页不会把外括约肌参照称为完整肛门。

## Blender 导出要求

1. 删除相机、灯光、测试平面和不可见辅助对象。
2. 对模型应用 Rotation、Scale 和必要的 Modifier。
3. 保持器官为独立对象或位于名称清晰的父节点下，不要把所有器官合并成一个无法识别的网格。
4. 导出格式选择 glTF Binary（`.glb`）。
5. 勾选材质、法线和必要纹理；动画仅在确有教学用途时导出。
6. 若使用骨骼或蒙皮，确保骨骼和权重随文件一起导出。

## 单位和坐标

- Blender Unit System：Metric。
- 推荐单位：米；人物真实高度可按约 1.6–1.8 米制作，应用会按整体包围盒归一化显示。
- Y 轴向上，人体面向 +Z 或在交付说明中明确正面方向。
- 人体躯干几何中心靠近世界原点，足端向 -Y，头部向 +Y。
- 不要让器官散落在多个相距很远的坐标原点。

## 科学性和空间关系

- 口腔、咽、食管、胃、小肠、大肠、直肠和肛门应形成连续消化道。
- 胃位于上腹部偏左，肝脏主要位于右上腹，胆囊位于肝脏下方。
- 胰腺位于胃后下方附近，小肠盘曲于腹腔中央，大肠基本包围小肠。
- `BodyShell` 应提供半透明人体躯干参照，不得遮蔽消化系统观察。

## 材质要求

- 使用 PBR 材质，避免纯镜面、过强金属度和玩具塑料质感。
- 推荐 roughness 0.45–0.75，metalness 接近 0。
- 纹理应内嵌或放在项目内，不得依赖外部 CDN。
- 不依赖发光材质照亮器官，场景已有环境光、主光和轮廓光。

## 检查节点名称

在 Blender Outliner 中逐项检查对象名。接入网页后，浏览器控制台会输出：

```text
[DigestiveModel] GLB nodes: [...]
```

将输出与推荐节点表对照。若模型使用其他名称，请在 `src/data/modelNodeMap.ts` 中补充别名，并运行测试。

## 后续小肠资源

- `vh-m-small-intestine.glb`：NIH 3D / HuBMAP Visible Human 男性完整小肠参考器官，中央主模型用它替换原来断裂的小肠段；来源 `https://3d.nih.gov/entries/3DPX-021017`，CC BY 4.0。

- `intestinal-cross-section.glb`
- `intestinal-fold.glb`
- `villus.glb`
- `villus-microstructure.glb`（需包含绒毛内毛细血管与毛细淋巴管）

小肠整体层级已复用 NIH/HuBMAP 小肠 GLB。学生主路径只保留整体小肠、横切面、环形皱襞和绒毛四级；横切面与环形皱襞已接入经教材依据复核的二维教学图，并明确标注“非真实比例、非三维医学模型”，不冒充真实 GLB。

肠上皮杯状细胞显微浮雕和毛细淋巴管运输属于教师拓展素材，不进入学生主路径；现有 GLB 仍保留在 `public/models/microstructure/` 作为独立、非完整绒毛资源，不会冒充初中科学所需的完整结构。
