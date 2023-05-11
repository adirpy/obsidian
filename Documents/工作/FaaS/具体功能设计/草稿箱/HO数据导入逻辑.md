
##  1. HO数据样例
![iShot_2023-02-06_19.46.35.png - Droplr](https://d.pr/i/uvVdfh+)

这次，我们只导入下列表中的图层数据

对应的映射关系如下：

| 图层名 | 对应资源规格 | 特殊说明 |
| - | - | - |
| co | SITE | |
| 1fcp | F_CLOSURE | ho_box_type = '1fcp'|
| agg | F_CLOSURE | ho_box_type = 'agg' |
| d_sc | F_CLOSURE | ho_box_type = 'd_sc' |
| f_sc | F_CLOSURE | ho_box_type = 'f_sc' |
| fdp | ODB | ho_box_type = 'fdp' |
| fdp_sc | ODB | ho_box_type = 'fdp_sc'|
| cables_feeder | F_CABLE | cable_type = 'TRUNK' |
| D1_cables_1_fcp_agg_distribution | F_CABLE | cable_type = 'BRANCH' |
| D2_cables_agg_fdp_distribution | F_CABLE | cable_type = 'DISTRIBUTION' |
| fdp_boundaries | ODB_GRID |  |
| 1fcp_boundaries | OCC_GRID |  |

HLD不做选型操作。为了产品的普适性，当HO数据导入时，对于设备，光缆，根据ho_box_type和cable_type映射到对应的设计元素上。
而数据中的model和template，则直接写入表中对应字段。
这里要求HO的设计方案上，需要定义一套符合数据的设计元素。


## 2. HLD的版本归档

当HLD单完成提交后，当前HLD任务单的状态改为Closed.

## 3. 自动LLD拆分逻辑

根据Closed的HLD任务单进行查询数据。按照以下规则进行任务创建。

1. CO做一个10米的缓冲区(网格），生成一个LLD任务单， 类型是CO  
2. cables_feeder根据对象中的trunk属性进行分组，每个分组做一个10米的缓冲区，每个trunk生成一个LLD任务单（多个）
3. 1FCP_boundaries，每个生成一个LLD任务单， 类型是Distribution

拆分到LLD任务单的时候：
1. CO放入CO任务单
2. Feeder cable放入Feeder任务单, Feeder任务单的起点为CO（引用CO任务单的CO），末端节点为1FCP
3. 1FCP/f_sc根据其trunk属性放入对应的Feeder任务单
4. 1fcp_boundaries对应的任务单加入1fcp的名称属性，
5. agg/d_sc/fdp/fdp_sc, D1_cablexxx, D3_cable_xxx 根据id_1fcp放入对应的Distribution任务单，起点为1FCP（引用Feeder任务单的1FCP），末端节点为接入点。

‘放入’的概念，是指资源的主体在对应任务单，核心在于:
im_project_res在主体任务单中存在一条数据,is_reference = 'N'，gis数据对应schema_id为当前任务单也存在一条数据。如果有其余任务单范围也包含这个资源，则当前GIS数据的be_ref_flag='Y'.
使用对应数据的任务单引用当前资源，则，im_project_res在引用任务单中也存在一条数据，is_reference = 'Y'。gis数据对应schema_id为当前任务单也存在一条数据。be_ref_flag='N', ref_gis_key和ref_schema_id,ref_sprint_id 写主任务单id和对应的gis对象gis_key和关联信息。



这里所有的GIS数据，从Closed的HLD任务单进行复制一份完整的数据，并放入对应的LLD任务单中，注意：新的GIS对象的schema_id写LLD任务单的ID。


## 4. LLD任务单的初始化

要做的事情：

1. 自动选型：特殊注意的是CO，需要自动。另外，HO需要根据字段来生成对应的内部分光器，逻辑如下：
	* 1FCP中，根据splitters数量，放置对应的1:4分光器，对应信息：T221122135040092 HO_spliter 1:4
	* FDP中，根据splitters数量，放置对应的1:16分光器，对应信息：T221122102836863 HO_spliter 1:16
2. PM单创建
3. Build单生成