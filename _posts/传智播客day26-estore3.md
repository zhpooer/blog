title: 传智播客day26-estore3
date: 2014-05-09 09:05:29
tags:
- 传智播客
---

# 修改购物车 #

解决页面 中数字显示, 保留位数问题

服务器Java代码解决方案`java.text.NumberFormat`

页面中解决方案: jstl 提供的 fmt 标签库 (用于国际化显示, 和格式化显示)
~~~~~~
<%@ taglib uri="....jstl/fmt" frefix="c" %>
<fmt:formatNumber value="${0.111111}" maxFractionDigits="2"
                  minFractionDigits="2" />
~~~~~~


## 修改购物车商品数量 ##
修改数量后, 通过 js 的 blur 事件, 提交请求给服务器,
服务器根据请求修改购物车中的商品数量
1. 为页面添加 onblur 事件, 函数接受商品编号和修改后的数量
2. 判断修改后的数量, 必须为正整数(不能是字母, 负数, 小数, 允许为0)
3. 提交请求给服务器, 根据商品编号, 修改 Session 的购物车对象商品对应的数量.
如果修改后的数量为0, 删除购物商品
4. 修改数量后, 回到购物车页面

~~~~~~
<input type="text" value="${entry.value}"
       onblur="changeBuyNum(${entry.key.id}, ${entry.key.pnum}, this.value);"/>

<script type="text/javascript">
function changeBuyNum(id, maxnum, num){
    // 对修改后的合法性, 进行校验
    var regex = /^[0-9]+$/
    if(regex.test(num)) {
        if(num>maxnum) {
            // 存货不足
            alert("商品库存数量不能草果库存数量")
        } else {
            window.locatioin.href="${pageContext.request.contextPath}/" + 
            +"upldateCartItem?id=" + id + "&num="  + num
        }
    } else {
        alert("请输入正确格式")
    }
}
</script>
~~~~~~

~~~~~~
// UpdateCartItemServlet
public void doGet(req, resp){
    int id = Integer.parseInt(req.getParameter("id"));
    int num = Integer.parseInt(req.getParameter("num"));
    
    // 从session获取购物车对象
    Map cart = req.getSession.getAttribute("cart");
    
    // 服务器校验商品数量(根据id查询商品数量)
    ProductService productService = new ProductService();
    int num = productService.findProductNumById(id);
    if(num>pnum) {
        throw new RuntimeException("库存不足");
    }

    // 修改数量
    Product product = new Product();
    product.setId(id);
    if(num==0){
        cart.remove(product);
    } else {
        cart.put(product, num); // key不会覆盖, value覆盖
    }
    resp.sendRedirect("/cart.jsp");
}

// ProductService
public int findProductNumById(int id){
    return productDao.findProductNumById(id);
}

// ProductDao
public int findProductNumById(int id){
    String sql = "select pnum from project where id = ?";
    return queryRunner.query(conn, sql, new ScalarHandler("pnum"), id);
}

~~~~~~

## 商品删除 ##
1. 在页面中显示购物车信息时, 每个商品后, 提供删除按钮
2. 点击删除按钮, 将不需要购买商品编号, 发送到服务器
3. 根据商品编号, 从Session的cart对象, 移除商品
~~~~~~
<a href="deleteCartItem?id=${entry.key.id}"></a>
~~~~~~

~~~~~~
// DelCartItemServlet
public void doGet(){
    int id = req.getParameter("id");
    Map cart = req.getSession().getAttribute("cart");

    Product product = new Product();
    product.setId(id);

    // 因为重写的hashCode和equals
    cart.remove(product);
    // 跳转回购物车页面
    resp.sendRedirect("/cart.jsp");
}
~~~~~~

# 订单生成 #

在购物车选好后, 点击结算按钮, 跳转到订单生成页面

1. 在生成订单页面中, 生成真实的页面信息, 参考 cart.jsp 实现
2. 产生订单

分析数据变化:
* Order订单表: 存放订单整体信息(总金额, 创建时间, 支付状态, 收货人信息)
* OrderItem 订单项目表, 存放每个商品购买几件

生成订单页面, 需要提交收货人信息 和 订单总金额 到服务器  
订单总金额用`<input type='hidden' value="totalprice" name="totalprice"/>`来存放

## 服务器代码 ##
~~~~~~
// OrderAddServlet
public void doGet(){
    // 将请求数据封装到model对象中
    Order order = new Order();
    BeanUtils.populate(order, req.getParameter());

    // 生成订单号
    DateFormat dateFormat = new SimpleDateFormat("yyyyMMdd");
    String id = "estore_" + dateFormat.format(new Date()) + "_" + UUID;
    order.setId(id);

    order.sestPaystate(0);
    order.setCreateTime(new TimeStamp(System.currentTimeMillis()));

    User user = req.getSession().getAttribute("user");
    order.setUser_id(user.getId());

    List<OrderItem> orderItems = new ArrayList<OrderItem>();
    // 封转订单项项目
    Map cart = req.getSession.getAttribute("cart");
    for (entry <- cart.entrySet()) {
        OrderItem orderItem = new OrderItem();
        orderItem.setOrder_id(orderid);
        orderItem.setProduct_id(entry.getKey().getId());
        orderItem.setBuynum(entry.getValue);
        orderItems.add(orderItem);
    }
    orderService.addOrder(order, orderItems);

    req.getSession().removeAttribue("cart");
}

// OrderServiceImpl
public class OrderServiceImpl implements OrderService{
    public void addOrder(Order order , List<OrderItem> orderItems){
        orderDao.insert(order, orderItems);
    }
}

public class OrderDaoImpl implements OrderDao {
    public void insert(Order order , List<OrderItem> orderItems){
        String orderSql = "insert into orders values(?...)";
        queryRunner.update(conn, orderSql, order.getId);
        // 保存订单项数据
        String orderitemSql = "insert into orderitem values(?,?,?)"
        List<Order []> argList = new ArrayList<Order[]>();
        for(OrderItem orderItem : orderItems){
            // 用 batch, 先将参数保存到二维数组
            Object[] orderitemArgs = {orderItem.getOrder_id(), orderItem.getProduct_id(), any}
            argList.add(orderitemArgs);
        }
        // 将多种参数, 一次提交给服务器, 提高效率
        QueryRunner.update(conn, orderitemSql, argList.toArray(new Object[0][]));
    }
}
~~~~~~
tips: `Connection.setSavePoint()` 以及 `Connection.commit()` 要放在 错误处理里

### tips: 回滚点 ###

回滚点, 可以在事务执行过程中进行记录, 从而在回滚事物的时候, 不会滚到事务最开始,
而回滚到指定保存点.

主流场景: 大规模批量插入数据.
~~~~~~
Connection conn = null;
Savepoint savepoint = null;
try{
    for(int i=1; i<1000000; i++) {
        // 每1000条, 发送一次
        String sql = "";
        Statement statement.addBatch(sql);
        if(i%1000==0){
        // 1000条发送一次, 清除缓存
            statement.excuteBatch();
            statement.clearBatch();
            savepoint = connn.setSavePoint();
        }
        // 如果插入1000条没有问题保存一个回滚点
    }
} catch {
    conn.rollback(savepoint);
} finally {
    conn.commit();
}


~~~~~~

# 订单查询 #

点击订单查看功能按钮, 在服务器端将订单信息输出, 显示页面
* 管理员显示所有人的订单
* 用户显示自己的订单

~~~~~~
// OrderSearchServlet
public void doGet(req, resp) {
    // 从Session中获取用户登陆信息
    User user = req.getSession().getAttribute("user");
    List<Order> orders = orderService.showOrders(user);
    req.setAttribute("orders", orders);
    req.getRequestDispatcher("orders.jsp").forward(req, resp);
}
// OrderService
public List<Order> showOrders(User user) {
    if(user.getRole().equals("normal")){
        // 普通用户
        return orderDao.findOrdersByUser(user.getId());
    } else if(user.getRole().equals("admin")){
        // 管理员
        return orderDao.findAllOrders();
    }
}
// OrderDao
public List<Order> findAllOrders(){
    String sql = "select * from order"
    // 其余代码 参照 findOrdersByUser
}
public List<Order> findOrdersByUser(int id){
    String sql = "select * from order where user_id = ?"
    List<Order> orders = queryRunner.query(conn, sql, new BeanListHandler<>(), id);

    // 根据用户id, 查询用户信息
    sql = "select * from user where id=?";
    for(order <- orders){
        String nickname = queryRunner.query(sql, new ScalarHandler("nickname"), order.getUser_id());
        order.setNickname(nickname);
        // 订单关联订单项, 查询订单项
        String sql2 = "select * from orderitem where order_id=?";
        List<OrderItem> orderItems = queryRunner.query(conn, sql2, new BeanListHandler<>(), order.getId);
        order.setOrderItems(orderItems);
        // 查询每个订单项 关联商品信息
        for(OrderItem orderItem : orderItems) {
            String sql4 = "select * from product where id=?";
            Product product = queryRunner.query(conn, sql4, new BeanHandler(), orderItem.getProduct_id);
            orderItem.setProduct(product);
        }
    }
    return orders;
}
~~~~~~

# 在线支付 #

* 支付方案一: 网站直接与银行对接
  * 服务器根据银行的接入规范, 生成银行需要的数据, 然后通知用户浏览器重定向到银行,
并把这些数据发给银行, 以完成支付
  * 缺点, 网站需要针对不同银行开发不同的支付程序, 且银行规范一旦发生变动, 网站也要跟着改
* 支付方案二: 网站通过第三方支付公司与银行对接
  * 用户登陆商城, 选择银行, 只需要根据根据第三方公司的接入规范,
将参数发给第三方支付公司, 再由第三方支付公司, 根据不同银行接入规范, 对接银行
  * 缺点, 网站与第三方支付公司定期资金结算, 资金安全是个大问题,
也收取一定的手续费, 这种支付方案只适合金额在百万以下的公司

## 易宝支付 ##
支付流程
1. 商城在易宝网站 商家注册(审核), 易宝提供给商家: 商家编号, 密钥
2. 用户支付流程
  * 用户选择银行, 获取银行编码
  * 商城根据订单号 金额 银行, 根据第三方公司接入规范,
  将参数发送给第三方支付公司(302重定向)
  * 第三方公司转到网银支付页面, 支付
  * 支付完成, 网银重定向到支付公司页面, 支付公司通知商城, 并调用商城的返回页面

密钥解决数字签名
  * 商城将订单号 金额 callback 商家编号, 发送给支付公司,
  支付公司为了验证消息来自商城, 商城将发送数据, 使用商家提供密码,
  进行加密获取数字签名；支付公司将发送来的数据, 采用通用密钥也加密一次,
  对比两个签名
  * 支付公司在通知商城支付成功, 也要使用数字签名

## 编程实现 ##
1. 需要商家编号和加密密钥
2. 在支付的第一步, 选择银行, 发送支付银行id
  * 修改Order.jsp, 将订单号和金额发送给选择银行的界面
  * 选择好银行, 转到商城 Servlet, 根据第三方接入规范, 加密数据

~~~~~~
// PayServlet
public void doGet(req, resp) {
    String orderId = req.getParameter("orderid");
    String money = req.getParameter("mondey");
    String pd_FrpId = req.getParameter("pd_FrpId");

    // 根据第三方接入, 准备参数
    String p0_Cmd = "Buy";
    String p1_MerId = "1234566";

    // 将所有数据进行数字签名
    String key = ""; // 密钥
    String hmac = PaymentUtil.buildHmac(p0_cmd, p1_MerId);

    // 将所有的数据发送给易宝指定页面
    // 或跳转到 确认支付页面
    req.getRequestDispatcher("").forward(req, resp);
}
~~~~~~


当用户付款后, 第三方支付方重定向(第一次)商城支付结果页面, 提供给用户付款已经
成功提示,(并没有真正收到钱), 当银行通过程序通知第三方支付, 钱已经支付了,
第三方支付通知商城(第二次), 钱到了, 商城修改订单状态

~~~~~~
// CallBackServlet 付款后回调程序
public void doGet(){
    // 校验, 易宝数字签名
    String p0 = req.getParameter("p0");
    String hmac = req.getParameter("");

    // 将响应数据加密, 校验 hmac
    boolean isValid = PaymentUtil.verifyCallback(hmac, ...);
    if(isValid) {
        // 区分两次访问类型
        if(r9_BType == 1){
            //银行付款后的友好页面
        } else if(r9_BType == 1) {
            // 银行已经真正收到钱, 修改订单状态
        }
    } else {
        throw new RuntimeException("数字签名错误");
    }
}
~~~~~~
