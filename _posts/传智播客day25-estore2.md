title: 传智播客day25-estore2
date: 2014-05-08 08:52:58
tags:
- 传智播客
---

# 添加商品功能(图片上传) #

修改 Product_add.jsp


## 文件上传必要知识 ##

文件上传, HTTP 请求格式, 符合MIME协议(Content-Type, Content-Disposition)

客户端: 请求提交方式必须是POST, 编码格式 multipart/form-data,
上传文件输入框必须提供 name 属性

服务端: 采用 apache commons-fileupload (早期流行技术 jsp smartload),
内置文件上传技术 Part 技术

上传文件问题:
* 文件名乱码问题
* 上传文件重名问题
* 大文件上传, 空间占用

## 表现层 ##
~~~~~~
// ProductAddServlet
public void doGet(req, resp){
    // 请求是否为文件上传
    if(ServletFileUpload.isMultipartContent(req)){
        // 构造自定义请求参数 Map
        Map parameterMap = new HashMap<String, String>();
        DiskFileItemFactory fac = new DiskFileItemFactory();
        ServletFileUpload sfu = new ServletFileUpload(fac);
        List fileItems = sfu.parseRequest(req);
        for(f <- fileItems) {
            if(f.isFormField()){
                // 是普通元素
                parameterMap.put(f.getFieldName(), f.getString("UTF-8"));
            } else {
                // 是上传元素
                // 上传图片, 唯一文件名字, 目录分散
                InputStream in = new BufferInputStream(fileItem.getInputStream());
                // 将图片信息保存到服务器
                // 商品图片是可以直接访问的, 建立在 WebRoot 下
                String realName = fileItem.geName();
                //根据真实文件名, 生成唯一文件名 UUID.扩展名
                String filename = UploadUtils.generateRandomFileName(realName);
                String hashDir = UploadUtils.generateRandomDir(filename); // /E8/A9
                // 存放文件的绝对路径
                String absoluteDir = this.getServletContext().getRealPath("/upload") + hasDir;
                new File(absoluteDir).mkdirs();

                // 创建文件输出流
                OutputStream out = new FileOutputStream(absoluteDir + File.pathSaperate + filename);
                // apache commonio.jar 拷贝文件 FileUtils.copyFile();
                // CopyUtils 拷贝文件 apache工具包, 简化下面操作
                OutputStream out = new BufferedOutputStream(out);
                int b;
                while(b=in.read()!=-1){
                     out.write(b);
                }
                in.close();
                out.close();
                // 在浏览器显示图片
                parameterMap.put("imgurl", "/upload" + hashDir + "/" + filename);
            }
        }
        //将那个求取的的数据封装java对象
        Product product = new Product();
        BeanUtils.populate(product, parameterMap);
        productService.addProduct(product);
        // 跳转
    } else {
        throw new RuntimeException("添加上传, 必须为 multipart/form-data")
    }
}
~~~~~~
## 业务层 ##
~~~~~~
public interface ProductService{
    public void addProduct(Product product);
public class ProductServiceImpl implements ProductService{
    public void addProduct(Product product) {
        productDao.insert(product);
    }
}
~~~~~~

## 持久层 ##
~~~~~~
public class ProductDaoImpl implements ProductDao{
    public void insert(Product product){
        String sql = "insert into product values(...);"
        queryRunner.update(conn, sql, product.getName, ...);
    }
}
~~~~~~

# 查看商品 #
查询所有商品功能, 先访问Servlet, 查询所有商品信息, 保存request对象, 转发, 用 jstl 显示

**把页面中的所有相对路径 都换成绝对路径**,
如果 从 1.jsp 转发到 2.jsp, 但是url是 1.jsp, 会产生路径问题

~~~~~~
// ProductSearchServlet
public void doGet(req, resp) {
    // 调用业务层, 查询商品集合列表
    List products = products.listAllProducts()
    req.setAttribute("products", products)
    // 将列表通过request对象, 传递给jsp对象
    req.getRequestDispatcher("/product.jsp").forward(req,resp);
}

// ProductService
public List listAllProducts(){
    return productDao.listAllProducts(); 
}

// ProductDao
public List listAllProducts(){
    String sql = "select * from Product";
    return queryRunner.query(conn, sql, new BeanListHandler());
}
~~~~~~

## 显示结果数据 ##
~~~~~~
<c:forEach var="p" items="${requestScope.products}">
    
</c:forEach>
~~~~~~

展示列表的图片要使用缩略图
* 使用 img 标签 width 和 height 缩放图片(但是不建议, 浪费带宽)
* 在上传图片过程中, 对图片缩放(老师工具包PicUtils)

# 购物车功能 #

## 添加商品到购物车 ##
在商品列表中点击购买, 将商品加入到购物车, 提示用户商品已经添加到购物车,
继续购物还是查看购物车. (修改product.jsp)

~~~~~~
// 购买链接
<a onclick="buy(${product.id}, ${procuct.pnum})">购买</a>
<script type="text/javascript">
function buy(id, pnum){
    //判断商品是否有货
    if(pnum==0) {
        // 无货
        alert("您购买的商品, 已经售罄");
        return;
    }
    // 递交请求给服务器, 加入购物车
    window.loacation.href = "${pageContext.request.contextPath}/addCart?id=" + id;
}
</script>
~~~~~~

~~~~~~
// AddCartServlet
public void doGet(req, resp){
    /// 对 product 重写 hashcode 和 equals, 根据id生成
    Map<Product, Integer> cart = req.getSession().getAttribute("cart");
    if(cart==null) cart = new HashMap();
    // 判断是否存在在购物车中
    int id = Integer.parseInt(req.getParameter("id"));
    Product product = new Product();
    product.setId(id);
    if(cart.contains(product)) {
        cart.put(product, cart.get(product) + 1);
    } else {
        // 商品没有在购物中, 放入新的商品, 数量为1
        productService.findProduct(id);
        cart.put(p, 1);
        // 调用业务层, 查询完整的商品信息
    }
    req.getSession().setAttribute("cart", cart);
    out.print("商品已经加入购物车, 查看购物车, 继续购物");
}

// ProductService
public Product findProduct(int id) {
    return productDao.findById(id);
}

// ProductDao
public Product findById(int id) {
    String sql = "select * from product where id=?";
    return queryRunner.query(conn,sql, new Beanhandler<>(), id)
}
~~~~~~
## 查看购物车 ##

查看购物车, 只要修改 jsp 页面

~~~~~~
<!-- 计算节省和公花费 -->
<!-- 定义变量 -->
<c:set var="saveprice" scope="page" value="0"> </c:set>
<c:set var="totalprice" scope="page" value="0"> </c:set>
<!-- 计算价格 -->
<set var="saveprice" scope="page" value="${saveprice+**}"> </set>
<set var="totalprice" scope="page" value="${totalprice+**}"></set>
~~~~~~
