# 1、简单类开发要求



在实际的工作中，针对简单Java类的开发给出如下要求：

- 考虑到日后程序有可能出现的分布式应用问题，所以简单Java类必须要实现就java.io.Serializable接口；
- 简单Java类的名称必须与表名保持一致；
    - 有可能采用这样的名字：student_info，类名称为：StudentInfo；
- 类中的属性不允许使用基本数据类型，都必须使用基本数据类型的包装类；
    - 基本数据类型的默认值为0，而如果是包装类型的默认值为null；（我们需要的是null）
- 类中的属性必须使用private封装，同时给出setter、getter方法
- 类中可以定义有多个构造方法，但是必须保留一个无参数的构造方法。
- 【可选要求，基本不用】 覆写equals()、toString()、hashCode()；

```java
public class Emp implements Serializable {
    private Integer empno;
    private String ename;
    private String job;
    private Data hiredata;
    private Double sal;
    private Double comm;

    public Integer getEmpno() {
        return empno;
    }

    public void setEmpno(Integer empno) {
        this.empno = empno;
    }
```

不写构造方法，也就代表有一个无参的构造方法。



# 2、数据层开发要求



## 2.1、接口定义



对于数据层的接口给出如下的开发要求：

- 数据层既然是进行数据操作的，那么就将其保存在dao包下；
- 既然不同的数据表的操作有可能使用不同的数据层开发，那么就针对于数据表进行命名；
    - emp表，那么数据层的接口就应该定义为IEmpDAO；
- 对于整个数据层的开发严格来讲就只有两类功能：
    - 数据更新：建议它的操作方法以doXxx()的形式命名，例如：doCreate() doRemove；
    - 数据查询，对于查询分为两种形式：
        - 查询表中数据：以findXxx()形式命名，例如：findById()
        - 统计表中的数据：以getXxx()形式命名，例如：getAllCustomers()



范例：定义IEmpDAO接口

```java
/**
 * 定义Emp数据层的接口规范
 * @author Kicc on 20/6/9.
 */
public interface IEmpDAO {
    /**
     * 实现数据的增加操作
     * @param vo 包含了要增加数据的vo对象
     * @return 数据保存成功，返回true，否则返回false
     * @throws SQLException
     */
    public boolean doCreate(Emp vo) throws SQLException;


    /**
     * 实现数据的修改操作，本次修改是根据id进行全部字段的修改
     * @param vo 包含了需要修改的vo对象, 一定要提供ID内容
     * @return 成功：true
     * @throws SQLException
     */
    public boolean doUpdate(Emp vo) throws SQLException;


    /**
     * 执行数据的批量删除操作，所有要删除的数据以Set集合的形式
     * @param ids 包含了所有删除的数据ID，不包含重复内容
     * @return 删除成功（全部成功）为true
     * @throws SQLException
     */
    public boolean doRemoveBatch(Set<Integer> ids) throws SQLException;

    /**
     * 根据id号查询雇员的信息
     * @param id 要查询的雇员编号
     * @return 如果emp存在，就返回Emp，否则返回null
     *
     * @throws SQLException
     */
    public Emp findByID(Integer id) throws SQLException;

    /**
     * 查询指定数据表的全部记录，并且以集合的形式返回
     * @return 如果表中有数据， 封装为VO对象后以List形式返回
     * 如果没有数据，那么集合的长度（size() == 0， 不是null)
     * @throws SQLException
     */
    public List<Emp> findAll() throws SQLException;

    /**
     * 分页进行数据的模糊查询，查询的结果以集合的形式返回
     * @param currentPage 当前所在的页
     * @param lineSize 显示数据行数
     * @param column 要进行模糊查询的数据列
     * @param keyWord 模糊查询的关键字
     * @return 如果表中有数据， 封装为VO对象后以List形式返回
     * 如果没有数据，那么集合的长度（size() == 0， 不是null)
     * @throws SQLException
     */
    public List<Emp> findAllSplit(Integer currentPage, Integer lineSize, String column, String keyWord) throws SQLException;
}
```

这是属于数据层层面的操作，每一个操作都是一个原子性的操作。而具体的业务逻辑操作是由业务层完成的。业务层会来调用数据层中这些原子性操作。





## 2.2、数据层具体实现

所有的数据层实现类要求保存在dao.impl子包下。

范例：EmpDAOImpl子类

