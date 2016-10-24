---
title: java 反射机制
tags: java
---
 
JAVA反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法；这种动态获取的信息以及动态调用对象的方法的功能称为java语言的反射机制。

Java反射机制主要提供了以下功能： 在运行时判断任意一个对象所属的类；在运行时构造任意一个类的对象；在运行时判断任意一个类所具有的成员变量和方法；在运行时调用任意一个对象的方法；生成动态代理。

1. **得到某个对象属性**
```
    public Object getProperty(Object owner, String fieldName) throws Exception {  
        Class ownerClass = owner.getClass();  
        Field field = ownerClass.getField(fieldName); //通过Class得到类声明的属性
        Object property = field.get(owner);  //通过对象得到该属性的实例，如果这个属性是非公有的，这里会报IllegalAccessException。
        return property;  
    }  
```
2. **得到某个类的静态属性**
```
    public Object getStaticProperty(String className, String fieldName)
             throws Exception {
        Class ownerClass = Class.forName(className);
        Field field = ownerClass.getField(fieldName);
        Object property = field.get(ownerClass); //因为该属性是静态的，所以直接从类的Class里取。
        return property;
     }
```
3. **执行某对象的方法**
```
public Object  Object invokeMethod(Object owner, String methodName, Object[] args)
             throws Exception {
        Class ownerClass = Class.forName(className);
        Class[] argsClass = new Class[args.length];
        for (int i = 0, j = args.length; i < j; i++) {  
         argsClass[i] = args[i].getClass();  
        }  
        Method method = ownerClass.getMethod(methodName,argsClass);  //通过methodName和参数的argsClass（方法中的参数类型集合）数组得到要执行的Method。
        return method.invoke(owner, args);//执行该Method.invoke方法的参数是执行这个方法的对象owner，和参数数组args，可以这么理解：owner对象中带有参数args的method方法。返回值是Object，也既是该方法的返回值。
     }
```
4. **执行某个类的静态方法**
```
public Object  Object invokeMethod(Object owner, String methodName, Object[] args)
             throws Exception {
        Class ownerClass = Class.forName(className);
        Class[] argsClass = new Class[args.length];
        for (int i = 0, j = args.length; i < j; i++) {  
         argsClass[i] = args[i].getClass();  
        }
        Method method = ownerClass.getMethod(methodName,argsClass);
        return method.invoke(null,args);//基本的原理和第3点相同，不同点是最后一行，invoke的一个参数是null，因为这是静态方法，不需要借助实例运行。
     }
```
5. **判断是否为某个类的实例**
```
public boolean isInstance(Object obj, Class cls) {  
     return cls.isInstance(obj);  
}
```
6. **得到数组中的某个元素**
```
public Object getByArray(Object array, int index) {  
     return Array.get(array,index);  
}
```

7. **Method.invoke**
```
public class InvokeTester {
    int sum;
    String msg;

    public int add(int param1, int param2) {
        return param1 + param2;
    }

    public String echo(String msg) {
        return "echo " + msg;
    }

    public int getSum() {
        return sum;
    }

    public void setSum(int sum) {
        this.sum = sum;
    }

    public String getMsg() {
        return msg;
    }

    public void setMsg(String msg) {
        this.msg = msg;
    }

    public static void main(String[] args) throws Exception {
        Class<InvokeTester> classType = InvokeTester.class;
        InvokeTester invokerTester = classType.newInstance();

        Method addMethod = classType.getMethod("add", int.class,
                int.class);
        //Method类的invoke(Object obj,Object args[])方法接收的参数必须为对象，
        //如果参数为基本类型数据，必须转换为相应的包装类型的对象。invoke()方法的返回值总是对象，
        //如果实际被调用的方法的返回类型是基本类型数据，那么invoke()方法会把它转换为相应的包装类型的对象，
        //再将其返回

        Object result = addMethod.invoke(invokerTester, 100, 200);
        System.out.println(result);

        Method echoMethod = classType.getMethod("echo", String.class);
        result = echoMethod.invoke(invokerTester, "hello");
        System.out.println(result);

        Method setSumMethod = classType.getMethod("setSum", int.class);
        setSumMethod.invoke(invokerTester, 12);
        System.out.println(invokerTester.getSum());

        Method setMsgMethod = classType.getMethod("setMsg", String.class);
        setMsgMethod.invoke(invokerTester, "Hello world!");
        System.out.println(invokerTester.getMsg());
    }
}
```