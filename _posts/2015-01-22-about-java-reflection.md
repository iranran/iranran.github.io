---
layout: post
title: java反射相关知识
categories:
- Java基础知识
feature_image: "https://picsum.photos/2560/600?image=570"
---

JAVA反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法。对于任意一个对象，都能够调用它的任意一个方法和属性。这种动态获取的信息以及动态调用对象的方法的功能称为java语言的反射机制。

* **涉及到的类**
       java.lang.Class 、java.lang.reflect 包中相关的类，包括Constructor、Field、Method、Modifier
获取Class的方法
      Class 类十分特殊，它和一般类一样继承自Object。其实体用以表达Java程序运行时的classes和interfaces，也用来表达enum、array、primitive Java types（boolean, byte, char, short, int, long, float, double）以及关键词void。
* **获取类名的方法**
 每个类都继承于Object，通过Object的getClass方法获取，如new String().getClass(),这种用法要首先知道用的是哪个类再获取Class
 利用Class.forName(className),这个是最常用的，可以通过类的全名获取类，之后再进行其它的反射操作。所有的反射操作都是基于Class来操作的。
```java
    @Test
	public void testGetClass() throws ClassNotFoundException{
		Class<?> s = Class.forName("proxyandreflection.Person");
		assertEquals(s.getName(),Person.class.getName());
		
		Person person = new Person();
		assertEquals(s.getName(),person.getClass().getName());
	}
	
	@Test(expected = RuntimeException.class)
	public void testGetClassException(){
		try {
			Class.forName("Person");
		} catch (ClassNotFoundException e) {
			throw new RuntimeException(e);
		}
	}
 ```
 ```
getFields()
getMethods()
getField(String)
getDeclaredField()
getDeclaredFields()
getMethods()
getDeclaredMethods()
getMethod(String, Class<?>...)
getDeclaredMethod(String, Class<?>...)
 ```
图中的方法都是反射编程中会用到的，其中每个方法基本上都有两个实现，有Declared及没有Declared的，如getFields getDeclaredFields,区别就是没有Declared的方法，只能获取public修饰的相关内容，而有Declared的方法则可以获取所有的相关内容不只是public的
 ```
	/**
	 * 测试反射方法 getFields与getDeclaredFields的区别，其它Methods、Constructor也一样
	 * @throws ClassNotFoundException
	 */
	@Test
	public void test4Declared() throws ClassNotFoundException{
		Class<?> s = Class.forName("proxyandreflection.Person");
		Field[] fields1 = s.getDeclaredFields();
		Field[] fields2 = s.getFields();
		assertNotEquals(fields1.length,fields2.length);
		
		for(Field f1 : fields1){
			System.out.print(Modifier.toString(f1.getModifiers())+" "+f1.getName()+";");
		}
		System.out.println();
		
		for(Field f2 : fields2){
			System.out.print(Modifier.toString(f2.getModifiers())+" "+f2.getName()+";");
		}
	}
 ```
* **对类进行实例化**
对于无参数构造函数直接调用s.newInstance()即可
对于有参数的构造函数则要先调用getConstructor生成构造器，然后再操作
 ```
@Test
	public void testInstance() throws Exception{
		Class<?> s = Class.forName("proxyandreflection.Person");
		assertTrue(s.newInstance() instanceof Person);
	}
	
	@Test
	public void testConstructor() throws Exception{
		Class<?> s = Class.forName("interview.proxyandreflection.Person");
		Constructor<?> c = s.getConstructor(String.class);
		Person person = (Person)c.newInstance("jackie");
		assertEquals(person.getName(),"jackie");
	}
 ```

* **动态执行方法**
 ```
@Test
	public void testMethodInvoke() throws Exception{
		Class<?> s = Class.forName("interview.proxyandreflection.Person");
		Object obj = s.newInstance();
		Method setmethod = s.getDeclaredMethod("setMobileno", String.class);
		Method getmethod = s.getDeclaredMethod("getMobileno");
		setmethod.invoke(obj, "jackie");
		assertEquals(getmethod.invoke(obj),"jackie");
	}
 ```
* **应用场景**
框架类应用为了适应各种需求，要动态处理大量的使用反射。

* **缺点**
动态处理类、方法等的调用，难于跟踪调试
反射包括了一些动态类型，所以JVM无法对这些代码进行优化。因此，反射操作的效率要比那些非反射操作低得多。
我们应该避免在经常被执行的代码或对性能要求很高的程序中使用反射。
代码执行一些在正常情况下不被允许的操作（比如访问私有的属性和方法），所以使用反射可能会导致意料之外的副作用，降低可移植性。

当然也不必过分担心，当反射代码被执行多次后，jvm会把它生成本地代码。