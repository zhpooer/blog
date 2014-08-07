title: 传智播客day62-医药集中采购平台 用户管理
date: 2014-07-22 14:22:07
tags:
- 传智播客
---

# 模型 #

## 单位信息 ##

系统中所有业务关联都是关联的单位信息表的id

卫生局管理卫生院, 卫生院管理医院, 行政管理

医院信息表(useryy)
* id
* DQ, 地区, 村

监督单位(userjd)
* DQ, 市或乡

监督供应商(usergys)

供货商供货区域
* 供货商id
* 地区

bss_sys_area(区域表)
* areaid, 0:根, 1:某某市, 1.1:某某市某某区
* areaname
* arealevel, 行政级别

~~~~~~
-- 供货商下面区域的卫生室 TODO
-- camtasia recorder

~~~~~~

## 用户信息 ##

sysuser(系统用户表)
* groupid, (用户类别: 系统管理员, 卫生局, 卫生院, 卫生室, 供货商)
* sysid(单位id)
* id, 关联信息
* userid,账号
* username
* userstate, 用户状态(正常, 暂停)


# 系统用户列表 #

~~~~~~
@RequestMapping("/userquery")
public String userquery(Model model) {
    return "userquery.jsp";
}

@RequestMapping("/userquery_result")
// form 表单中的数据可以被序列化成 json 数据
public @ResponseBody DataGridResultInfo userquery_result(
  SysuserQueryVo sysuserQueryvo,
  int page,
  int rows
) {
    if(sysuserQueryvo = null) sysuserQueryvo = new SysuserQueryvo();
    /// 查询列表的数量
    int count = userManagerService.findSysuserCount(sysuserQueryvo);
    // 穿件分页参数
    PageQuery pageQuery = new PageQuery();
    pageQuery.setPageParams(count, rows, page);
    
    sysuserQeury.setPageQuery(pageQeury);
    // 根据page 和rows查询数据列表
    List list = userManagerList.findSysuserList(sysuserQueryVo);
    DataGridResultInfo dataResult = new DataGridResultInfo();
    data.setTotal(count);
    data.setRows(list);
    return data;
}

public class SysuserCustom extends Sysuser {}

public class SysuserQueryVo {
    private SysuserCustom sysuserCustom;
    private PageQuery pageQeury; // 封装 开始坐标, 结束坐标
}

public interface SysuserMapperCustom {
// 用户查询列表
// 用户查询总数
}
public interface UserManagerService {
// 用户查询列表
// 用户查询总数
}
~~~~~~

~~~~~~
<mapper>
    <select id="findUserList" parameterType="vo.SysuserQueryVo"
       resultType="voSysuserCustom">
          select sysuser.id, sysuser.userid, sysuser.username, sysuer.groupid,
              nvl(sysuser,useryymc, nvl(sysuser.gysmc, sysuser.userjdmc)) sysmc from (
                select stsuser.*, useryy.mc, userjd.mc, usergys.mc from sysuser
                  left join useryy on sysuser.sysid=useryy.id
                  left join userjd on sysuser.sysid=userjd.id
                  left join usergys on sysuser.sysid=usergys.id
              ) sysuser
          <where>
          <if test="sysuserCustom!=null">
              <if test="sysuserCustome.groupid!=null and sysuserCustom.groupid!=''">
                  and groupid=#{sysuserCustome.groupid}
              </if>
              <if test="sysuserCustome.userid!=null and sysuserCustom.userid!=''">
                  and userid=#{sysuserCustome.userid}
              </if>
              <if test="sysuserCustome.username!=null and sysuserCustom.username!=''">
                  and username like '%${sysuserCustome.username}%'
              </if>
              <if test="sysuserCustome.sysmc!=null and sysuserCustom.sysmc!=''">
                  and sysmc like'%${sysuserCustome.sysmc}%'
              </if> 
          </if>
          </where>
            -- 加入分页 写法 思路
            -- pageQuery.start
            -- pageQuery.end
            select * from (
            select rownum page_rownum, page_0.*
            from (
              select * from user0
            ) page_0
              where rownum<=100
            ) page_1
            where page_1.page_rownum > 90;
        </select>
        
    <select id="findUserCount" parameterType="vo.SysuserQueryVo"
      resultType="int">
        select count(*) from (
            select stsuser.*, useryy.mc, userjd.mc, usergys.mc from sysuser
              left join useryy on sysuser.sysid=useryy.id
              left join userjd on sysuser.sysid=userjd.id
              left join usergys on sysuser.sysid=usergys.id
            ) sysuser
        <where>
            <include refid="query_sysuser_where"> </include>
        </where>
    </select>
</mapper>
~~~~~~

## 添加页面 ##

使用 jqueryEasyUI, modal 嵌套 iframe 来生成添加页面, `parent.closewindow()`


~~~~~~
function sysusersave(){
    if($.formValidator.pageIsValidate()) {
        var formObj = $("#formId");
        var options = {
            dataType: "json"
            ...
        }
        // jquery form 提交
        formObj.ajaxSubmit(options);
    }
}
~~~~~~

~~~~~~
// userAction

// 用户添加页面
@RequestMapping("/useradd")
public String useradd(Model model) {
    return "/base/user/useradd";
}


private Sysuser sysuser;  // 用于添加/修改用户信息提交

// 用户添加提交
@RequestMapping("/useradd")
public @ResponseBody ResultInfo useraddsubmit(Model model, SysuserQueryvo sysuserQeuryVo) {
    sysuserQeury = sysuserQeuryVo != null ? sysuserQeuryVo : new SysuserQueryvo;
    ResultInfo resultInfo = null;
    // 调用service 添加用户
    // 捕获异常添加信息
    try {
       userManagerService.insertsysuser()
    } catch (ResultInfoException e) {
       resutlInfo = e.getResultInfo();
    } catch (Exception e){
        resultInfo = new ResultInfo()
        resultInfo.setType(ResultInfo.TYPE_RESULT_FAIL);
        resultInfo.setMessage("未知错误");
    }
    if(resultInfo == null ) {
    }
    return resultInfo;
}

// userManagerService
// 添加用户
public void insertsysUser(Sysuser sysuser){
   // 校验用户账号唯一
   // 根据用户账号查询数据库sysuser, 如果查出一条表示重复
   Sysuser sysuser_l = findSysuserByUserid(sysuser.getUserId());
   if( sysuser_l != null) { // 表示账号重复
       ResultInfo info = new ResultInfo;
       info.setType(ResultInfo.Failed);
       info.setMessage("账号重复");
       // 提示用户账号存在
       throw new ResultInfoException(info);
   }
   
   // 调用dao插入用户信息
   // 根据单位名称 得到单位id
   // 根据页面提交的groupid, 如果groupid等于1或2, 单位名称在 userjd 表存在
   // TODO 11.35
   if(groupid == "1" || groupid == "2") {
       Userjd userjd = findUserjdByMc(mc);
       sysid == userjd.getId();
   } else if(groupid=="3") { // 医院
   } else if(groupid=n="4") { // 供货商
   }
   sysuser.setId(UUIDBuild.getUUID());
   sysuser.setSysid(sysid);
   sysuserDao.insert(sysuser);
}

public Sysuser findSysuserByUserid(String userid) {
   SysuserExample sysuserExample = new SysuserExample();
   SysuserExample.Criteria criteria = sysuperExample.createCriteria();
   criteria.addUseridEqualTo(sysuser.getUserid);
   List list = sysuserMapper.selectByExample(sysuserExample);
   if(list!=null && list.size()==1) {
       return list.get(0);
   }
   return null;
}
~~~~~~
自定义异常类
~~~~~~
public class ExceptionResultInfo extends Exception {
    // 习题同一使用的结果类
    private ResultInfo resultInfo;
    public  ExceptionResultInfo(Info resultInfo){
    }
}
public class ResultInfo {
  private int type;
  private String message;
}

~~~~~~

## 异常处理器 ##

`yycg.base.process`, 存储信息处理类

全局异常处理器:
如果返回的类型是 json 数据, 异常以json返回.
如果返回的类型是 页面, 返回错误页面

~~~~~~
<!-- Http messageConverters，用于将对象转换为json输出到客户端 -->
<bean id="jsonmessageConverter"
    class="org.springframework.http.converter.json.MappingJacksonHttpMessageConverter">
</bean>
<bean id="handlerExceptionResolver" class="ExceptionResolverCustom">
    <property name="jsonmessageConverter" ref="jsonmessageConverter"></property>
</bean>

<!-- web.xml -->
<!-- -->
<servlet>
    <!-- 屏蔽自动注册异常处理器，固定使用bean的id为handlerExceptionResolver的异常处理器 -->
    <init-param>
        <param-name> detectAllHandlerExceptionResolvers</param-name>
        <param-value> false </param-value>
    </init-param>
</servlet>

~~~~~~

系统全局异常处理器
~~~~~~
public class ExceptionResolverCustom implements HandlerExceptionResolver {
    // json 转换
    private HttpMessageConverter<ExceptionResultInfo> jsonmessageConverter;
    
    public ModelAndView resolverException(request, response, handler, ex){
        // 找action方法是否是标注了根据注解responseBody
        // 获取action的方法
        HandlerMethod handlerMethod = (HandlerMethod) handler;
        Method method = handlerMethod.getMethod();
        ResponseBody responseBodyAnn = AnnotationUtils.findAnnotaion(method, ResponseBody.class);
        if(responseBodyAnn!=null) { // 说明方法返回的responseBody注解
            // 处理返回类型是json的异常信息
            //创建http输出对象
            HttpOutputMessage outputMessage = new ServletServerHttpResponse(response);
            //将异常信息输出json
            jsonmessageConverter.write(value, MediaType.APPLICATION_JSON,outputMessage);
            return new ModelAndView();
        }
        // handler excpeption to get resultInfo
        // 返回的是页面
        ModelAndView modelAndView = new ModelAndView();
        modelAndView.setViewName("/base/error");
        modelAdnView.addObject("exceptionresultInfo", resultInfo);
        return modelAdnView;
    }
}
~~~~~~

~~~~~~
public class SubmitResultInfo {
    private ResultInfo resultInfo;
}
~~~~~~

# 国际化 #

Internationalization: I18N.

`java.util.ResourceBundle`: 用于加载一个资源

`java.util.Local`: 对应一个特定的国家/区域, 语言环境

`java.text.MessageFormat`: 用于将消息的格式化

资源文件命名格式

    message_zh_CN.properties 中文, 中过
    message_en_US.properties 英语, 美国
    message.properties

~~~~~~
Locale locale = Locale.getDefault();

// 先在 message_zh_CN找, 再到 message.proerties
ResourceBundle rb = ResourceBundle.getResource("message.properties", locale);
String retValue = rb.getString(102);

// 102=恭喜你登陆成功{1}
MessageFormat.format(retValue, new Object[]{"张三"})
~~~~~~

~~~~~~
// 208=用户名已经存在
ResultUtils.throwException(ResultUtils.createFail(Config.Message, 208, null));

ResultUtils.createSubmitResult(ResultUtils.createSuccess(Config.Message, 906, null));
// 客户端封装函数 message_alert(value); value = {resultInfo...}
~~~~~~

# 删除 #

~~~~~~
// userManagerService
public void deleteSysuser(String id){
    // 校验用户是否存在
    Sysuser sysuser = sysuserMapper.selectByPrimaryKey(id);
    if(sysuser == null) {
        ResultUtils.throwException(ResultUtil.createWarning(config.Message, , null));
    }
    sysuserMapper.deleteByPrimarKey(id);
}

// action
@RequestMapping("/userdelete")
public @ResponseBody SubmitResultInfo userdel(String deleteid){
    sysuserManager.deleteSysuser(deleteid);
    return ResultUtil.createSubmitResult(ResultUtils.createSuccess(cofnig.message, , null));
}
~~~~~~

# 修改用户 #
~~~~~~
// action 参考 添加用户, 要查询用户和单位名称 SysuserCustom
// BeanUtils.copyProperties(sysuser, sysuserCustomer);d

// service
public void updateSysuser(Sysuser sysuser) {
    // 校验用户
    // 如果根据主键查询出来一条记录, 如果这条记录的账号是自己的表示用户没有修改账号(userid
    // 如果查询出来的账号是别人, 抛出占用别人账号
    
    // 先查询用户信息
    Sysuser sysuser_update = this.findSysuserById(sysuser.getId());
    sysuser_update.setUsername();
    sysuser_update.setUserstate();
    // ...userid username userstate sysid
    // 如果页面传过来密码信息表示用户要修改密码
    // 更新用户信息
    // 如果用户输入的单位id不存在, 则抛出提示信息
    return sysuserMapper.updateByPrimaryKey(sysuser);
}
// 用户信息显示
public Sysuser findSysuserById(String id) {
    // 根据页面提交的账号查询用户信息

    return userMapper.selectByPrimaryKey(id);
}
// 医院信息显示
public void findUseryyBId(String id) {
    return useryyMapper.selectByPrimaryKey(id);
}
// 供货商信息显示
public void findUsergysBId(String id) {
    return usergysMapper.selectByPrimaryKey(id);
}
// 监督单位信息显示
public void findUserjdBId(String id) {
    return userjdMapper.selectByPrimaryKey(id);
}

~~~~~~

