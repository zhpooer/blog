title: 传智播客day43-ssh综合练习 仓库管理系统
date: 2014-06-03 09:05:17
tags:
- 传智播客
---

# 仓库管理系统的分析 #

项目开发两种:
1. 前端人员制作页面(CSS+DIV/Frame)
2. Java工程师开发前端页面(UI框架 ExtJS, JqueryEasyUI, flex),
同时开发服务器端

导入页面新项目:
* 新建项目, 将代码复制(不建议 import), 不同开发工具, 结构不同
* 使用 JavaEE5 版本

功能
* 用户登陆功能
* 仓库管理(仓库CURD), 单表的增删改查
* 货物的入库管理
* 货物的出库管理
* 库存管理(多条件的组合查询, 分页效果)
* 历史记录管理

开发目标:
1. 实现所有功能
2. 能够熟练开发 ssh, 完成 多表, 单表增删改查, 多条件分页查询, Ajax功能

# 项目开发流程 #
采用 瀑布开发模型(自上而下)
1. 需求调研和需求分析
2. 项目分析设计
  1. 页面原型制作(CSS+DIV, Frameset, ExtJS, EasyUI), 为了和客户 确认需求
  2. 数据库设计(E-R图, 分析图, 企业不需要),
  使用 PowerDesigner 数据库建模工具(概念数据模型CDM, 物理数据模型PDM, 面向对象模型OOM)
  3. 软件功能进行分析设计(UML同一建模语言)
          用例图, 描述系统功能需求
          时序图, 活动图, 系统流程
          类图, 程序结构
          工具: Ration Rose, StarUML, Jude, 生成基础程序代码
3. 系统技术选项, struts2+spring3+Hibernate3+JQuery+C3P0+MySQL+Tomcat
  1. 程序代码编写
  2. 代码测试
  3. 项目实施部署, 后期运维
  
# 数据库建模 #
仓库管理模块分析, 需要四张表
* Userinfo 用户信息表
* Store 仓库表
* Goods 货物表
* History 进出库历史数据

关系:
一个仓库存放多种货物
一个货物进出库产生多条历史记录

企业进行数据库设计, 通过 PowerDesigner 完成, 生成sql脚本, 经典版本 12.5
* CDM, 概念模型, 类似 E-R图分析
* PDM, 表结构, 直接生成 sql 语句(入口)
* OOM 面向对象模型, 类图, 直接生成程序代码
* BPM 业务流程模型, 流程设计

![PowerDesigner](/img/pd_ssh.png)

~~~~~~
create database store character set UTF8;
use store;
create table userinfo( /*用户表*/
   id varchar(32) primary key,
   name varchar(50),
   password varchar(32)
);
/*仓库表*/
create table store(
	id varchar(32) primary key,
	name varchar(32),/*仓库名称*/
	addr varchar(100),/*仓库所在地*/
	manager varchar(32) /*仓库管理人员，不关联userinfo表*/
);

/*货物表*/
create table goods(
	id varchar(32) primary key,
	name varchar(50),/*货物名称*/
	nm   varchar(10),/*货物简记内码,如阿斯匹林为ASPL*/
	unit varchar(10),   /*计量单位,1:个，2:GK,3:只，..*/
	amount numeric(10,2),/*库存数量*/
	storeid varchar(32),/*所在仓库ID*/
	constraint foreign key(storeid) references store(id)
);
/*出入库历史记录*/
create table history(
	id varchar(32) primary key,
	goodsid varchar(32),
	datetime varchar(19),/*出入库时间*/
	_type   char(1),/*类型1:入库，２：出库*/
	amount numeric(10,2),/*这次出入库的数量*/
	remain numeric(10,2),/*余量*/
	_user varchar(50),   /*操作员名称，直接保存名称，不引用userinfo表*/
	constraint foreign key(goodsid) references goods(id)
);
~~~~~~

# 登陆页面 #
~~~~~~
package web.action
public class LoginAction extends ActionSupport implements ModelDriver{
    private Userinfo userinfo = new Userinfo();
    private static final Logger LOG = Logger.getLogger(LoginAction.class);

    @Override public Userinfo getModel(){
        return userinfo;
    }

    public String execute(){
        Userinfo loinUser = userinfoService.login(userinfo);
        if(loginUser==null) {
            this.addActionError(getText("loginfail"));
            return INPUT;
        } else {
            LOG.info("登陆");
            ServletActionContext.getRequest().getSession().setAttribute("user", loginUser);
            return SUCCESS;
        }
    }
}
~~~~~~

~~~~~~

<action name="login" class="LoginAction">
    <result name="input"> login.jsp </result>
    <result> main.jsp </result>
</action>

// LoginAction-login-validation.xml
<validators>
    <field name="name">
        <field-validator type="requiredstring">
            <message key="name.required"></message>
        </field-validator>
    </field>
    <field name="password">
        <field-validator type="requiredstring">
            <message key="password.required"></message>
        </field-validator>
    </field>
</validators>
~~~~~~

~~~~~~
// LoginAction.properties
password.required=密码不能为空
name.required=用户名不能为空
~~~~~~

## 业务层 ##

~~~~~~
public class UserinfoServiceImpl implements UserinfoService{
    public Userinfo login(Userinfo user) {
        return userinfoDao.login(user);
    }
}

public class UserinfoDaoImpl impletments UserinfoDao{
    public Userinfo login(Userinfo userinfo) {
        template.find("from Userinfo where name=? and passwor?",
            userinfo.getName(), MD5Util.encypt(userinfo.getPassword()));
    }
}
~~~~~~

# 添加仓库 #

~~~~~~
public class StoreActoin extends ActoinSupport implements ModelDirver {
    private Store store = new Store();
    public String add(){
        return "addSuccess";
    }
}

public class StoreServiceImpl implements StoreService {
    public void saveStore(Store store) {
        storeDao.saveStore(store);
    }
}

public class StoreDaoImpl implements StoreDao {
    public void saveStore(Store store) {
        template.save(store);
    }
}
~~~~~~

~~~~~~
<action name="store_*" class="StoreAction">
    <result name="addSuccess" type="redirectAction"> stort_list </result>
</action>
~~~~~~

# 仓库列表查询 #
仓库列表页面: jsp/store/store.jsp

~~~~~~java:
// storeActoin
public String list(){
    List<Store> stores = storeService.findAllService();
    ActionContext.getContext().put("stores", stores);
    return "listSuccess";   // storeList.js
}

// StoreService
public List<Store> findAllStores(){
    return storeDao.findAll();
}
// <query name="store-query-all"> from Store </query>
// StoreDao
public List<Store> findAll(){
    template.findByNamedQuery("Store.findAll").list();
}
~~~~~~

# 仓库删除和仓库修改 #

脑补


# 窗口嵌套 #

~~~~~~javascript:
if(window.self != window.top){
    window.top.location.href = "${pageContext.request.contentPath}/login.jsp";
}
~~~~~~

# 货物入库操作 #

登记货物入库, 将货物与仓库进行关联, 如果仓库中存在货物,
在原有的数量上增加, 如果没有就新建. 还有进行历史记录的保存.

## 仓库下拉列表的制作 ##

做法一: 先执行 Action 查询所有仓库, 跳转 jsp 通过 <s:select> 生成下拉列表

做法二: 先加载 jsp, 通过Ajax查询所有仓库内容, 返回json, 显示下拉列表

~~~~~~javascript:
$(funciton(){
    $.get("${contextPath}/store_ajaxlist.action", function(data){
        $(data).each(function(){
            var $option = $("<option value='" + this.id + "'></option>")
            $("#selectOption").append($option);
        })
    });
})
~~~~~~

~~~~~~
// storeAction
public String ajaxlist(){
    List<Store> stores = storeService.findAllStores();
    
    // 方案一: flexJson
    JSONSerializer jsonSerializer = new JSONSerializer();
    String json = jsonSerializer.serialize(stores);
    resopons.setContentType("text/json;charset=utf-8");
    response.getWriter().print(json);
    
    // 方案二: struts-json-plugin
    ActionContext.getContext().getValueStack().push(store);
    return "ajaxSuccess"
}
public class Store{
    // 不序列化 Goodses
    @Json(serialize=false)
    public Set getGoodses(){
    }
}
~~~~~~

~~~~~~
<!-- struts.xml  -->
<package name="default" namespace="/" extends="jsondefault">
    <action name="">
        <result name="ajaxSuccess" type="json"></result>
    </action>
</package>
~~~~~~

## 仓库列表缓存使用 ##

如果项目中不经过改变, 经常进行入库操作, 每次入库操作都显现下拉列表,

如果将仓库列表放入缓存, 在下次查询, 调用缓存, 不需要查询数据

Spring直接继承 Ehcache, 导入 `context-support.jar`

~~~~~~
<beans xmlns:cache="http://www.springframework.org/schema/cache">
<!-- 缓存工厂 -->
    <bean id="ehCahceFactory" class="EhCacheManagerFactoryBean">
    </bean>
    <!-- 缓存管理器 -->
    <bean id="ehCacheManager" class="EhCacheManager">
    </bean>
    <!-- 注解驱动缓存 -->
    <!-- todo -->
</beans>

<!-- 配置默认缓存区域 -->
<!-- ehcache.xml -->
<cache name="store" maxElementInMemory="1000">
    <!-- any other config -->
</cache>
~~~~~~

使用 spring 缓存注解
1. `@Cacheable`, 负责将缓存
2. `@CacheEvict`, 负责将缓存中的数据清空

~~~~~~
// service, 放入缓存 
@Cachable(value="store")
public List<> findAllStore(){}

// 调用后清理缓存
@CacheEvict(value="store", allEntries=true)
public void saveStore(){
}
~~~~~~

## 简记码的数据回显 ##

~~~~~~
<!-- 隐藏域id,用来标识货物是否存在, 当用户输入 后, 把查询到的货物id存入 -->
<s:hidden name="id" id="goodeId"></s:hidden>
~~~~~~

~~~~~~
$("input[name='nm']").blur(function(){
     // 发送请求给服务器, 判断简记码是否存在
     $.post("${contextPath}/goods_findByNm.action", {"nm": $(this).val}, function(data){
         if(data==null) {
             // 清空表单, 但是不能隐藏域
             // $('form[name='select]').get(0).reset(); // 会出bug, 手动清空
             $("#goodsId").val("");
         } else {
             // 查询到货物
             $("#goodsId").val(data.id);
             $("input[name='name']").val(data.name);
             $("input[name=unit']").val(data.unit);
             $("#storeSelect").val(data.store.id); // 通过val函数, 使选中
         }
     });
});
~~~~~~

~~~~~~
public class GoodsAction extends ActioinSupport implements ModelDriver<Goods> {
    // 模型驱动
    private Goods goods = new Goods();
    // 业务犯法根据, 简记码查询
    public String findByNum(){
        // 会报错, 可以用 OpenSessionInView
        Goods goodsResult = goodsService.findByNm(goods.getNm());
        return "findByNmSuccess"; // result type=json
    }
}
public class GoodsDao {
    public Good findByNm(String nm) {
        template.findByNamedQeury("Goods.findByNm", nm);
    }
}
~~~~~~

## 入库操作 ##

~~~~~~
// GoodsAction
public String save(){
    Userinfo user = session.get("user");
    goodsService.saveGoods(goods);
    return "saveSccess";  // remain.jsp
}

// GoodeService, goods history -> uuid
public void saveGoods(Goods goods, Userinfo user) {
    // 判断页面是否传递货物 id
    // 如果goods 里id不为空, 说明货物已经存在, 只需要修改数量
    // apache common-lang 工具
    if(StingUtils.isNotBlank(goods.getId())){
        // 修改数量----update
        Goods persistGoods = goodsDao.findById(goods.getId());
        persistGoods.setAmount(persistGoods.getAmount() + goods.getAmount());
        History history = new History();
        history.setUser(user.getName());
        history.setType("1");
        history.setDatetime(new Date().toLocaleString);
        history.setGoods(persistGoods);
        history.setAmount(goods.getAmount());
        history.setRemain(persistGoods.getAmount());// 余量
        historyDao.save(history);
    } else {
        // 新货物入库
        goodsDao.save(goods);

        History history = new History();
        history.setUser(user.getName());
        history.setType("1");
        history.setDatetime(new Date().toLocaleString);
        history.setGoods(persistGoods);
        history.setAmount(goods.getAmount());
        history.setRemain(goods.getAmount());// 余量
        historyDao.save(history);
    }
}
~~~~~~

## 货物名自动补全 ##

使用 Jquery UI 库完成

~~~~~~
// 静态, 一次性查询所有货物
$("input[name='name']").autocomplete({
    source: []
});
// 动态, 根据用户当前输入, 自动补全
$("input[name='name']").autocomplete({
    source: function(request, response) {
        // 获取用户当前输入的字符
        var term = request.term;
        $.post("goods_findNameLick", {name: term}, function(data){
            // 回显数据
            response(data);
        })
    },
    // 选择补全列表回显 
    select: function(event, ui){
        // ui.item可以是 source 绑定数组的某个元素
        // ui.item.value 返回选中值
        // TODO 进行回显
    }
});
~~~~~~

~~~~~~
// GoodsAction
public String findNameLike(){
    goodsService.findNameLike(name);
    return "findNameLickSuccess"; // type json
}

// GoodsDao
public List<Goods> findNameLike(){
    String hql = "from Goods where name lick ?";
    return template.findQuery(hql, "%" + hql + "%").list();
}

// Goods
public String getValue(){
    return name;
}
~~~~~~


# 多条件组合分页查询 #

1. 修改库存管理的查询表单


## 服务器完成分页条件查询 ##

~~~~~~
// Pagination
package page
public class Pagination{
    private int pageno; // 页码
    private int numberPerPage = 10; // 每页记录条数

    private Map parameterMap;// request 提供的方法

    // 结果显示数据
    private long totalCount; /// 总记录数
    private int totalPageCount; //总页数

    privaate List data; // 当前显示数据

    public Strinig getParamUrl(){
        for(Entry<String, String[]> entry : parameterMap.entrySet()){
             // TODO 
        }
    }
}

// option value= 请选择
// GoodAction
public String pageQuery(){
     Pagination pagination = new Pagination();
     if(req.getParameter("pageno") == null) {
         pagination.setPageno(1);
     } else {
         pagination.setPageinfo(Integer.parseInt(req.getParameter("pageno")));
     }
     pagination.setParameterMap(req.getParameterMap());
     goodsService.findPageData(pagination);
     return "pageQuerySuccess";
}

public void findPageData(Pagination pagination){
    Map pMap = pagination.getParameterMap();
    DetachedCriteria detachedCriteria = DetachedCriteria.forClass(Goods.class);

    // QBC 单表条件 TODO, pmap.get()[0], 可能会报错
    if(StringUtis.isNotBlank(pMap.get("nm")[0])) {
        detachedCriteria.add(Restriction.eq("nm", pMap.get("nm")));
    }
    if(StringUtis.isNotBlank(pMap.get("name"))) {
        detachedCriteria.add(Restriction.lick("name", "%" + pMap.get("name")) + "%");
    }
    
    if(StringUtis.isNotBlank(pMap.get("store.id"))) {
        Store s = new Store();
        store.setId(pMap.get("store.id"));
        detachedCriteria.add(Restriction.eq("store", store);
    }
    // 设置投影, select count(*)
    detachedCriteria.setProjection(Projections.rowCount()); 
    Long count = goodsDao.findTotalCount(detachedCriteria);
    pagination.setTotalCount(totalCount);

    int totalPageCount = (totalCount + pagination.getNumberPerPage()-1) / pagination.getNumberPerpage();
    
    // 分页代码查询
    int firstResult = (paination.getPageno()-1)*pagination.getNumberPerpate();
    int maxResults = pagination.getNumberPerpage();
    
    // 清除投影效果
    detachedCriteria.setProjection(null); 
    List<Goods> goodses = goodsDao.findPageData(detachedCriteria, firstResult, maxResult )
    pagination.setData(goods);
}
// GoodsDao
public long findTotalCount(DetachedCriteria criteria){
    return template.findByCriteria(criteria).get(0);
}
~~~~~~

# Get编码过滤器 #

TODO

问题: 如果开发需要在url中拼接中文, 手动对url编码
1. `response.encodeUrl()`
2. `c:url` 在页面中对url编码
3. `UrlEncoder.encode(paramValue)`

# 自定义标签(分页工具) #

自定义标签最常见应用: 权限控制(控制按钮没有权限不显示), 分页工具条

~~~~~~
public class PaginationTag extends SimpleTagSuppport{
    private int pageno; // 当前页码
    private long totalCoiunt; // 总记录上
    private int numberPerpage; // 每页记录数
    private String paramUrl; .// 查询条件的参数url
    
    @Override public void doTag() {
        JspWriter out = getJspContext().getOut();
        // 生成div输出分页工具条
        out.write("<a href=>" + request.getContextPath() + "div");
        
        // 输出页码, 总记录数, 没页记录数
        // TODO
        // 以上太麻烦, 可以用freemark
    }
}
~~~~~~

# 历史记录查询 #
查询条件来自 goods 表字段, 查询结果来自 history表字段

~~~~~~
public class HistoryAction extends ActionSupport implements ModelDriven {
    private History history = new History();
    private HistoryService historyService;
    
    public String pageQuery(){
        // 根据请求的参数, 封装 pagination
        Pagination pagination = new Pagination();
        String pageno = ServletActionContext.getRequest().getParameter("pageno");
        if(pageno==null){
            pagination.setPageno(1);
        } else {
            pagination.setPageno(Integer.parseInt(pageno));
        }
        // 参数
        pagination.setParameterMap(SevletActionContext.getRequest().getParameterMap());
        historyService..findPageData(pagination);
        return "pageQeurySuccess";
    }
}

public class HistoryServiceImpl implements HistoryService{
    public void findPageData(Pagination pagination){
        // 将参数转换为 查询 sql, 使用QBC
        Map<String, String[]> parameterMap = pagination.getParameterMap();
        DetachedCriteria detachedCriteria = DetachedCriteria.forClass(History.class);
        // 添加条件
        // 如果查询根对象和查询条件位于同一张表中, 单表查询
        // 跟对象history ,条件goods --- 多表查询
        String nm = getValueFromMap(parameterMap, "goods.nm");
        String name = getValueFromMap(parameterMap, "goods.name");
        String storeid = getValueFromMap(parameterMap, "goods.store.id");

        // 多表查询
        detachedCriteria.createAlias("goods", "g", Criteria.INNER_JOIN);
        // 三表关联 detachedCriteria.createAlias("g.store", "s", Criteria.INNER_JOIN);
        
        if(String Utils.isNotBlank(nm)) {
            // 根据货物的简记码查询
            detachedCriteria.add(Restriction.eq("g.nm", nm));
        }
        if(String Utils.isNotBlank(name)) {
            detachedCriteria.add(Restriction.like("g.name", "%"+ name + "%"));
        }
        if(String Utils.isNotBlank(name)) {
            Store store = new Store();
            detachedCriteria.add(Restriction.eq("g.store", store));
        }
        detachedCriteria.setProjection(Projections.rowCount());
        long totalCount = historyDao.findTotalCount(detachedCriteria);
        pagination.setTotalCount(totalcount);

        int totalPageCount = TODO;
        pagination.setTotalPageCount();
        // 参照上一次分页代码
        
        //多表关联和投影查询, 返回两个对象的数组
        // 投影改变结果策略, 不再将结果封装到Root数组, 而是根据查询数据类型进行返回
        detachedCriteria.setProjection(null); // 清除了投影, 也重置了封装策略
        // 恢复封装策略
        // AliaToBeanResultTransformer, 组装非 bean 对象
        detachedCriteria.setResultTransformer(Criteria.ROOT_ENTITY);
    }
}

public class HistoryDaoImpl implements HistoryDao {
    // TODO
}
~~~~~~

# paginationTag的freemarker重构 #

在标签类中输出HTML十分不方便

freemarker: 基于模板, 用来生成输出文本的通用工具

原理: ftl模板文件 + 动态数据 = 输出(通常是 html 访问文件)

应用: 页面静态化
~~~~~~
<#-- 这是注释 -->
<html>${title}</html> <!-- 变量 -->
${title ! 默认值}
<#user ??> 判断是否存在
<#list>               <!-- ftl指令 -->

<!-- if指令 -->
<#if animals.python < 100>
    大于
<#else>
    小于
</#if>

<!-- list指令 -->
<#list as being>
   输出${being.price}
</#list>

<!-- 页面包含 -->
<#inlcude "/a.html">

<!-- 内置函数 -->
${test?html} <!-- 转义 -->
${test?cap_first} <!-- 第一个字母大写 -->
${test?lower_case}
${test?size} 
~~~~~~
~~~~~~
Configuration conf = new Configuration();
conf.setDirectoryForTemplateLoading(new File("模板文件文件夹"));

Template tmp = conf.getTemplate("flt文件名");
Map<String, String> dataMap = new HashMap<String, String>();
dataMap.put("title", "freemarker入门");

tmp.process(map, new PrintWriter(System.out));
~~~~~~
