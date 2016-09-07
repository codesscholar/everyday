# Lombok
在javaEE的开发过程中，不可避免的会出现很多的POJO（简单的java对象，一般不带有业务逻辑或者是持久逻辑），通常情况下做VO（value-project） 和 与数据库表对应的对象。

我们以StudentInfo类为例：包含一下几个属性。

	private String name;
	private int age;
	private float grade;
	
一般情况下

	1. 需要构造方法；
	2. 需要针对每个属性写相对应的getter 和 setter 方法；
	3. 如果在大型工程开发中，打印日志的话，还需要 重写 toString（）方法；
	4. 如果这个类可能出现在HashMap这种数据结构中的时候，还需要重写 hashCode() 和 equals()方法。

当然 Eclipse 这种IDE 有很便捷的一键生成按钮。只是即使这样，写完之后的类大概是这样：

```
package wzp.java.lombok;

public class StudentInfo {
	private String name;
	public StudentInfo(String name, int age) {
		super();
		this.name = name;
		this.age = age;
	}
	@Override
	public int hashCode() {
		final int prime = 31;
		int result = 1;
		result = prime * result + age;
		result = prime * result + ((name == null) ? 0 : name.hashCode());
		return result;
	}
	@Override
	public boolean equals(Object obj) {
		if (this == obj)
			return true;
		if (obj == null)
			return false;
		if (getClass() != obj.getClass())
			return false;
		StudentInfo other = (StudentInfo) obj;
		if (age != other.age)
			return false;
		if (name == null) {
			if (other.name != null)
				return false;
		} else if (!name.equals(other.name))
			return false;
		return true;
	}
	@Override
	public String toString() {
		return "StudentInfo [name=" + name + ", age=" + age + "]";
	}
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public int getAge() {
		return age;
	}
	public void setAge(int age) {
		this.age = age;
	}
	private int age;
}

```

## Lombok的使用
lombok 为我们提供了不多但是非常管用的几个注解，来解决这个问题：
	
	1. @Getter/@Setter : 注解类（覆盖所有属性）， 也可以注解单个属性
	2. @EqualsAndHashCode(exclude={"name", "age"}) : 生成 hashCode() 和 equals方法
	3. 如果注解在类上的话：提供 @ToString @EqualsAndHashCode @RequiredArgsConstructor，
	所有属性的 @Getter，以及非final 属性的@Setter.

类似的注解还有几个，具体可依查看官网：https://projectlombok.org/features/index.html
解释非常仔细，也有非常实用的DEMO。

下面看一下使用完Lombok之后的代码：

```
package wzp.java.lombok;

import lombok.Data;

@Data
public class StudentInfo {
	private String name;
	private int age;
}
```

## Lombok 的安装
在官网：https://projectlombok.org/ 上有一个视频，上面有很直观的说明。

如果你不想你的POJO看起来总是那么臃肿的话，Lombok是一个不错的选择，当然在大型项目中比较适用。