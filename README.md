# test
Neste exemplo, demonstraremos como configurar o Spring Framework para se comunicar com o banco de dados usando JPA e Hibernate como fornecedor do JPA. Os benefícios de usar o Spring Data é que ele remove muito código clichê e fornece uma
e implementação mais legível da camada DAO. Além disso, ajuda a tornar o código livremente acoplado e, como tal, alternar entre diferentes fornecedores de JPA é uma questão de configuração.
Então, vamos configurar o banco de dados para o exemplo. Usaremos o banco de dados MySQL para esta demonstração. Criamos uma tabela "funcionário" com
2 colunas como mostrado:

```
CREATE TABLE ‘employee‘ (
‘employee_id‘ bigint(20) NOT NULL AUTO_INCREMENT,
‘employee_name‘ varchar(40) ,
PRIMARY KEY (‘employee_id‘)
)

```

E aqui está a estrutura do projeto:













Em primeiro lugar, criamos a classe Employee com EmployeesId e EmployeesName. A classe Person será a entidade que vamos armazenar e recuperar do banco de dados usando o JPA.
O @Entity marca a classe como a Entidade JPA. Mapeamos as propriedades da classe Employee com as colunas da classe Employee tabela e a entidade com a própria tabela de funcionários usando a anotação @Table.
O método toString é substituído para que possamos obter uma saída significativa quando imprimirmos a instância da classe.


Employee.java
```
package com.jcg.bean;
import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.Table;
@Entity
@Table(name="employee")
public class Employee
{
@Id
@GeneratedValue(strategy=GenerationType.AUTO)
@Column(name = "employee_id")
private long employeeId;
@Column(name="employee_name")
private String employeeName;
public Employee()
{
}
public Employee(String employeeName)
{
this.employeeName = employeeName;
}
public long getEmployeeId()
{
return this.employeeId;
}
public void setEmployeeId(long employeeId)
{
this.employeeId = employeeId;
}
public String getEmployeeName()
{
return this.employeeName;
}
public void setEmployeeName(String employeeName)
{
this.employeeName = employeeName;
}
@Override
public String toString()

{
return "Employee [employeeId=" + this.employeeId + ", employeeName=" + this ←-
.employeeName + "]";
}
}

```

Uma vez que a Entidade esteja pronta, definimos a interface para armazenamento e recuperação da entidade, ou seja, criaremos um Data Access Interface.

EmployeeDao.java
```
package com.jcg.dao;
import java.sql.SQLException;
import com.jcg.bean.Employee;
public interface EmployeeDao
{
void save(Employee employee) throws SQLException;
Employee findByPrimaryKey(long id) throws SQLException;
}

```

Em seguida, tentaremos implementar a interface de acesso a dados e criar o objeto de acesso a dados real que modificará o Person entity.


EmployeeDaoImpl.java
```
package com.jcg.impl;
import javax.persistence.EntityManager;
import javax.persistence.PersistenceContext;
import org.springframework.stereotype.Repository;
import org.springframework.transaction.annotation.Propagation;
import org.springframework.transaction.annotation.Transactional;
import com.jcg.bean.Employee;
import com.jcg.dao.EmployeeDao;
@Repository("EmployeeDaoImpl")
@Transactional(propagation = Propagation.REQUIRED)
public class EmployeeDaoImpl implements EmployeeDao
{
@PersistenceContext
private EntityManager entityManager;
@Override
public void save(Employee employee)
{
entityManager.persist(employee);
}
@Override
public Employee findByPrimaryKey(long id)
{
Employee employee = entityManager.find(Employee.class, id);
return employee;
}

/**
* @return the entityManager
*/
public EntityManager getEntityManager()
{
return entityManager;
}
/**
* @param entityManager the entityManager to set
*/
public void setEntityManager(EntityManager entityManager)
{
this.entityManager = entityManager;
}
}

```
A classe DAO Implementation é anotada com @Repository, que marca como um Repository Bean e solicita o Spring Bean Factory para carregar o Bean. @Transactional pede ao container para fornecer uma transação para usar os métodos desta classe. Propagation.REQUIRED denota que a mesma transação é usada se uma estiver disponível quando vários métodos que exigem transação são aninhadas. O contêiner cria uma única transação física no banco de dados e várias transações lógicas para cada método aninhado. No entanto, se um método não conseguir concluir uma transação com sucesso, toda a transação física será rolou para trás. Uma das outras opções é Propagation.REQUIRES_NEW, em que uma nova transação física é criada para cada método. Existem outras opções que ajudam a ter um bom controle sobre o gerenciamento de transações.
A anotação @PersistenceContext informa ao contêiner para injetar uma instância de entitymanager no DAO. 
A classe implementa os métodos save e findbyPk que salvam e buscam os dados usando a instância de EntityManager injetada.
Agora definimos nossa Unidade de persistência no Persistence.xml que é colocado na pasta META-INF em src. Nós então mencione a classe cujas instâncias serão usadas Persisted. Para este exemplo, é a entidade Employee que criamos anteriormente.


persistence.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<persistence xmlns="https://java.sun.com/xml/ns/persistence"
version="1.0">
<persistence-unit name="jcgPersistence" transaction-type="RESOURCE_LOCAL" >
<class>com.jcg.bean.Employee</class>
</persistence-unit>
</persistence>

```

Agora configuramos o Spring Container usando o arquivo spring-configuration.xml.


spring-configuration.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="https://www.springframework.org/schema/beans"
xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" xmlns:aop="https://www. ←-
springframework.org/schema/aop"
xmlns:context="https://www.springframework.org/schema/context" xmlns:tx="https:// ←-
www.springframework.org/schema/tx"
xsi:schemaLocation="https://www.springframework.org/schema/beans
https://www.springframework.org/schema/beans/spring-beans-3.0.xsd
https://www.springframework.org/schema/aop
https://www.springframework.org/schema/aop/spring-aop-3.0.xsd
https://www.springframework.org/schema/context
https://www.springframework.org/schema/context/spring-context-3.0.xsd
https://www.springframework.org/schema/tx
https://www.springframework.org/schema/tx/spring-tx.xsd">
<context:component-scan base-package="com.jcg" />
<bean id="dataSource"
class="org.springframework.jdbc.datasource.DriverManagerDataSource">
<property name="driverClassName" value="com.mysql.jdbc.Driver" />
<property name="url" value="jdbc:mysql://localhost:3306/jcg" />
<property name="username" value="root" />
<property name="password" value="toor" />
</bean>
<bean id="jpaDialect" class="org.springframework.orm.jpa.vendor.HibernateJpaDialect ←-
" />
<bean id="entityManagerFactory"
class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
<property name="persistenceUnitName" value="jcgPersistence" />
<property name="dataSource" ref="dataSource" />
<property name="persistenceXmlLocation" value="META-INF/persistence.xml" />
<property name="jpaVendorAdapter" ref="jpaVendorAdapter" />
<property name="jpaDialect" ref="jpaDialect" />
<property name="jpaProperties">
<props>
<prop key="hibernate.hbm2ddl.auto">validate</prop>
<prop key="hibernate.dialect">org.hibernate.dialect.MySQL5Dialect</prop>
</props>
</property>
</bean>
<bean id="jpaVendorAdapter"
class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter">
</bean>
<bean id="txManager" class="org.springframework.orm.jpa.JpaTransactionManager">
<property name="entityManagerFactory" ref="entityManagerFactory" />
<property name="dataSource" ref="dataSource" />
<property name="jpaDialect" ref="jpaDialect" />
</bean>
<tx:annotation-driven transaction-manager="txManager" />
</beans>

```
Definimos os beans que precisamos no arquivo spring-configuration.xml. A fonte de dados contém as propriedades básicas de configuração, como URL, nome de usuário, senha e o nome de classe do driver JDBC. Criamos um EntityManagerFactory usando o LocalContainerEntityManagerFactoryBean. as propriedades são fornecidos como a fonte de dados, persistenceUnitName, persistenceUnitLocation, dialeto etc. A instância de EntityManager é injetado deste FactoryBean na instância EmployeeDaoImpl. A linha 51 no XML acima solicita que o contêiner Spring gerencie transações usando o contêiner Spring. A classe TransactionManagerProvider é a classe JpaTransactionManager. Agora que concluímos todo o trabalho duro, é hora de testar a configuração:
A classe SpringDataDemo extrai o EmployeeDaoImpl e tenta salvar uma instância de Employee na tabela de funcionários e recuperar a mesma instância do banco de dados.


SpringDataDemo.java

```
package com.jcg;
import java.sql.SQLException;
import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import com.jcg.bean.Employee;
import com.jcg.dao.EmployeeDao;
public class SpringDataDemo
{
public static void main(String[] args)
{
try
{
ApplicationContext context = new ClassPathXmlApplicationContext(" ←-
resources\\spring-configuration.xml");
//Fetch the DAO from Spring Bean Factory
EmployeeDao employeeDao = (EmployeeDao)context.getBean(" ←-
EmployeeDaoImpl");
Employee employee = new Employee("Employee123");
//employee.setEmployeeId("1");
//Save an employee Object using the configured Data source
employeeDao.save(employee);
System.out.println("Employee Saved with EmployeeId "+employee. ←-
getEmployeeId());
//find an object using Primary Key
Employee emp = employeeDao.findByPrimaryKey(employee.getEmployeeId ←-
());
System.out.println(emp);
//Close the ApplicationContext
((ConfigurableApplicationContext)context).close();
}
catch (BeansException | SQLException e)
{
e.printStackTrace();
}
}
}

```
