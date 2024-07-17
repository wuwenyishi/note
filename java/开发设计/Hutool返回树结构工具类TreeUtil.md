考虑到菜单等需求的普遍性，有用户提交了一个扩展性极好的树状结构实现。这种树状结构可以根据配置文件灵活的定义节点之间的关系，也能很好的兼容关系数据库中数据

数据的实体类如下：



```java
public class MenuInfo implements Serializable {
    private static final long serialVersionUID = -8839460886585312600L;

    @Column(columnDefinition = "varchar(50)  comment '菜单名称'")
    private String menuName;

    @Column(columnDefinition = "varchar(50)  comment '允许操作编码'")
    private String permission;

    @Column(columnDefinition = "varchar(50)  comment '菜单地址'")
    private String path;

    @Column(columnDefinition = "int(1)  comment '父级ID'")
    private Integer parentId;

    @Column(columnDefinition = "varchar(50)  comment '菜单图标'")
    private String icon;

    @Column(columnDefinition = "int(1)  comment '排序'")
    private Integer sort;

    @Column(columnDefinition = "int(1)  comment '路由缓冲'")
    private Integer keepAlive;

    @Column(columnDefinition = "int(1)  comment '菜单类型,0:菜单 1:按钮'")
    private Integer type;

}
```

数据库数据如下：

![](https://xuemingde.com/pages/image/2022/04/29/1021-c2LkKr.png)



查询出来的数据源：

```java
List<MenuInfo> menuInfos = this.menuService.list();
```

将`List<MenuInfo>`转化为树结构：



```java
private List<Tree<Integer>> menuTreeConvert(List<MenuInfo> menuInfos,Integer parentId){
    List<TreeNode<Integer>> nodeList = menuInfos.stream().map(menuInfo -> {
        
       //扩展字段
        Map<String, Object> map = new HashMap<>();
        map.put("type", menuInfo.getType());
        map.put("keepAlive", menuInfo.getKeepAlive());
        map.put("icon", menuInfo.getIcon());
        map.put("path", menuInfo.getPath());
        map.put("sort", menuInfo.getSort());
        map.put("permission", menuInfo.getPermission());
      
        TreeNode<Integer> treeNode = new TreeNode<Integer>()
                 //ID
                .setId(menuInfo.getId())
                .setName(menuInfo.getMenuName())
                .setParentId(menuInfo.getParentId())
                .setWeight(menuInfo.getSort())
                .setExtra(map);
        return treeNode;
    }).collect(Collectors.toList());
    //配置
    TreeNodeConfig treeNodeConfig = new TreeNodeConfig();
    treeNodeConfig.setChildrenKey("children");
    treeNodeConfig.setParentIdKey("parentId");
    treeNodeConfig.setNameKey("name");
    treeNodeConfig.setWeightKey("sort");
    // 最大递归深度
    treeNodeConfig.setDeep(3);
    //转换器
   return TreeUtil.build(nodeList,parentId);
}
```



返回实例：

```json
 [
        {
            "id": 1,
            "parentId": 0,
            "weight": 1,
            "name": "容器管理",
            "path": "/container",
            "keepAlive": 0,
            "icon": "icon-mokuai",
            "permission": "",
            "sort": 1,
            "type": 0
        },
        {
            "id": 2,
            "parentId": 0,
            "weight": 2,
            "name": "样本源管理",
            "path": "/sample_source",
            "keepAlive": 0,
            "icon": "icon-zaixianyonghu",
            "permission": "",
            "sort": 2,
            "type": 0
        },
        {
            "id": 3,
            "parentId": 0,
            "weight": 3,
            "name": "样本管理",
            "path": "/sample_manage",
            "keepAlive": 0,
            "icon": "icon-yangbenzu",
            "permission": "",
            "sort": 3,
            "type": 0
        },
        {
            "id": 4,
            "parentId": 0,
            "weight": 4,
            "name": "随访管理",
            "path": "/followup",
            "keepAlive": 0,
            "icon": "icon-gongyingshang2",
            "permission": "",
            "sort": 4,
            "type": 0
        },
        {
            "id": 5,
            "parentId": 0,
            "weight": 5,
            "name": "数据采集",
            "path": "/plc",
            "keepAlive": 0,
            "icon": "icon-wodeyingyong",
            "permission": "",
            "sort": 5,
            "type": 0
        },
        {
            "id": 6,
            "parentId": 0,
            "weight": 6,
            "name": "权限管理",
            "path": "/user",
            "keepAlive": 0,
            "icon": "icon-renwu",
            "permission": "",
            "sort": 6,
            "type": 0,
            "children": [
                {
                    "id": 7,
                    "parentId": 6,
                    "weight": 1,
                    "name": "菜单管理",
                    "path": "/admin/menu/index",
                    "keepAlive": 0,
                    "icon": "icon-caidan",
                    "permission": "",
                    "sort": 1,
                    "type": 0,
                    "children": [
                        {
                            "id": 8,
                            "parentId": 7,
                            "weight": 1,
                            "name": "菜单查阅",
                            "path": "",
                            "keepAlive": 0,
                            "icon": "",
                            "permission": "sys_menu_view",
                            "sort": 1,
                            "type": 1
                        },
                        {
                            "id": 9,
                            "parentId": 7,
                            "weight": 2,
                            "name": "菜单新增",
                            "path": "",
                            "keepAlive": 0,
                            "icon": "",
                            "permission": "sys_menu_add",
                            "sort": 2,
                            "type": 1
                        },
                        {
                            "id": 10,
                            "parentId": 7,
                            "weight": 3,
                            "name": "菜单修改",
                            "path": "",
                            "keepAlive": 0,
                            "icon": "",
                            "permission": "sys_menu_edit",
                            "sort": 3,
                            "type": 1
                        },
                        {
                            "id": 11,
                            "parentId": 7,
                            "weight": 4,
                            "name": "菜单删除",
                            "path": "",
                            "keepAlive": 0,
                            "icon": "",
                            "permission": "sys_menu_del",
                            "sort": 4,
                            "type": 1
                        }
                    ]
                },
                {
                    "id": 12,
                    "parentId": 6,
                    "weight": 2,
                    "name": "用户管理",
                    "path": "/admin/user/index",
                    "keepAlive": 0,
                    "icon": "icon-wodezhanghu",
                    "permission": "",
                    "sort": 2,
                    "type": 0
                }
            ]
        }
    ]
```

