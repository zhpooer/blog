title: 传智播客day47-Jquery EasyUI2
date: 2014-06-13 09:09:05
tags:
- 传智播客
---

# 完善系统菜单 #
~~~~~~
var simplenodes = [
   // id是当前节点编号, pid是父节点编号
   {name: "系统功能", id:1, pId:0},
   {name:"部门管理", id: 2, pId:1, url:"department.jsp"},
   {name:"员工管理", id: 3, pId:1, url:"employee.jsp"}
]
$.fn.zTree.init($("#simpleTree"), simplesettings, simplenodes);
~~~~~~

# 数据表格
数据表格, 生成table组件, 具有编辑, 显示, 排序的基本功能

~~~~~~
<!-- 完全针对html本地table数据 -->
<table class="easyui-datagrid">
    <!-- 表头 -->
    <thead>
    <tr>
        <!-- field主要用于远程json数据的匹配-->
        <th data-options="field:'id', width:200">编号</th>
        <th data-options="field:'name'"> 名称</th>
        <th data-options="field:'price'"> 价格</th>
    </tr>
    </thead>
    <tbody>
    <tr>
        <td>100 </td>
        <td>手机 </td>
        <td>1999 </td>
    </tr>
    </tbody>
</table>

<!-- 远程json数据, 本地html定义 -->
<!-- pagination: true 设置分页条-->
<table class="easyui-datagrid" data-options="url:data.json, pagination:true">
    <thead>
       <tr>
         <!-- field主要用于远程json数据的匹配-->
         <th data-options="field:'id', width:200">编号</th>
         <th data-options="field:'name'"> 名称</th>
         <th data-options="field:'price'"> 价格</th>
       </tr>
    </thead>
</table>

<!-- 远程json数据, js定义datagrid -->
<script type="text/javascript">
    $(function(){
       // js定义表格
       $('#grid').datagrid({
           columns: [[
           // field 用于匹配远程json属性, width宽度, align居中
           {
               field: "id",
               title: "编号", 
               width: 100,
               align: center,
               // 复选效果, 不设置singleSelect
               checkbox: true
           },
           {
               field: "price",
               title: "价格"
               width: 100,
               align: center
           }
           ]],
           url: "data.json",
           // 只允许单选a
           singleSelect: true,
           // 显示行号
           rownumbers: true,
           // 工具栏, 表格上方工具按钮
           toolbar: [
               { id: 'save', text:'保存',
                 iconCls:'icon-save',
                 handler:function(){log("save")}
               }, //每一个对象就是一个按钮
           ]
       })
    });
</script>
<table id="grid"></table>
~~~~~~

~~~~~~
// data.json
{
    "total": "28",
    "rows":[
        {"id": 1, "name": "name", "price": "100"}
    ]
}
// 简单格式, 设置pagination: flase时使用
{
    {"id": 1, "name": "name", "price": "100"},
    {"id": 1, "name": "name", "price": "100"}
}

~~~~~~

## 数据表格的分页 ##
采用 Ajax 分页方式, 页面table不动,
所有查询条件都缓存在datagrid数据表格中.
每次查询都是以ajax方式将条件发送到服务器,
服务器返回json

页面第一次加载时, 自动向服务器发送参数,
page当前页码, 当前页数,
服务器只需要返回完整数据格式`{total:"", rows: [{}]}`

每次页面条件改变, 都会发送请求`?page=..&&rows=`, 更新效果

# 员工管理页面 #

~~~~~~
<!-- employee.jsp -->
<body class="easyui-layout">
    <!-- 页面布局, 中间区域显示数据 -->
    <div data-options="region:'center'">
        <table id="grid">
        </table>
    </div>
</body>

<script type="text/javascript">
$(function(){
    $("#grid").datagrid({
        columns:[[
            { field:"id", checkbox: true },
            { field:"name", title:"姓名", width:200 },
            { field:"age", title:"年龄", width:100 },
            { field:"birthday", title:"生日", width:200 },
            { field:"department", title:"部门", width:200,
                formatter: function(value, row, index){
                    // 格式化输出
                    // value 是页面中显示的值
                    // row 整行对应的 json 数据
                    return rowData.department.name;
                }
            },
        ]],
        // 数据来源
        url: "${contextPath}/empPageQuery.action",
        pagination: true,
        rownumbers: true,
        // 填满区域
        fit: true,
        toolbar:[
            {
                id: "save",
                text: "保存员工",
                iconCls: funciton(){
                    log(执行保存);
                }
            }
        ]
    })
})
</script>
~~~~~~

~~~~~~
public class Pagnination {
    // 请求参数
    private int pageno; // 页码
    private int numberPerPage; // 每页记录数
    // 查询结果
    private DetachedCriteria criteria; // 存储任意的查询条件
    // 结果数据
    private long total;
    private List rows;
}

// EmployeeAction
// 接收分页参数
private int page;
private int rows;

// TODO
@Action(value="empPageQuery", result={@Result})
public String empPageQuery(){
     Pagination pagination = new Pagination();
     pagination.setPageno(page);
     pagination.setNumberPerPage(rows);

     DetachedCriteria detachedCriteria = DetachedCriteria.forClass(Employee.class);
     pagination.setDetachedCriteria(detachedCriteria);
     
     employeeService.findPageData(pagination);
     context.push(pagination); // return json
}

public findPageData(Pagination pagination) {
    DetachedCriteria criteria = pagination.getDetachedCriteria();
    criteria.setProjection(Projections.rowCount());
    long total = dao.findTaotal

    criteria.setProjection(null);
    criteria.setResultTransformer(Criteria.ROOT_ENTITY);
    
    int firstResult;
    int maxResult;
    dao.findRowData(criteria);
}
~~~~~~

## 添加功能 ##

可以通过 jquery easyui 进行客户端校验
~~~~~~
<div data-options="region:'east'" style="width:250px;">
    <h3>添加员工</h3>
    <table id="saveEmployeeForm" aciton="saveEmployee.action">
        <!-- 非空校验  -->
        <input type="text" name="name" class="easyui-validatebox" data-options="required: true" />
        <!-- 长度校验 -->
        <input type="password" name="password"
               class="easyui-validatebox" data-options="required: true, validType:'length[3,12]'"/>
        <!-- 数字校验  -->
        <input type="text" name="age"
               class="easyui-numberbox" data-options="required: true, min:18"/>
        <!-- 日期选择框  -->
        <input type="text" name="birthday"
               class="easyui-datebox" data-options="required: true, editable: false"/>
        <!--  mode: remote, 在输入过程中会自动向服务器发送请求, 可以完成自动提示 -->
        <input name="department.id"
               class="easyui-combobox" data-options="valueField:'', textField: '', url:'listDepartment.json'"/>
        <a id="savebtn" href="javascript:void(0)" class="easyui-linkbutton"
            data-optionis="plain:true"></a>
    </table>
</div>
<script type="text/javascript">
$(function(){
    // 会先执行表单的 validate, 然后再提交
    $("saveBtn").click(function(){
        // $("saveEmployeeForm").submit(); 同步提交方式
        // 异步方式, 不是通过ajax,而是通过 iframe来模拟ajax
        $("saveEmpolyeeForm").form('submit',{
            // 可以指定url, 如果不指定, 使用form默认的action
            success: function(data){
                // data是服务器返回的内容, 没有经过处理
                // 处理成功后的函数
                if(data=="Success") {
                    $.messager.alert("信息", "保存员工信息成", "info");
                    // 重置form
                    $("#saveEmployeeForm").get(0).reset(); //或 $("#saveEmployeeForm").form("reset");
                    // 刷新datagrid
                    $("grid").datagrid("reload");
                } else {
                    $.messager.alert("信息", "保存失败", "error")
                }
            }
        })
    })

})
</script>
~~~~~~

~~~~~~
@ParentPackaget("json-default")
@Namespace("/")
@Controller
@Scope("prototype")
public class DepartmentAction extends ActionSupport implements ModelDriver<>{
    private Department department = new Department();
    private DepartmentService departmentService;

    @Action(value="listDepartments" results = {@Results(name="", type="")})
    public String listDepartment() {
        service.findAllDepartments();
    }
}


@Service("departmentService")
@Transactional
public class DepartmentService {
    @Resources
    private DepartmentDao dao;
    // 脑补
}

@Repository("departmentdao")
public class DepartmentDao{
    // 脑补
}

// 保存逻辑, EmployeeAction
@Action(value="saveEmployee")
public String saveEmployee(){
    response.setContentType("text/html;charset=utf-8")
    // 调用业务层完成
    try {
        employeeService.saveEmployee(employee);
        // easyUI 处理的是原始的字符, 所以直接写字符
        response.getWriter().print("Success");
    } catch (Exception e){
        response.getWriter().print("Failed");
    }
    return NONE;
}

~~~~~~

## 修改功能 ##

~~~~~~
toolbar:[
    {
        id: "edit",
        text: "修改员工",
        handler: function(){
            // 获取到表格中用户选中
            var array = $("#grid").datagrid('getSelections');
            if(array.length==0) {
                // 用户一条也没有选
                $message.alert("警告", "修改数据前必须选中一条数据", "warning");
            }
            if(array.length>1){
                $.messager.alert("警告", "只能选中一条数据", "warning");
            }
            var row = array[0];
            // 回显, 有一些不足, 可以手动修改
            $("#saveEmployeeForm").form('load', row);
            // 在页面中给一个隐藏域, id, 默认值为0,
            // 在保存时, 服务器根据id的值, 判断新添加还是更新
            // 页面添加表单重置按钮, 清空表单值,
            // 注意: $("input[name='id']") 需要手动清空
        }
    }
]
~~~~~~

## 员工批量删除 ##

~~~~~~
toolbar: [{
    id: "delete",
    title: "删除员工",
    handler: function() {
        var array = $("#grid").datagrid('getSelections');
        if(array.length==0) {
            $.messager.alert("警告", "必须选中一条数据", "warning");
            return;
        }
        vara idArray = [];
        for(var i=0; i<array.length;i++) {
            idArray.push(array[i].id);
        }
        var ids = idArray.join(", ");
        $.post("${contextPath}/delteEmployees.action", {"ids": ids}, function(data){
            if(data.result=="success") {
            } else {
            }
            // 刷新表格
            $("#grid").datagrid('reload');
        });
    }
}
]
~~~~~~

~~~~~~
// EmployeeAction
private String ids;
@Action(value="deleteEmployees", results={@Result(name="", type="json")})
public String deleteEmployees(){
    // 获取删除员工的id
    String idArray = ids.split(", ");
    
    // 调用业务层完成删除
    service.deleteEmployees(idArray);
    //返回 Map("result" -> "success|failure")
}

~~~~~~

## datagrid 快速编辑 ##
快速编辑, 在表格内编辑

~~~~~~
columns: [[
    {
        field: "id",
        // 具有别编辑的能力
        editor: {
            type: `validatebox`,
            options: {
                reqiure: true
            }
        }
    }
]],
onAfterEdit: function(rowIndex, rowData, changes){
    // 如果不加, 会将仓库修改为空
    rowData[`department.id`] = rowData.department.id;
    // rowData当前编辑后数据
    $.post("${contextPath}/saveEmployee.action", rowData, function(data){
        $.messager.alert("信息", "编辑成功", "info");
    })
}
// 开启编辑状态
var index = undefined;
toolbar: [{
    id: 'lineedit',
    text: '行内编辑',
    handler: function(){
        var array = $('#grid').datagrid('getSelections');
        // 只能在一行中开启
        if(array.length == 1) {
            // 获取选中行
            var row = array[0];
            index = $("grid").datagrid('getRowIndex', row);
            $("#grid").datagrid('beginEditor', index)
        }
    }
},
{
    id: "endedit",
    text: "结束编辑",
    handler: function(){
        $("#grid").datagrid('endEditor', index)
    }
}]
~~~~~~

## datagrid 表格右键自定义菜单 ##
通过 easyui 的 menu 完成自定义菜单, 鼠标右键点击事件 `oncontextmenu`

~~~~~~
<!-- onRowcontetxMenu: function(e, rowIndex, rowData){
    e.preventDefault(); // 屏蔽原事件
    $("#rightMenu").menu("show", {
        left: e.pageX,
        right: e.pageY
    })
}
-->
<div id="rightMenu" class="easyui-menu" style="width:120px">
<div> New </div>
<div> 修改 </div>
<!-- 分割线 -->
<div class="menu-sep"></div>
<div>
    <span>员工管理</span>
    <!-- 子菜单 -->
    <div style="width='120px'">
        <div> 删除员工 </div>
    </div>
</div>
</div>

~~~~~~
## searchbox 搜索框 ##
只支持单条件查询

~~~~~~
<script>
function doSearch(value, name) {
    log("在name中搜索value");
}
</script>
<input type="text" class="easyui-searchbox"
       data-options="menu:#mm, prompt: '请输入您的搜索内容',
       search: doSearch"/>

<!-- 搜索项div -->
<div id="mm">
    <div data-options="name:'city'">按照城市搜索</div>
    <div data-options="name:'name'">按照姓名搜索</div>
</div>
~~~~~~
