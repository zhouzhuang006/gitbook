# 设计模式

不用设计模式并非不可以，但是用好设计模式能帮助我们更好地解决实际问题，设计模式最重要的是解耦。设计模式天天都在用，但自己却无感知。我们把设计模式作为一个专题，主要是学习设计模式是如何总结经验的，把经验为自己所用。学设计模式也是锻炼将业务需求转换技术实现的一种非常有效的方式。

### 回顾软件设计原则

七大原则

| 设计原则     | 解释                                                         |
| :----------- | :----------------------------------------------------------- |
| 开闭原则     | 对扩展开放，对修改关闭。                                     |
| 依赖倒置原则 | 通过抽象使个各类或者模块不互相影响，实现松耦合。             |
| 单一职责原则 | 一个类、接口、方法只做一件事                                 |
| 接口隔离原则 | 尽量保证接口的纯洁性，客户端不应该依赖不需要的接口。         |
| 迪米特法则   | 又叫知道最少原则，一个类对其所依赖的类知道得越少越好。       |
| 里氏替换原则 | 子类可以扩展父类的功能但不能改变父类原有的功能。             |
| 合成复用原则 | 尽量使用对象组合、聚合。而不使用继承关系达到代码复用的目的。 |

先来看个生活案例，当我们开心之时，总会寻求发泄的方式。在学设计模式之前，你可能会这样感叹：

`喝酒唱歌，人生真爽。`

学完设计模式之后，你可能会这样感叹：

`对酒当歌，人生几何。`

大家对比一下前后的区别，有何感受？

回到代码中，我们来思考一下，设计模式能够帮我们解决哪些问题？

**写出优雅的代码**

先来看一段很多年以前写的代码：

```java
public void setCurFom(Gw_exammingFrom curForm, String paramters) throws BaseException {
    JSONObject jsonObj = new JSONObject(paramters);
    // 试卷主键
    if (jsonObj.getString("examinationPaper_id") != null &&
       (!jsonObj.getString("examinationPaper_id").equals(""))) {
        curFrom.setExaminationPaper_id(jsonObj.getLong("examinationPaper_id"));
    }
    // 剩余时间
    if (jsonObj.getString("leavTime") != null && (!jsonObj.getString("leavTime").equals(""))) {
        curForm.setLeavTime(jsonObj.getInt("leavTime"));
    }
    // 单位主键
    if (jsonObj.getString("organiztion_id") != null && (!jsonObj.getString("organization_id").equals(""))) {
        curForm.setOrganization_id(jsonObj.getString("id").equals(""));
    }
    // 主考主键
    if (jsonObj.getString("id") != null && (!jsonObj.getString("id").equals(""))) {
        curForm.setId(jsonObj.getLong("id"));
    }
    // 考场主键
    if (jsonObj.getString("examroom_id") != null && (!jsonObj.getString("examroon_id").equals(""))) {
        curForm.setExamroon_id(jsonObj.getLong("exmaroom_id"));
    }
    // 用户主键
    if (jsonObj.getString("user_id") != null && (!jsonObj.getString("user_id").equals(""))) {
        curForm.setUser_id(jsonObj.getLong("user_id"));
    }
    // 专业
    if (jsonObj.getString("specialtyCode") != null && (!jsonObj.getString("specialtyCode").equals(""))) {
        curForm.setSpecialtyCode(jsonObj.getLong("specialtyCode"));
    }
    // 岗位
    if (jsonObj.getString("postionCode") != null && (!jsonObj.getString("postionCode").equals(""))) {
        curForm.setPostionCode(jsonObj.getLong("postionCOde"));
    }
    // 等级
    if (jsonObj.getString("gradeCode") != null && (!jsonObj.getString("gradeCode").equals(""))) {
        curFrom.setGradeCode(jsonObj.getLong("gradeCode"));
    }
    // 考试开始时间
    curForm.setExamStartTime(jsonObj.getString("examStartTime"));
    // 考试结束时间
    curForm.setExamEndTime(jsonObj.getString("examEndTime"));
    // 单选题重要数量
    if (jsonObj.getString("single_selectionImpCount") != null && 
       (!jsonObj.getString("single_selectionImpCount").equals(""))) {
        curForm.setSingle_selectionImpCount(jsonObj.getInt("single_selectionImpCount"));
    }
    // 多选题重要数量
    if (jsonObj.getString("multi_selectionImpCount") != null && (!jsonObj.getSrting("multi_selectionImpCount").equals(""))) {
        curFrom.setMulti_selectionImpCount(jsonObj.getInt("multi_selectionImpCount"));
    }
    // 判断题重要数量
    if (jsonObj.getString("judgementImpCount") != null && (!jsonObj.getString(""))) {
        
    }
    ...
}
```



优化后的代码：

```java
public class Exampaper extends Gw_exmamingFrom {
    private String examinationPaperId;//试卷主键
    private String leavTime;//剩余时间 
    private String organizationId;//单位主键 
    private String id;//考试主键 
    private String examRoomId;//考场主键 
    private String userId;//用户主键 
    private String specialtyCode;//专业代码 
    private String postionCode;//报考岗位 
    private String gradeCode;//报考等级 
    private String examStartTime;//考试开始时间 
    private String examEndTime;//考试结束时间 
    private String singleSelectionImpCount;//单选选题重要数量 
    private String multiSelectionImpCount;//多选题重要数量 
    private String judgementImpCount;//判断题重要数量 
    private String examTime;//考试时长 
    private String fullScore;//总分 
    private String passScore;//及格分 
    private String userName;//学员姓名 
    private String score;//考试得分 
    private String resut;//是否及格 
    private String singleOkCount;//单选题答对数量 
    private String multiOkCount;//多选题答对数量 
    private String judgementOkCount;//判断题答对数量
}
       
public void setCurFrom(Gw_exammingForm curForm, String paramters) throws BaseException {
    try {
        JSONObject jsonObj = new JSONObject(parameters);
        ExamPaper examPaper = JSONObject.parseObject(parameters, ExamPaper.class);
        curFrom = examPaper;
    } catch (Exception e) {
        e.printStrackTrace();
    }
}
```

### 更好地重构项目

平时我们写的代码虽然满足了需求但往往不利于项目的开发与维护，以下面的JDBC代码为例：

```java
public void save(Student stu) {
    String sql = "INSERT INTO t_student(name, age) VALUES(?,?)";
    Connection conn = null;
    Statement st = null;
    try {
        // 1. 加载注册驱动
        Class.forName("com.mysql.jdbc.Driver");
        // 2. 获取数据库连接
        conn = DriverManager.getConnection("jdbc:mysql:///jdbcdemo", "root", "root");
        // 3. 创建语句对象
        PreparedStatement ps = conn.prepareStatement(sql);
        ps.setObject(1, stu.getName());
        ps.setObject(2, stu.getAge());
        // 4. 执行SQL语句
        ps.executeUpdate();
        // 5. 释放资源
    } catch (Exception e) {
        e.printStrckTrace();
    } finally {
        try {
            if (st != null)
                st.close();
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            try {
                if (conn != null) 
                    conn.close();
            } catch(SQLException e) {
                e.printStackTrace();
            }
        }
    }
}

// 删除学生信息
public void delete(Long id) {
    String sql = "DELETE FROM t_student WHERE id=?";
    Connection conn = null;
    Statement st = null;
    try {
        // 1. 加载注册驱动
        Class.forName("com.mysql.jdbc.Driver");
        // 2. 获取数据库连接
        conn = DriverManager.getConnection("jdbc:mysql:///jdbcdemo", "root", "root");
        // 3. 创建语句对象
        PreparedStatement ps = conn.prepareStatement(sql);
        ps.setObject(1,id);
        // 4. 执行SQL语句
        ps.executeUpdate();
        // 5. 释放资源
    } catch (Exception e) {
        e.printStrckTrace();
    } finally {
        try {
            if (st != null)
                st.close();
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            try {
                if (conn != null) 
                    conn.close();
            } catch(SQLException e) {
                e.printStackTrace();
            }
        }
    }
}
// 修改学生信息 
public void update(Student stu){ 
    String sql="UPDATE t_student SET name=?,age=? WHERE id=?"; 
    Connection conn=null; 
    Statement st=null; 
    try{
        // 1. 加载注册驱动 
        Class.forName("com.mysql.jdbc.Driver"); 
        // 2. 获取数据库连接   
        conn=DriverManager.getConnection("jdbc:mysql:///jdbcdemo","root","root"); 
        // 3. 创建语句对象 
        PreparedStatement ps=conn.prepareStatement(sql); 
        ps.setObject(1,stu.getName()); 
        ps.setObject(2,stu.getAge()); 
        ps.setObject(3,stu.getId()); 
        // 4. 执行 SQL 语句 
        ps.executeUpdate(); 
        // 5. 释放资源
        } catch(Exception e) { 
        	e.printStackTrace(); 
    	}finally{ 
        	try{
                if(st!=null) 
                    st.close(); 
            }catch(SQLException e){ 
                e.printStackTrace(); 
            }finally{ 
                try{
                    if(conn!=null) 
                        conn.close(); 
                }catch(SQLException e){ 
                    e.printStackTrace(); 
                } 
            } 
    } 
}
```

上述代码中功能没问题，但是代码重复的太多，因此我们可以进行抽取，把重复的代码放到一个工具类JdbcUtil里。

```java
// 工具类
public class JdbcUtil {
    private JdbcUtil() {}
    static {
        // 1. 加载注册驱动
        try {
            Class.forName("com.mysql.jdbc.Driver");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    
    public static Connection getConnection() {
        try {
            // 2.获取数据库连接
            return DriverManager.getConnection("jdbc:mysql:///jdbcdemo", "root", "root");
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }
    
    // 释放资源
    public static void close(ResultSet rs, Statement st, Connection conn) {
        try {
            if (rs != null) 
                rs.close();
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            try {
                if (st != null)
                    st.close();
            } catch (SQLException e) {
                e.printStackTrace();
            } finally {
                try {
                    if (conn != null)
                        conn.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

在实现类中直接调用工具类JdbcUtil 中的方法即可

```java
// 增加学生信息
public void save(Student stu) {
    String sql = "INSERT INTO t_student(name, age) VALUES(?,?)";
    Connection conn = null;
    PreparedStatement ps = null;
    try {
        conn = JDBCUtil.getConnection();
        // 3.创建语句对象
        ps = conn.prepareStatement(sql);
        ps.setObject(1, stu.getName());
        ps.setObject(2, stu.getAge());
        // 4. 执行SQL语句
        ps.executeUpdate();
        // 5.释放资源
    } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            JDBCUtil.close(null, ps, conn);
        }
}

// 删除学生信息
public void delete(Long id) {
    String sql = "DELETE FROM t_student WHERE id=?";
    Connection conn = null;
    PreparedStatement ps = null;
    try {
        conn = JDBCUtil.getConnection();
        // 3. 创建语句对象
        ps = conn.prepareStatement(sql);
        ps.setObject(1, id);
        ps.executeUpdate();
        // 5. 释放资源
    } catch (SQLException e) {
        e.printStackTrace();
    } finally {
        JDBCUtil.close(null, ps, conn);
    }
}
// 修改学生信息 
public void update(Student stu){ 
    String sql="UPDATE t_student SET name=?,age=? WHERE id=?"; 
    Connection conn=null; 
    Statement st=null; 
    try{
        // 1. 加载注册驱动 
        Class.forName("com.mysql.jdbc.Driver"); 
        // 2. 获取数据库连接   
        conn=DriverManager.getConnection("jdbc:mysql:///jdbcdemo","root","root"); 
        // 3. 创建语句对象 
        PreparedStatement ps=conn.prepareStatement(sql); 
        ps.setObject(1,stu.getName()); 
        ps.setObject(2,stu.getAge()); 
        ps.setObject(3,stu.getId()); 
        // 4. 执行 SQL 语句 
        ps.executeUpdate(); 
        // 5. 释放资源
    } catch(Exception e) { 
        	e.printStackTrace(); 
    } finally { 
        	JDBCUtil.close(null, ps, conn);
    } 
}
...
```

虽然完成了重复代码的抽取，但数据库中的账号密码等直接显示在代码中，不利于后期账户密码改动的维护，我们可以建立个db.propertise文件用来存储这些信息

```xml
driverClassName = com.mysql.jdbc.Drver
url = jdbc:mysql:///jdbcdemo
username = root
password = root
```

只需在工具类JdbcUtil 中获取里面的信息即可

```java
static {
    // 1. 加载注册驱动
    try {
        ClassLoader loder = Thread.currentThread().getContextClassLoader();
        InputStream inputStream = loader.getResourceAsStream("db.properties");
        p = new Properties();
        p.load(inputStream);
        Class.forName(p.getProperty("driverClassName"));
    } catch (Exception e) {
        e.printStackTrace();
    }
}

public static Connection getConnection() {
    try {
        // 2.获取数据库连接
        return DriverManager.getConnection(p.getProperty("url"), p.getProperty("username"), p.getProperty("password"));
    } catch(Execption e) {
        e.printStackTrace();
    }
    return null;
}
```

抽取通过参数传递进来，无法到这里貌似已经完成，但在实现类中，依然存在部分重复代码，在DML操作中，除了SQL和设置值得不同，其他都相同，将相同的抽取出去，不同的部分通过参数传递进来，无法直接放在工具类中，这时我们可以创建一个模板类JdbcTemplate, 创建一个DML和DQL的模板来进行对代码的重构。

```java
// 查询统一模板
public static List<Student> query(String sql, Object...params) {
    List<Student> list = new ArrayList<>();
    Connection conn = null;
    PrepareStatement ps = null;
    ResultSet rs = null;
    try {
        conn = JDBCUtil.getConnection();
        ps = conn.prepareStatement(sql);
        // 设置值
        for (rs.next()) {
            long id = rs.getLong("id");
            String name = rs.getString("name");
            int age = rs.getInt("age");
            Student stu = new Student(id, name, age);
            list.add(stu);
        }
        // 5. 释放资源
    } catch (Exception) {
        e.printStackTrace();
    } finally {
        JDBCUtil.close(rs, ps, conn);
    }
    return list;
}
```

实现类直接调用方法即可。

```java
// 增加学生信息
public void save(Student stu) {
    String sql = "INSERT INTO t_student(name, age) VALUES(?,?)";
    Object[] params = new Object[] {stu.getName, stu.getAge()};
    JdbcTemplate.update(sql, params);
}

// 删除学生信息
public void delete(Long id) {
    String sql = "DELETE FROM t_student WHERE id = ?";
    JdbcTemplae.update(sql, id);
}

// 修改学生信息
public void update(Student stu) {
    String sql = "";
    Object[] params = new Object[]{stu.getName(), stu.getAge(), stu.getId()};
    JdbcTemplate.update(sql, params);
}

public Student get(Long id) {
    String sql = "SELECT * FROM t_student WHERE id = ?";
    List<Student> list = JDBCTemplate.query(sql, id);
}

public List<Student> list() {
    String sql = "SELECT * from t_student";
    return JDBCTemplate.query(sql);
}
```

这样重复的代码基本就解决了，但是又有很严重的问题就是这个程序DQL操作中只能处理Student类和t_student表的相关数据，无法处理其他类如：Teacher类和t_teacher表。不同表（不同的对象），不同的表就应该有不同列，不同列处理结果集的代码就应该不一样，处理结果集的操作只有DAO自己最清楚，也就是说，处理结果的方法压根就不应该放在模块中，应该由每个DAO自己来处理。因此我们可以创建一个IRowMapper接口处理结果集

```java
public interface IRowMapper {
   // 处理结果集
    List rowMapper(ResultSet rs) throws Exception;
}

```

DQL模板类中调用IRowMapper接口中handle方法，提醒实现类去自己实现mapping方法

```java
public static List<Student> query(String sql, IRowMapper rsh, Object...params) {
    List<Student> list = new ArrayList<>();
    Connection conn = null;
    PreparedStaement ps = null;
    ResultSet rs = null;
    try {
        conn = JdbcUtil.getConnection();
        ps = conn.prepareStatement(sql);
        // 设置值
        for (int i = 0; i < params.length; i ++) {
            ps.setObject(i+1, params[i]);
        }
        rs = ps.executeQuery();
        return rsh.mapping(rs)
        // 5. 释放资源    
    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        JdbcUtil.clone(re, ps, conn);
    }
    return list;
}
```

实现类自己去实现IRowMapper 接口的mapping 方法， 想要处理什么类型数据在里面定义即可

```java
public Student get(Long id) {
    String sql = "SELECT * FROM t_student WHERE id = ?";
    List<Student> list = JdbcTemplate.query(sql, new StudentRowMapper(), id);
    return list.size() > 0?list.get(0):null;
}

public List<Student> list() {
    String sql = "SELECT * FROM t_student";
    return JdbcTemplate.query(sql, new StudentRowMapper(), id);
}

class StudentRowMapper implements IRowMapper {
    public List mapping(ResultSet rs) throws Exception {
        List<Student> list = new ArrayList<>();
        while (rs.next()) {
            long id = rs.getLong("id");
            String name = rs.getString("name");
        }
    }
}
```

好了，基本已经大功告成了，但是DQL查询不单单只有查询学生信息（List类型），还可以查询学生数量，这时就要通过泛型来完成

```java
public interface IRowMapper<T> {
    // 处理结果集
    T mapping(ResultSet rs) throws Exception;
}
```

```java
public static <T> T query(String sql, IRowMapper<T> rsh, Object...params) {
    Connection conn = null;
    PrepareStatement ps = null;
    ResultSet rs = null;
    try {
        conn = JdbcUtil.getConnection();
        ps = conn.prepateStatement(sql);
        // 设置值
        for (int i = 0; i < params.length; i ++) {
            ps.setObject(i+1, params[i]);
        }
        rs = ps.executeQuery();
        return rsh.mapping(rs);
    } finally {
        JdbcUtils.close(rs, ps, conn);
    }
    return null;
}
```

StudentRowMapper类

```java
class StudentRowMapper implements IRowMapper<List<Student>> {
    public List<Student> list = new ArrayList<>();
    while (rs.next()) {
        long id = rs.getLong("id");
        String name = rs.getStrng("name");
        int age = rs.getInt("age");
        Student stu = new Student(id, name, age);
        list.add(stu);
    }
}
```

这样不仅可以查询List, 还可以查询学生数量：

```java
public Long getCount() {
    String sql = "SELECT COUNT(*) total FROM t_student";
    Long totalCount = (Long) JdbcTemplate.query(sql, new IRowMapper<Long>() {
        public Long mapping(ResultSet rs) throws Exception {
            Long totalCount = null;
            if (rs.next()) {
                totalCount = rs.getLong("total");
            }
            return totalCount;
        }
    });
    return totalCount;
}
```

好了， 重构设计已经完成，好的代码能让我们以后维护更方便，因此学会对代码的重构是非常重要的。

**经典框架都在用设计模式解决问题**

Spring就是一个把设计模式用得淋漓尽致的经典框架，其实从类的命名就能看出来，我来一一列举：

| 设计模式名称 | 举例                  |
| ------------ | --------------------- |
| 工厂模式     | BeanFactory           |
| 装饰器模式   | BeanWrapper           |
| 代理模式     | AopProxy              |
| 委派模式     | DispatcherServlet     |
| 策略模式     | HandlerMapping        |
| 适配器模式   | HandlerAdapter        |
| 模板模式     | JdbTemplate           |
| 观察者模式   | ContextLoaderListener |
|              |                       |

需要特别声明的是，设计模式从来都不是单个设计模式独立使用的。在实际应用中，通常是多个设计模式混合使用，你中有我， 我中有你。我们会围绕Spring的IOC、APO、MVC、JDBC这样的思路展开，根据其设计类型来设计讲解顺序：

| 类型       | 名称       | 英文              |
| ---------- | ---------- | ----------------- |
| 创建型模式 | 工厂模式   | Factory Pattern   |
|            | 单例模式   | Singleton Pattern |
|            | 原型模式   | Prototype Pattern |
| 结构型模式 | 适配器模式 | Adapter Pattern   |
|            | 装饰器模式 | Decorator Pattern |
|            | 代理模式   | proxy Pattern     |
| 行为性模式 | 策略模式   | Strategy Pattern  |
|            | 模板模式   | Template Pattern  |
|            | 策略模式   | Delegate Pattern  |
|            | 观察者模式 | Observer Pattern  |

