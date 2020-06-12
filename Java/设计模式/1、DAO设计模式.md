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
    
    /**
     * 进行模糊查询了数据量的统计，如果表中没有记录统计的结果就是0
     * @param column 要进行模糊查询的数据列
     * @param KeyWord 模糊查询的关键字
     * @return 所有数据量，Integer，如果没有数据，返回0
     * @throws SQLException
     */
    public Integer getAllCount(String column, String KeyWord) throws SQLException;

}
```

这是属于数据层层面的操作，每一个操作都是一个原子性的操作。而具体的业务逻辑操作是由业务层完成的。业务层会来调用数据层中这些原子性操作。





## 2.2、数据层具体实现

所有的数据层实现类要求保存在dao.impl子包下。

范例：EmpDAOImpl子类

```java
package shejimoui.DAO.dao.impl;

import com.nowcoder.ListNode;
import shejimoui.DAO.dao.IEmpDAO;
import shejimoui.DAO.pojo.Emp;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;
import java.util.Set;

/**
 * @author Kicc on 20/6/9.
 */
public class EmpDAOImpl implements IEmpDAO {
    private Connection conn;
    private PreparedStatement pstmt;

    /**
     * 如果要使用数据进行原子性的功能操作实现，必须要提供有Connection接口对象
     * 另外，由于开发之中业务层要调用数据层，所以数据库的打开和关闭由业务层处理
     * @param conn 表示数据库连接对象
     */
    public EmpDAOImpl(Connection conn) {
        this.conn = conn;
    }


    /**
     *
     * @param vo 包含了要增加数据的vo对象
     * @return
     * @throws SQLException
     */

    @Override
    public boolean doCreate(Emp vo) throws SQLException {
        String sql = "INSERT INTO emp(empno,ename,job,hiredate,sal,comm) VALUES (?,?,?,?,?,?)";
        this.pstmt = this.conn.prepareStatement(sql);
        this.pstmt.setInt(1, vo.getEmpno());
        this.pstmt.setString(2, vo.getEname());
        this.pstmt.setString(3, vo.getJob());
        this.pstmt.setDate(4, new java.sql.Date(vo.getHiredate().getTime()));
        this.pstmt.setDouble(5, vo.getSal());
        this.pstmt.setDouble(6, vo.getComm());
        return this.pstmt.executeUpdate() > 0 ;
    }

    /**
     *
     * @param vo 包含了需要修改的vo对象, 一定要提供ID内容
     * @return
     * @throws SQLException
     */
    @Override
    public boolean doUpdate(Emp vo) throws SQLException {
        String sql = "UPDATE emp SET ename=?,job=?,hiredate=?,sal=?,comm=? WHERE empno=?";
        this.pstmt = this.conn.prepareStatement(sql);
        this.pstmt.setString(1, vo.getEname());
        this.pstmt.setString(2, vo.getJob());
        this.pstmt.setDate(3, new java.sql.Date(vo.getHiredate().getTime()));
        this.pstmt.setDouble(4, vo.getSal());
        this.pstmt.setDouble(5, vo.getComm());
        this.pstmt.setInt(6, vo.getEmpno());

        return this.pstmt.executeUpdate() > 0 ;
    }

    /**
     * DELETE FROM emp WHERE empno IN (?,?,?,?)
     * @param ids 包含了所有删除的数据ID，不包含重复内容
     * @return
     * @throws SQLException
     */
    @Override
    public boolean doRemoveBatch(Set<Integer> ids) throws SQLException {
        if (ids==null || ids.size()==0){
            return false;
        }

        StringBuilder sql = new StringBuilder();
        sql.append("DELETE FROM emp WHERE empno IN (");

        Iterator<Integer> iter = ids.iterator();
        while (iter.hasNext()) {
            sql.append(iter.next()).append(",");
        }

        sql.delete(sql.length() - 1, sql.length());
        sql.append(")");
        this.pstmt = this.conn.prepareStatement(sql.toString());
        return this.pstmt.executeUpdate() == ids.size();

    }

    @Override
    public Emp findByID(Integer id) throws SQLException {
        Emp vo = null;
        String sql = "SELECT empno, ename, job, hiredate, sal, comm FROM emp WHERE empno=?";

        this.pstmt = this.conn.prepareStatement(sql);
        this.pstmt.setInt(1, id);
        ResultSet rs = this.pstmt.executeQuery();
        if (rs.next()){
            vo = new Emp();
            vo.setEmpno(rs.getInt(1));
            vo.setEname(rs.getString(2));
            vo.setJob(rs.getString(3));
            vo.setHiredata(rs.getDate(4));
            vo.setSal(rs.getDouble(5));
            vo.setComm(rs.getDouble(6));
        }
        return vo;
    }

    @Override
    public List<Emp> findAll() throws SQLException {
//        List<Emp> vos = null;
        List<Emp> vos = new ArrayList<Emp>();
        String sql = "SELECT empno, ename, job, hiredate, sal, comm FROM emp";

        this.pstmt = this.conn.prepareStatement(sql);

        ResultSet rs = this.pstmt.executeQuery();

        while (rs.next()) {
            Emp vo = new Emp();
            vo.setEmpno(rs.getInt(1));
            vo.setEname(rs.getString(2));
            vo.setJob(rs.getString(3));
            vo.setHiredata(rs.getDate(4));
            vo.setSal(rs.getDouble(5));
            vo.setComm(rs.getDouble(6));
            vos.add(vo);
        }

        return vos;
    }

    /**
     * 分页查询
     * @param currentPage 当前所在的页
     * @param lineSize 显示数据行数
     * @param column 要进行模糊查询的数据列
     * @param keyWord 模糊查询的关键字
     * @return
     * @throws SQLException
     */
    @Override
    public List<Emp> findAllSplit(Integer currentPage, Integer lineSize, String column, String keyWord) throws SQLException {

        List<Emp> vos = new ArrayList<Emp>();
        String sql = "SELECT * FROM "
                + " (SELECT empno, ename, job, hiredate, sal, comm, ROWNUM rn"
                + " FROM emp"
                + " WHERE " + column + " LIKE ? AND ROWNUM<=?) temp "
                + " WHERE temp.rn>?";

        this.pstmt = this.conn.prepareStatement(sql);
        this.pstmt.setString(1, "%" + keyWord + "%");
        this.pstmt.setInt(2, currentPage * lineSize);
        this.pstmt.setInt(3, (currentPage -1 ) * lineSize);

        ResultSet rs = this.pstmt.executeQuery();

        while (rs.next()) {
            Emp vo = new Emp();
            vo.setEmpno(rs.getInt(1));
            vo.setEname(rs.getString(2));
            vo.setJob(rs.getString(3));
            vo.setHiredata(rs.getDate(4));
            vo.setSal(rs.getDouble(5));
            vo.setComm(rs.getDouble(6));
            vos.add(vo);
        }

        return vos;
    }
    
    @Override
    public Integer getAllCount(String column, String KeyWord) throws SQLException {
        Integer count = 0;
        String sql = "SELECT COUNT(empno) FROM emp WHERE " + column + " LIKE ?";
        this.pstmt = this.conn.prepareStatement(sql);
        this.pstmt.setString(1, "%" + KeyWord + "%");
        ResultSet rs = this.pstmt.executeQuery();
        if (rs.next()) {
            count = rs.getInt(1);
            return count;
        }
        return null;
    }
}
```

实现类中多了一个构造方法。需要传入connection.这个Connection是另外层数据库连接后得到的。因此DAO不关心数据库的具体连接。



为了使业务层不关心具体子类的实现，我们添加Factory

```java
public class DAOFactory {
    public static IEmpDAO getIEmpDAOInstance(Connection conn) {
        return new EmpDAOImpl(conn) ;

    }
}
```



操作逻辑：

1. 业务层开始连接数据库，返回connection
2. 业务层去执行逻辑，与Factory交互。
3. Factory与具体子类EmpDAOImpl实现交互，返回结果。DAO层都是原子性操作
4. 返回给业务层



# 3、业务层开发

业务层是真正留给外部进行调用的，可能是控制层，或者是直接调用。（没有Controller就直接调用）



## 3.1、定义业务层的操作标准 IEmpService

既然以emp表举例，所以名称取为IEmpService。



**范例：**定义IEmpService操作标准

```java
/**
 * 定义emp表的业务层操作标准， 此类负责数据库的打开与关闭操作
 * 通过DAOFactory获取Connection
 * @author Kicc on 20/6/10.
 */
public interface IEmpService {

    /**
     * 实现雇员数据的增加操作，本次操作要调用IEmpDAO接口的如下方法
     * 需要调用IEmpDAO.findById()方法，判断要增加的数据id是否已经存在
     * 如果现在要增加的数据已经存在，则直接返回；否则调用IEmpDAO.doCreate()方法，返回成功\失败
     * @param vo 需要插入的雇员信息
     * @return
     */
    public boolean insert(Emp vo) throws Exception;

    /**
     * 实现雇员数据的更新，本次操作要调用如下方法
     * IEmpDAO.findById()需要要更新的Emp是否存在
     * 如果不存在，那么调用insert方法插入
     * 如果存在，调用IEmpDAO.doUpdate()进行更新
     * @param vo 需要更改的雇员信息
     * @return 修改成功返回true
     * @throws Exception
     */
    public boolean update(Emp vo) throws Exception;


    /**
     * 根据雇员的编号删除雇员信息，可以删除多个雇员信息，调用IEmpDAO.doRemoveBatch()
     * @param ids 包含了全部要删除的数据集合，没有重复的数据
     * @return 成功删除返回true，否则false
     * @throws Exception
     */
    public boolean delete(Set<Integer> ids) throws Exception;


    /**
     * 根据雇员的编号，查找雇员的完整信息，调用IEmpDAO.findById()
     * @param id 查找雇员的编号
     * @return 如果找到了，则以VO对象返回；否则返回null
     * @throws Exception
     */
    public Emp get(Integer id) throws Exception;


    /**
     * 查询所有雇员的信息，调用IEmpDAO。findAll()
     * @return 所有雇员的信息
     * @throws Exception
     */
    public List<Emp> list() throws Exception;


    /**
     * 实现数据的模糊查询与数据统计，要调用IEmpDAO的两个方法
     * 调用IEmpDAO.findAllSplit()方法，查询出所有的数据量，返回List<Emp>
     * 调用IEmpDAO.getAllCount() 方法，查询所有的数据量， 返回Integer；
     * @param currentPage 当前所在页
     * @param lineSize 每页显示的记录数
     * @param column 模糊查询的数据列
     * @param keyWord 模糊查询的关键字
     * @return 本方法由于需要返回多种数据类型，所以使用Map<返回。由于类型不统一，所以value用Object表示。
     * <li>
     *     key =  allEmps, value = IEmpDAO.findAllSplit() 返回结果，List<Emp>
     *     key = empCount, value = IEmpDAO.getAllCount() 返回结果， Integer;
     * </li>
     * @throws Exception
     */
    public Map<String, Object> list(int currentPage, int lineSize, String column, String keyWord) throws Exception;
}
```



在最后一个list()方法中，因为不确定最终返回结果的类型。因此用Map集合。



## 3.2、业务层实现类

业务层实现类的核心功能：

- 负责控制数据库的打开和关闭
- 根据DAOFactory调用getIEmpDAOInstance()方法 得到数据实现类的实例。

实现类包为 service.impl



EmpServiceImpl.java

```java
public class EmpServiceImpl implements IEmpService {
    private DataBaseConnection dbc = new MySQLConnection();

    @Override
    public boolean insert(Emp vo) throws Exception {
        try {
            if (DAOFactory.getIEmpDAOInstance(this.dbc.getConn()).findByID(vo.getEmpno()) == null) {
                return DAOFactory.getIEmpDAOInstance(this.dbc.getConn()).doCreate(vo);
            }
            return false ;
        } catch (Exception e) {
            throw e;

        } finally {
            this.dbc.close();
        }
    }

    @Override
    public boolean update(Emp vo) throws Exception {
        try {
            if (DAOFactory.getIEmpDAOInstance(this.dbc.getConn()).findByID(vo.getEmpno()) == null) {
                return insert(vo);
            } else {
                return DAOFactory.getIEmpDAOInstance(this.dbc.getConn()).doUpdate(vo);
            }
        } catch (Exception e) {
            throw e;

        } finally {
            this.dbc.close();
        }
    }

    @Override
    public boolean delete(Set<Integer> ids) throws Exception {
        try {
            return DAOFactory.getIEmpDAOInstance(this.dbc.getConn()).doRemoveBatch(ids);
        } catch (Exception e) {
            throw e;

        } finally {
            this.dbc.close();
        }
    }

    @Override
    public Emp get(Integer id) throws Exception {
        try {
            return DAOFactory.getIEmpDAOInstance(this.dbc.getConn()).findByID(id);
        } catch (Exception e) {
            throw e;

        } finally {
            this.dbc.close();
        }
    }

    @Override
    public List<Emp> list() throws Exception {
        try {
            return DAOFactory.getIEmpDAOInstance(this.dbc.getConn()).findAll();
        } catch (Exception e) {
            throw e;

        } finally {
            this.dbc.close();
        }
    }

    @Override
    public Map<String, Object> list(int currentPage, int lineSize, String column, String keyWord) throws Exception {
        try {
            Map<String, Object> map = new HashMap<String, Object>();
            map.put("allEmps", DAOFactory.getIEmpDAOInstance(this.dbc.getConn()).findAllSplit(currentPage, lineSize, column, keyWord));
            map.put("empCount", DAOFactory.getIEmpDAOInstance(this.dbc.getConn()).getAllCount(column, keyWord));
            return map;
        } catch (Exception e) {
            throw e;

        } finally {
            this.dbc.close();
        }
    }
}
```



需要拥有具体数据库Connection字段，来保证连接。



在每一个操作中：

- 通过DAOFactory调用静态方法获得EmpDAO的实例。
- 通过实例去调用DAO层的各个原子性操作来完成业务逻辑。





测试