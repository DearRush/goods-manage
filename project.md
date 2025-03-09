- **用户管理模块** ：用户登录、注册、密码找回（通过邮箱方式）、用户信息修改、密码修改
- **商品管理模块** ：商品增删改查、商品图片导入（存储在MongoDB）、导出商品报表、商品分类增删改查、库存查改，库存不足和积货提醒、商品回收和恢复。
- **订单管理模块** ：订单查询查看、订单退款管理（查看和审批）、发货管理、物流公司管理

## 1。数据库模型

mysql,InnoDB

order:订单（商品ID，用户ID，总价，是否付款等状态）

Item:商品

ReItem:商品回收表

User:用户

order_shipping:收件人寄件人信息表

order_delivery:快递单号、快递公司

Delivery:订单的快递信息

ItemCategory:商品的多级标签

## 2.Mybatis的使用

+ 安装依赖
+ 数据库创建表
+ 在`src/main/resources`目录下创建`mybatis-config.xml`文件，配置MyBatis的基本设置：
+ 创建`User`和`Item`实体类，用于映射数据库表
+ 创建`UserMapper`和`ItemMapper`接口，定义数据库操作方法
+ 在`src/main/resources`目录下创建`UserMapper.xml`和`ItemMapper.xml`文件，定义SQL语句。

## 3.Model对象分页的逻辑

设置了一个基类：里面有分页数、页大小、排序方法等信息；

所有关键数据Model都继承这个类；数据库中有记录

构建网址的时候会根据数据的分页数来构建

## 4.用户管理

用户登录POST：输入被转换为user类，findByIDandPassword；

用户注册POST：输入被转换为user类，findByName，不存在，创建

用户信息修改POST：输入被转换为user类，httpsession保存的USER，findByname，httpsession更新

忘记密码POST：输入是邮箱,findbyemailandname,用**springframework**.**mail**.**SimpleMailMessage**;构建邮箱内容（用户密码）

## 5.商品管理

商品根据状态（数据库对象是否删除的识别符）、价格、名字：用户输入构建一个Item查询条件，查找内容，得到数目，就能够得到分页的页数，同时找到所有的标签（用于前端）

商品图片的处理（商品信息修改）：图片ID和记录ID一致，MongoDB（用户输入图片由一个`MultipartFile`解析）和商品其他信息，根据其他信息findByID/name找到对象，对象存储了复制图片的路径，UPDATE，对MongoDB的作存储处理

查看符合条件的已下架商品：和商品查询逻辑区别不大，主要是查ReItem库。

商品下架：下架的商品有一个单独的数据库表ReItem,找到商品，从Item库删除，在ReItem中插入；

商品回复：找到商品，从ReItem库删除，在Item中插入；

库存查看：和商品查看没有区别（只是SQL语句不一样）

商品报表：用户输入构建一个Item查询条件，得到ItemList。HTTPresponse返回一个可写的outputstream,基于jxl**.**Workbook实现;主要流程计算表数量，创建表头，针对每一个Item变量其feature，写入Excel。（默认到浏览器）

商品分类标签：商品对象本身有cid和category_name，可以查找、修改、插入（首先GET对象，然后POST）（修改、插入：需要前端的选择来决定父标签）分类标签，此外商品插入会从一个category_list里面去选择，保证外键约束一致



## 6.订单管理

查询订单:和商品查询逻辑区别不大，唯一的问题在于Order数据对象没有对应Item，这个关系是存在OrderItem库中的，同时需要查询OrderItem库补全信息。

订单报表：和商品报表差别不大

申请退款订单查询：查找申请退款的（符合条件的）订单（同样需要查找OrderItem表格）

审批退款：找到该订单，前端修改，提交POST，更改RefundStatus的字段

用快递单号实现位置查询：首先程序会得到快递ID，去数据库查找得到快递单号，构建一个url query前往kuaidi100.com网站，得到**HttpRequest**.**readData**输出，json解包得到位置，返回

## 7.Redis

redis的配置：首先写配置文件，写@**ConfigurationProperties**(**prefix** = "redis") **RedisConfig**类（属性：host,port,pool_number......getter和setter，自动从springboot配置文件读取）；随后实现Bean:**JedisPoolFactory**：首先获取Config的Bean，初始化一个JedisPool

redis的使用：首先jedisPool.**getResource**();jedis.**get**(realKey);**returnToPool**(jedis);

主要用于统计量（基本数字）的获取：对于当月收入、当月订单量......每一次访问统计数据时，首先查缓存，随后查数据库，缓存失效时间：1天......



