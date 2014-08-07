title: 传智播客day63-医药集中采购平台 采购单
date: 2014-07-28 10:41:29
tags:
- 传智播客
---

# 业务分析 #
供货商将药品目录整理完成, 供货商供货供货区域内的医院,
医院向所在区域供应药品的供货商创建采购单

1. 创建采购单(基本信息)
2. 设置采购药品明细, 添加采购药品, 设置采购量
3. 审核采购单, 由卫生室所在乡/镇卫生院审核提交的采购单
  * 审核通过, 供货商受理
  * 审核不通过, 退回重新修改
5. 供应商受理, 发货
4. 采购入库

# 采购单 #

医院采购单(yycgd)
* id
* 采购单状态, (zt, 未提交, 已提交未审核, 审核通过, 不通过)
* 编号
* 医院id

医院采购单明细(cgdmx), 存放采购单采购药品, 唯一约束(yygdid, ypxxid)
* 采购单id
* 药品信息id, ypxxid
* 供货企业, ghqy
* 中标价
* 交易价
* 采购量


动态分表: 
随着系统使用, 时间长了采购明细信息表中的记录非常多,
如果进行分析速度很慢. 按年分开存储, mysql分库存储, oracle分表存储.
根据业务需求, 按年, 月, 日分. 如 `yycgdmx2014` `yycgdmm2014`

`bm=4位年+6位流水号` 使用序列解决流水号问题, 由于采购单主表和明细表年分开,
序列也按年分开.


~~~~~~
<select id="getYycgdBm" parameterType="string" resultType="string">
  -- 采购单编号 sequence 生成
  select '$value'||yycgdbm${value}.nextval from dual
</select>
~~~~~~

用户参数转换器

~~~~~~
<!-- 注解驱动 -->
<mvc:annotation-driven >
    <mvc:argument-resolvers>
        <bean class="yycg.base.action.filter.UserArgumentResolver"/>
    </mvc:argument-resolvers>
</mvc:annotation-driven>
~~~~~~

~~~~~~
public class UserArgumentResolver implements WebArgumentResolver {
    public Object resolveArgument(MethodParameter methodParameter, NativeWebRequest webRequest) throws Exception {
        //如果conroller方法的参数类型是ActiveUser，则执行
        if (methodParameter.getParameterType().equals(ActiveUser.class)) {
            ActiveUser user =(ActiveUser)webRequest.getAttribute(Config.ACTIVEUSER_KEY, RequestAttributes.SCOPE_SESSION);
            //return一个对象就是给controller参数赋值
            return user;
        }
        return UNRESOLVED;
    }
}
~~~~~~

`@SessionAttributes` 该注解用来绑定HttpSession中的attribute对象的值，便于在方法中的参数里使用。
该注解有value、types两个属性，可以通过名字和类型指定要使用的attribute 对象
~~~~~~
// 方法中通过形参获取activeUser，如下：
// 方法名(ActiveUser activeUser)
@SessionAttributes(value="activeUser")
~~~~~~
