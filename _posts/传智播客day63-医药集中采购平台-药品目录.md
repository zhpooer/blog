title: 传智播客day63-医药集中采购平台 药品目录
date: 2014-07-25 09:04:29
tags:
- 传智播客
---

# 数据字典 #

系统中定义的类型数据, 在数据字典表定义.

数据字典明细表 (dictinfo)
* info
* typecode 外键关联

数据字典类型表 (dicttype)
* typecode primary key
* dictcode

~~~~~~
-- 选择药品
select * form dictinfo where typecode='001';
~~~~~~

应用
1. 将数据字典明细表中的id存储在业务表的字段中, 如医院信息(useryy)中的级别
~~~~~~
select t.*,
  (select info from dictinfo where id=t.jb)
t.rowid from useryy t;
~~~~~~
2. 将数据字典表中的dictcode字段存储在业务表中, 比如系统用户表中的groupid,
对应数据字典表中typecode为 01的 dictcode

~~~~~~
<!-- 系统配置接口 -->
<bean id="" class="SystemConfigServiceImpl">
</bean>
~~~~~~

~~~~~~
public class UserAction {
    @RequestMapping("/userquery")
    public String userquery(Model model) {
        List<DictInfo> grouplist = systemConfgiService.findDictInfoByType("s01");
        model.addAttribut("groupinfo", grouplist)
        return "base/userquery";
    }
}
~~~~~~

将 硬编码判断, 数据库用户预定义类型(1->info,2->info,3->info) 在数据库关联

# 用户身份认证 #


静态密码方式, 用户名对应的密码设置在系统, 以设置一边不再改变,
安全性低容易被木马窃取, 可以定期修改密码, 但是不容易记忆

动态密码: 短信密码

验证码, 防止恶意攻击

~~~~~~
public class LoginAction {
    // 登陆页面
    @RequestMapping("/login")
    public String login(){
        return "/base/login"
    }
    // 登陆提交
    @RequestMapping("/loginsubmit")
    public @ResponseBody SubmitResultInfo loginSubmit(
        HttpSession session, 
        String userid, String pwd, String randomcode 
    ) {
        // 校验验证码是否正确
        String validateCode_session = session.getAttribute("validateCode");
        if(!validateCode_session.equals(randomcode)) {
            ResultUtil.throwException(ResultUtil.createFail(Config.MESSAGE, 113, null));
        }
        // 校验用户身份
        ActiveUser activeUser = userManagerService.userloginCheck(userid, pwd);
        session.setAttribute(Config.ACTIVE_key, activeUser);
        return ResultUtil.createSubmitResult(ResultUtil.createSuccess(Config.Message, 107, new Object[]{activeUser.userId}));
    }
}

// userManagerService

// 校验通过返回用户身份信息
public ActiveUser userloginCheck(String userid, String pwd) {
    Sysuser sysuser = findSysuserByuserid(userid);
    if(sysuser==null) {
        ResultUtil.throwException(ResultUtil.createFail(Config.MESSAGE, 110, null));
    }
    String pwd_md5 = new MD5.getMD5ofStr(pwd);
    if(!sysuser.getPwd().equals(pwd_md5)) {
        // 提示 用户密码不正确
        ResultUtil.throwException(ResultUtil.createFail(Config.MESSAGE, 114, null)); 
    }
    ActiverUser activeUser = new ActiveUser();
    activeUser.setGroupid(sysuser.getGroupid);
    activeUser.setUserId(); // username, sysid
    return activeUser;
}

@Controller
public class First {
   @ResourceMapping("/logout")
   public String logout() {
       // 清除session
       session.setAtrribute("", null);
       return View.recirect("login.action"); // "redirect:login.action"
   }
}
~~~~~~

## 登陆拦截器 ##
`anonimousAction.properties`, 配置用户无需登录就可操作的配置

~~~~~~
login.action
login.submit.action
~~~~~~

~~~~~~
public class LoginInterceptor implements HandlerInterceptor {
    // 进入action方法之前调用
    // 权限管理
    public void preHandler() {
        Activeuser activeUser = session.getAttribute();
        if(activeUser!=null) {
            return true;
        }
        List<String> open_url = ResourcesUtil.getKeyList(Config.ANONYMOUST_ACIOTNS);
        // 获取请求地址
        String url = request.getRequestURI();
        for(String url_l : open_url){
            if(url.contains(url_l)) {
                return true;
            }
        }
        // request.getRequestDispatcher("login.jsp").forword(request, response);
        // 抛出一个异常信息, 由全局异常处理器同一执行
            // 如果异常代码为106, 跳转到登陆页面
        return false;
    }
    // 执行 action 之后, 返回视图之前
    // 给页面添加共有数据
    public void postHandle() {
    }
    // 执行 action 之后
    // 捕获异常, 性能监视
    public void afterCompletion() {
    }
}
~~~~~~

# 药品目录 #

药品目录: 通用名, 价格, 企业名称, 商品名称, 规格, 药品流水号(编号, 通用),
剂型(胶囊, 针剂), 规格(一个胶囊毫克), 转换系数(40片胶囊), 药品状态(1. 正常交易, 暂停交易)

药品目录表的记录id与业务表存在很多的外键关联, 规定药品目录不再删除, 进行逻辑删除,
更新药品交易状态为暂停交易


省级的药品目录通过接口将信息录入到市药品采购平台, 通过 excel 导出, excel 导入

excel需要导入导出模板

## POI ##

操作微软文档的 java api

HSSF, 只支持 excel 97-2003

XSSF, 支持2007以上的版本, `.xlsx`, 文件是基于xml存储的

### 导出 ###

Workbook(工作簿), Sheet, Row(行), Cell(单元格)

~~~~~~
// HSSF 操作
Workbook wb = new HSSFWorkbook();

Sheet sheet = wb.createSheet("new sheet");
sheet.createRow((short)0);
// row 表示单元格所在行, 创建时指定单元格所在列
Cell cell = row.createCell(0);
cell.setValue(1);
~~~~~~

HSSF导出数据如果数据量足够大, 可能会发生内存溢出


~~~~~~
// 关闭自动刷新, 100, 保持100条行在内存
SXSSFWorkbook wb = new SXSSWorkbook(-1);
Sheet sh = createSheet();
Row row = sh.createRow(0);
Cell cell = row.createCell(0);

// 刷入硬盘, 但是保留最后插入的100行
// 会被写入临时文件
(SXSSSheet)sh.flushRows(100); 

wb.dispose(); /// 清除临时文件
~~~~~~

处理数据速度不快, 除非由大数据要求, 否则建议使用 XSSF

## 导出药品目录 ##

~~~~~~

// 药品信息
@Controller
public class YpxxActiona {
    // 药品信息导出页面
    @RequestMapping("/ypxxExport")
    public String ypxxExport(Model model) {
        // 药品类别
        List<Dictinfo> yplbList = systemConfigService.findDictinfoByType("001");
        // 交易状态
        List<Dictinfo> jyztList = systemConfigService.findDictinfoByType("003");
        model.addAttribute("yplbList", yplbList);
        model.addAttribute("jyztList", jyztList);
        
        return View.toBusiness("/ypml/ypxxExport");
    }
    @RequestMapping("/ypxxExportSubmit")
    public @ResponseBody SubmitResultInfo ypxxExportSubmit() {
        String webpath = ypmlservice.ypxxExport(ypxxQueryVo);
        // 导出成功, 点击下载
        return ResultUtil.createSubmitResult(ResultUtil.createSuccess(Config.Message, 318, ))
    }
}

public class YpmlService {
    public List<YpxxCustom> findYpxxList(YpxxQueryVo ypxxQueryVo) {
       return ypxxMapperCustom.findYpxxList(ypxxQeuryVo);
    }
    
    // 药品信息导出
    // 导出后将下载的文件路径返回
    public String ypxxExport(YpxxQeuryVo ypxxQueryVo) {
        List<YpxxCustom> list = this.findYpxxList(ypxxQeuryVo);
        // 调用封装类, 执行导出
        
    }
}
~~~~~~

~~~~~~
<select id="findYpxxList" parameterType="YpxxQueryVO" resultType="YpxxCustom">
    select * from ypxx.*,
        (select form dictinfo where dictcode=ypxx.jyzt and typecode='003') jyztmc
    from ypxx
    <where>
        <if test="ypxxCustom!=null">
           <if test="ypxxCustom.bm!=null and ypxxCustom.bm!=''">
               ypxx.bm=#{ypxxCustom.mc}
           </if>
           and ypxx.mc like ''
           and ypxx.lb=''
           and ypxx.jyzt=''
        </if>
    </where>
</select>
~~~~~~

basicinfo表, 保存系统配置, 如 导出文件的保存路径


## 药品信息的导入 ##

~~~~~~
// 03版本读取, 只能读取xsl的文件,
// 如果使用用户驱动, 如果数据量过大, 内存溢出
// 但是可以使用 事件驱动机制 来避免 内存溢出, 但是速度不快,
// 需要程序员自定义 processRecord 方法去处理读取到的数据
HSSFWorkbook wb = new HSSFWorkbook(inputStream);
wb.getNumberOfSheets();
Sheet sheet = wb.getSheetAt(index);
HSSFRow row = wb.getRow(rownum);
HSSFCell cell = row.getCell();
~~~~~~

使用 XSSF 方式读取 excel 文件, 是基于 sax 方式, 属于事件驱动

~~~~~~
//YpxxAction
@RequestMapping("/ypxximport")
public String ypxximport {
    // 药品信息导入页面
    return View.toBusiness("/ypxl/ypxximport");
}

@ReqeustMappng("/ypxximportstudent")
public @ResponseBody SubmitResultInfo ypxximportsubmit(MultipartFile ypxximportfile) {
    String filename = ypxximportfile.getOriginalFilename();
    File file = new File("d:/upload/" + filename_original);
    if(!file.exist()) {
        file.mkdirs(); // 如果文件所在的路径不存在则创建
    }
    // 将内存中文件的数据写入磁盘
    ypxximportfile.transferTo(file);
    // 导入, 若导入失败, 形成失败记录文件, 返回给用户
}
// 上传多个文件
// public SubmitResultInfo ypxximportsubmit(@RequestParam("ypxximportfile") MultipartFile[] ypxximportfiles)



public class YpxxImportServiceImpl implements HxlsOptRowsInterface {
    // 流水号, 再插入时, 使用触发器自动生成
}
~~~~~~

内连接, 通过外键关联的两个表的查询, 也可以通过子查询,
不是外键关联可能会出现重复记录


# 供货商药品目录 #

供货商将自己供应的药品发布到供应商药品目录(gysypml)
* id
* ypxxid
* usergysid

监督单位可以对供货商对应的药品进行控制, 允许供货或不允许供货(gysypml_control),
记录药品控制的状态(只更新, 不添加)
* id
* ypxxid, 药品信息
* usergysid, 供应商id
* control, 监督机构控制状态(1. 正常, 2. 暂停)
* advice, 监督机构意见

两张表在数据库级别没有关系

供货商药品目录维护, 供货商在ypsypml表中添加, 删除记录
* 添加, 供货商添加自己供应的药品
* 删除, 供货商删除自己不再供应的药品

如果将控制状态设置在供货商商品目录表,
供应商添加一条记录, 监督单位更新一条记录(控制状态更新为暂停),
供货商删除一条记录, 删除的同时将监督单位的控制状态也删除了.
**供货商操作影响了监督单位**

查询供货商药品目录和供货商药品目录控制表时, 通过供货商id和药品信息一块儿关联查询


## 业务操作 ##

* 供货商药品目录查询, 供货商 + 药品信息 + 状态
* 供货商添加药品查询, 药品总目录存在的药品, 但是在供货商目录没有的药品
* 确认添加药品, 到供货商药品目录
* 监督单位对供货商的药品进行控制
  1. 查询所有供货商供应的药品的控制状态
  2. 进行控制, 选择要控制的对象, 选择供货状态, 点击确认提交


## 供货商目录维护 ##

~~~~~~
<select id="findGysypmlList" parameterType="GysypxxQeuryVo" resultType="GysypmlCustom6">
select gysypml.*, ypxx.*,
      (seelct info from dictinfo where 控制状态),
      (seelct info from dictinfo where 交易状态)
    from gysypml, ypxx, gysyml_control
    where gysypml.ypxxid=ypxx.id
      and gysypml.ypxxid=gysyml_control.ypxxid
      and gysypml.usergysid=gysyml_control.usergysid;
    <!-- 药品信息查询条件  -->
    <!-- 控制状态条件 -->
    <!-- 分页条件, 参考 baseuser -->
</select>
<select id="findGysypmlCount" parameterType="GysypxxQeuryVo" resultType="GysypmlCustom6">
select gysypml.*, ypxx.*,
      (seelct info from dictinfo where 控制状态),
      (seelct info from dictinfo where 交易状态)
    from gysypml, ypxx, gysyml_control
    where gysypml.ypxxid=ypxx.id
      and gysypml.ypxxid=gysyml_control.ypxxid
      and gysypml.usergysid=gysyml_control.usergysid;
    <!-- 药品信息查询条件  -->
    <!-- 控制状态条件 -->
</select>
~~~~~~


~~~~~~
// ypmlAction
// 供货商药品目录查询
public String gysypmlquery(Model model, GyspmQeuryVo gyspmlQeuryVo) {
    
    return View.toBusiness("/ypml/gysypmlquery");
}

// 供货商药品目录查询结果集
@ResultMapping("/y")
public @ResponseBody DatagridResultInfo findGysypmlCount(
      GyspmQeuryVo gyspmlQeuryVo
      int page,
      int rows) {
    int count = ypmlService.findGysymlcount(gysypmlQeuryVo);
    // 分页
    PageQuery pageQuery = new PageQuery()
    pageQuery.setPageParams(count, rows, page);
    gpsymlQeuryVo.setPageQuery(pageQuery);

    List<GysypmlCustom> list = ypmlService.findGysymlList(gpsymlQeuryVo);
    DataGridResultInfo dinfo = new DataGridResultInfo();
    dinfo.setTotal(count);
    dinfo.setRows(list);
    return dinfo;
}
~~~~~~

## 数据范围权限 ##

企业控制范围权限
* 身份用户认证
* 用户权限控制(功能操作)
* 数据范围权限(通过 sql 硬编码, 必须要有用户id传入)
~~~~~~
在service层传入 用户id 和 queryVo, 然后再封装
~~~~~~

## 供货商药品目录添加查询 ##

查找供应商允许添加的药品目录

~~~~~~
<select id="findGysypmlAddList" parameterType="yycg.business.vo.GysypmlQueryVo"
        resultType="GysypmlCustom">
    select ypxx.*,
        (select info from dictinfo where dictcode=ypxx.jyzt and typecod='003') jyztmc
    from ypxx;
</select>
<select id="findGysypmlAddListCount">
</select>
~~~~~~


## 批量提交 ##

供货商选择药品添加药品到供货商药品目录(gysypml)

后置条件: 数据库操作, 在供货商药品目录表插入数据时, 同时向供货商药品目录控制表插入记录

Action 用 List 接收页面提交的数据, `String[] index`
接收被选择的序列

使用 hidden 命名规范 `gysypmls[index].ypxxid`

~~~~~~
public @Response SubmitResultInfo gysypmladdsubmit(
        GysymplQueryVo qgysypmlQueryVo, // 这里
        Sring[] index // 选择的行){
    List gyspmls = gysspmlQueryVo.getGysypmls();
    int num = indexs.length;
    for(int i=0;i<num;i++) {
        int index = Integer.parseInt(indexs[i]);
        gysypmls.get(index);
        ypmlService.insertGysypl(usergysid, ypxxid);
    }
}
// YmplService
// 根据供应商的id和药品信息id获取供应商目录表的一条记录
public Gysypml findGysypmlByYpxxAdnUsergysid(String usergysid, String ypxxid) {
    GysypmlExample gysypmlExample = new GysypmlExample();
    GysypmlExample.Createria c = gysypmlExample.createCriteria;
    c.andUsergysidEqualTo(usergysid);
    c.andYpxxidEqualTo(yapxixid);
    List list  = gysypmlMapper.selectByExample(gysypmlControlExample)
    return list.get(0);
}
public Gysypml findGysypmlControlByYpxxAdnUsergysid(String usergysid, String ypxxid) {
    GysypmlExample gysypmlExample = new GysypmlExample();
    GysypmlExample.Createria c = gysypmlExample.createCriteria;
    c.andUsergysidEqualTo(usergysid);
    c.andYpxxidEqualTo(yapxixid);
    List list  = gysypmlMapper.selectByExample(gysypmlControlExample);
    return list.get(0);
}
public void insertGysypml(String usergysid, String gyxxid) {
    Ypxx ypxx = ypxxMapper.selectByPrimaryKey(ypxxid);
    // 校验记录是否重复
    Gysypml gysypml = findGysypmlByYpxxAdnUsergysid(usergysid, ypxxid);
    if(gysypml!=null) {
        ResultUtil.throwException(createFail());
    }
    
    Gysypml gysympl_insert = new Gysypml();
    gysypml_insert.setId(UUDIBuild.getUUID());
    gysypmlMapper.insert(gysypml_insert);

   // 向供货商药品目录写一条表
   // 校验, 再添加
}
~~~~~~

## 供货商删除供货商药品目录 ##

提交内容, 药品信息id

删除供货商药品目录提交

## 监督单位对供货单位进行控制 ##

* 查询被控制对象(gysypml_control)
* 进行控制, 供货商药品目录控制提交

