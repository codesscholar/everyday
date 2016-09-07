# Stream
Stream 就目前的使用场景来说更多的是针对集合（Collection）提供更多更便利的聚合操作。以前大部分需要利用遍历完成的工作，现在都提供了非常便利的接口。最重要的是Stream背后是并行化设计，所以效率有很大的提高。

我们从 获取对象 － 调用接口 － 返回结果 三步来详细学习一下这个类库。

## 简介
	1. 数据源 （如何初始化Stream）
	2. 数据转换
	3. 获取结果

案例 ： 我们初始化一部分数据，需求是按照年龄从大到小给男性排序，若是java8之前的做法大概是：

	1. 取出所有男性玩家赋值给新的集合
	2. 自定义一个comparator
	3. 排序
总体来说十几行代码是必须的。其中还不乏内部类这种生涩的语法，使用起来并不是特别方便。

利用Stream :

``` java

	List<Student> result = sourcedata.stream().	
							   filter(e -> e.getSex() == 'M').
							   sorted( (Student s1, Student s2) -> (s1.getAge() - (s2.getAge())) ).
							   // sorted( (s1, s2) -> (s1.getAge() - (s2.getAge())) ).
							   collect(Collectors.toList());
		
```
最终不过 5行代码。由此我们开始学习Stream。



### 数据源
所谓的数据源就是如何获取一个Stream对象，最常见的方式莫过于从集合Collection和数组Array中生成Stream:

	1. Collections.Stream()
	2. Arrays.stream(T array) or Stream.of(T array)

Stream 转换成其他数据结构：

	1. List<T> list = stream.collect(Collections.toList());
	
当然还有很多。	


### 数据操作
Stream 的背后是fork/join框架，所以支持并行化操作。

流的操作类型 分为两种：

* Intermediate :  **转换操作** 可以有很多，大多是过滤和映射，排序工作。特别注意的是此类操作并非立即开始执行，我们可以把此类操作理解成一种状态，它会一直保留到最后，知道终止操作时才会正在开始执行。这个很关键，如果简单分析一下上述的排序例子，我们会以为至少执行了两遍Iterator操作，filter一次，sort一次，其实并没有。这也是Stream性能高的原因之一。
> map, filter, distinct, peek, limit, skip, parrallel, sequential ...


* Terminal : **终止操作** 只会有一个，大多会返回数据，这种操作开始后就会正在遍历数据并进行所有的复合处理。
> forEach, toArray, reduce, collect, min, max ,count, anyMatch, findFirst...


### 细节API学习
具体请看详细代码。
``` java
package streams;

import java.util.ArrayList;
import java.util.Collections;
import java.util.Comparator;
import java.util.List;
import java.util.OptionalDouble;
import java.util.stream.Collectors;

/**
 * @author wangzhiping movingwzp@gmail.com
 * @date 201/08/19 15:20
 * @brief learning the new feature of java8 - stream
 *
 * 把函数式风格引入了java
 */
public class Stream {
	public static void main(String[] args) {
		List<Student> sourcedata = getData();
		
		// 需求1 ： M（男性） W（女性） 按照从小到大给男性排序
		// before java8
		List<Student> datanew = new ArrayList<Student>();
		for (Student s : sourcedata) {
			if (s.getSex() == 'M') {
				datanew.add(s);
			}
		}
		Collections.sort(datanew, new Comparator<Student>() {

			@Override
			public int compare(Student o1, Student o2) {
				// TODO Auto-generated method stub
				return o1.getAge() - o2.getAge();
			}
		});
		//showData(datanew);
		
		// java8 in stream
		List<Student> result = sourcedata.stream().	
							   filter(e -> e.getSex() == 'M').
							   sorted( (Student s1, Student s2) -> (s1.getAge() - (s2.getAge())) ).
							   // sorted( (s1, s2) -> (s1.getAge() - (s2.getAge())) ).
							   collect(Collectors.toList());
		//showData(result);
		
		// learning strem api in detail
		// problem 1 - 男性平均年龄
		OptionalDouble resultage  = sourcedata.stream().
									filter(e -> e.getSex() == 'M').
									mapToInt(e -> e.getAge()).
									average();
		System.out.println(resultage);
		
		// reduce :  提供一个初始种子和运算规则，依照运算规则，依次跟第一个，第二个。。。元素进行组合，返回结果 包括：sum max min
		int resultagetwo = sourcedata.stream().
						   filter(e -> e.getSex() == 'M').
						   mapToInt(e -> e.getAge()).
						   reduce(0, Integer::max);
		System.out.println(resultagetwo);
		
		// map : 获取所有人的名字大写
		List<String> resultname = sourcedata.stream().
						map(e -> e.getName().toUpperCase()).
						sorted().
						collect(Collectors.toList());
		System.out.println(resultname);	
		
		
		// forEach 遍历每一个元素并做相应的修改，并行状态下无法保证遍历的元素次序，多线程安全
		sourcedata.stream().forEach(e -> e.setName("new name"));
		showData(sourcedata);
	}
	private static void showData(List<Student> data) {
		for (Student s : data) {
			System.out.println(s);
		}
	}
	
	private static List<Student> getData() {
		List<Student> result = new ArrayList<Student>();
		result.add(new Student("wang1", 22, 'M'));
		result.add(new Student("wang2", 35, 'W'));
		result.add(new Student("wang3", 12, 'M'));
		result.add(new Student("wang4", 52, 'W'));
		result.add(new Student("zhang1", 35, 'M'));
		result.add(new Student("zhang2", 21, 'W'));
		result.add(new Student("zhang4", 34, 'M'));
		result.add(new Student("li3", 12, 'W'));
		result.add(new Student("li4", 23, 'M'));
		result.add(new Student("li9", 34, 'W'));
		result.add(new Student("li1", 16, 'M'));
		return result;
	}
}

```

github 代码链接: <https://github.com/codesscholar/java8/tree/master/src/streams> 




















 
	