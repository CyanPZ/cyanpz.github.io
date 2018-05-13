---
title: Spring Boot JPA (一)
date: "2017-06-28 21:14:04"
tags:
---

### 使用Spring-Data-JPA进行数据库操作只需四步 ###
#### 1. 声明一个接口继承Repository或者Repository的子接口   #####
    interface PersonRepository extends Repository<Person, Long{ … }
#### 2. 声明所要的查询方法 ####
    interface PersonRepository extends Repository<Person, Long> {
      List<Person> findByLastname(String lastname);
    }
#### 3. 设置Spring支持Spring-Data-JPA ####
    import org.springframework.data.jpa.repository.config.EnableJpaRepositories;
    
    @EnableJpaRepositories
    class Config {}
#### 4. 注入repository实例 ####
    public class SomeClient {
    
      @Autowired
      private PersonRepository repository;
    
      public void doSomething() {
    	List<Person> persons = repository.findByLastname("Matthews");
      }
    }
### CrudRepository提供的方法 ###
    public interface CrudRepository<T, ID extends Serializable>
    extends Repository<T, ID> {
    
    < S extends T > S save(S entity); 
    
    T findOne(ID primaryKey);   
    
    Iterable< T > findAll();  
    
    Long count();   
    
    void delete(T entity);  
    
    boolean exists(ID primaryKey);  
    
    // … more functional

### PagingAndSortingRepository提供的方法 ###
    public interface PagingAndSortingRepository<T, ID extends Serializable>
      extends CrudRepository<T, ID> {
    
      Iterable<T> findAll(Sort sort);
    
      Page<T> findAll(Pageable pageable);
    }
### 创建查询语句 ###
Spring-Data中的查询生成器会根据接口方法名中 find…By，read…By，query…By，count…By，get…By关键字来生成查询语句，并且可以使用And或者Or

    public interface PersonRepository extends Repository<User, Long> {
    
      List<Person> findByEmailAddressAndLastname(EmailAddress emailAddress, String lastname);
    
      // Enables the [distinct](http://www.w3school.com.cn/sql/sql_func_count_distinct.asp) flag for the query
      List<Person> findDistinctPeopleByLastnameOrFirstname(String lastname, String firstname);
      List<Person> findPeopleDistinctByLastnameOrFirstname(String lastname, String firstname);
    
      // Enabling ignoring case for an individual property
      List<Person> findByLastnameIgnoreCase(String lastname);
      // Enabling ignoring case for all suitable properties
      List<Person> findByLastnameAndFirstnameAllIgnoreCase(String lastname, String firstname);
    
      // Enabling static ORDER BY for a query
      List<Person> findByLastnameOrderByFirstnameAsc(String lastname);
      List<Person> findByLastnameOrderByFirstnameDesc(String lastname);
    }

Spring Data JPA 提供的关键字　　


关键字 | 示例 | JPQL　　
----|------|----
And	|	findByLastnameAndFirstname	|	… where x.lastname = ?1 and x.firstname = ?2
Or	|	findByLastnameOrFirstname	|	… where x.lastname = ?1 or x.firstname = ?2
Is,Equals	|	findByFirstname,findByFirstnameIs,findByFirstnameEquals	|	… where x.firstname = ?1
Between	|	findByStartDateBetween	|	… where x.startDate between ?1 and ?2
LessThan	|	findByAgeLessThan	|	… where x.age < ?1
LessThanEqual	|	findByAgeLessThanEqual	|	… where x.age <= ?1
GreaterThan	|	findByAgeGreaterThan	|	… where x.age > ?1
GreaterThanEqual	|	findByAgeGreaterThanEqual	|	… where x.age >= ?1
After	|	findByStartDateAfter	|	… where x.startDate > ?1
Before	|	findByStartDateBefore	|	… where x.startDate < ?1
IsNull	|	findByAgeIsNull	|	… where x.age is null
IsNotNull,NotNull	|	findByAge(Is)NotNull	|	… where x.age not null
Like	|	findByFirstnameLike	|	… where x.firstname like ?1
NotLike	|	findByFirstnameNotLike	|	… where x.firstname not like ?1
StartingWith	|	findByFirstnameStartingWith	|	… where x.firstname like ?1 (parameter bound with appended %)
EndingWith	|	findByFirstnameEndingWith	|	… where x.firstname like ?1 (parameter bound with prepended %)
Containing	|	findByFirstnameContaining	|	… where x.firstname like ?1 (parameter bound wrapped in %)
OrderBy	|	findByAgeOrderByLastnameDesc	|	… where x.age = ?1 order by x.lastname desc
Not	|	findByLastnameNot	|	… where x.lastname <> ?1
In	|	findByAgeIn(Collection<Age> ages)	|	… where x.age in ?1
NotIn	|	findByAgeNotIn(Collection<Age> age)	|	… where x.age not in ?1
TRUE	|	findByActiveTrue()	|	… where x.active = true
FALSE	|	findByActiveFalse()	|	… where x.active = false
IgnoreCase	|	findByFirstnameIgnoreCase	|	… where UPPER(x.firstame) = UPPER(?1)



### 特殊参数 ###
在查询方法中使用Pageable，Slice和Sort
    Page<User> findByLastname(String lastname, Pageable pageable);
    
    Slice<User> findByLastname(String lastname, Pageable pageable);
    
    List<User> findByLastname(String lastname, Sort sort);
    
    List<User> findByLastname(String lastname, Pageable pageable);
上例方法一返回Page，Page中包含了总数，这在查询中需要更大的代价，方法二返回Slice，Slice只能知道是否还有下一个Slice，如果是一个很大的集合，使用Slice足够。
### 限制结果集 ###
结果集可以通过关键字first或top来限制,并且可以跟上一个数字来限制返回的数量，如果不指定数组，默认为1  

    //返回按Lastname升序排序的第一个
    User findFirstByOrderByLastnameAsc();
    //返回按Lastname升序排序的第一个
    User findTopByOrderByAgeDesc();
    //返回按Lastname升序排序的前10个,并且分页
    Page<User> queryFirst10ByLastname(String lastname, Pageable pageable);
    
    Slice<User> findTop3ByLastname(String lastname, Pageable pageable);
    //返回按Lastname升序排序的前10个
    List<User> findFirst10ByLastname(String lastname, Sort sort);
    
    List<User> findTop10ByLastname(String lastname, Pageable pageable);
### 使用自定义查询 ###
- 在接口中使用@Query注解自定义查询  

	    public interface UserRepository extends JpaRepository<User, Long> {
	      @Query("select u from User u where u.emailAddress = ?1")
	      User findByEmailAddress(String emailAddress);
	    }
    


- 使用自定义查询返回特定字段  
根据deptName返回DeptNo   

	    public interface DepartmentsDAO extends CrudRepository<Departments,String> {
		    @Query("select d.deptNo from Departments d where d.deptName=?1")
		    String findDeptNoByDeptName(String deptName);
	    }



- 使用命名参数  
在自定义查询中使用命名参数  

		public interface UserRepository extends JpaRepository<User, Long> {
		  @Query("select u from User u where u.firstname = :firstname or u.lastname = :lastname")
		  User findByLastnameOrFirstname(@Param("lastname") String lastname,
		 @Param("firstname") String firstname);
		}

### 修改查询 ###
在@Query注解中可以更新数据库  

	@Modifying
	@Query("update User u set u.firstname = ?1 where u.lastname = ?2")
	int setFixedFirstnameFor(String firstname, String lastname);

### 返回部分字段 ###
在查询中可以返回部分字段。
假定有实体类   

    @Entity
    public class Person {
    
      @Id @GeneratedValue
      private Long id;
      private String firstName, lastName;
    
      @OneToOne
      private Address address;
      …
    }
    
    @Entity
    public class Address {
    
      @Id @GeneratedValue
      private Long id;
      private String street, state, country;
    
      …
    }

现在不返回Person表中的Address属性

1. 定义一个返回类型(带有getter的interface)

	    interface NoAddresses {  
	    
	      String getFirstName(); 
	    
	      String getLastName();  
	    }
2. 将Repository中的返回值改为定义的返回类型

		interface PersonRepository extends CrudRepository<Person, Long> {
		
		  NoAddresses findByFirstName(String firstName);
		}
