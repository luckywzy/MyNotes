## Mockito

 参考链接：https://www.cnblogs.com/Ming8006/p/6297333.html



### mock可以做哪些事情

##### 1.验证行为

```java
    @Test
    public void verify_behaviour() {
        List mock = mock(List.class);
        //  使用mock对象执行操作
        mock.add(1);
        mock.clear();
        // 验证操作是否执行
        verify(mock).clear();
        verify(mock).add(1);
    }
```

**注意**：verify验证操作是否执行，并不关心顺序；但是如果需要顺序上的验证，那么就需要每操作一步就验证一步

##### 2.模拟期望的结果

```java
    @Test
    public void when_andReturn() {

        Iterator mock = mock(Iterator.class);
        //模拟期望的结果
        when(mock.next()).thenReturn("first").thenReturn("2_n_time").thenReturn("haha");
        //实际执行的结果
        String actual = mock.next() + "\n" + mock.next() + "\n"+"haha";
        //断言
        assertEquals("first\n2_n_time\nhaha", actual);
    }
```

##### 3.RETURNS_SMART_NULLS 和 RETURNS_DEEP_STUBS

```java
    @Test
    public void returnSmartNull(){
        List mock = mock(List.class,RETURNS_SMART_NULLS);
        // 有些方法返回值没有默认值，返回null再操作，可能会抛出NullPointerException，
        // 如果通过RETURNS_SMART_NULLS参数创建的mock对象在没有调用stubbed方法时会返回SmartNull
        System.out.println(mock.get(0));
    }
```

**说明**：返回类型是String，会返回"";是int，会返回0；是List，会返回空的List。另外，在控制台窗口中可以看到SmartNull的友好提示。

另外一个参数：RETURNS_DEEP_STUBS：创建mock对象时的备选参数

##### 4.模拟方法体抛出异常

```java
    @Test(expected = RuntimeException.class)
    public void exceptionTest(){
        List mock = mock(List.class);
        // 模拟抛出异常
        doThrow(new RuntimeException()).when(mock).add(1);
        mock.add(1);
    }
```

##### 5.使用注解快速模拟

```java
    @Mock
    public Map mockMap;
    @Before
    public void setData(){
        mockMap = new HashMap();
    }
```

**说明**：使用Mock注解来创建对象，不过需要自己来new实现类；并且会有很多限制，使用时需要注意

##### 6.参数匹配

```java
    @Test
    public void argsMatch(){
        Map map = mock(Map.class);
        //模拟当 map put 时返回 1
        when(map.put(anyString(),anyInt())).thenReturn(1);
        map.put("2",2);
        Integer rt= 1;
        verify(map).put("2",2);
        //断言put 的返回值
        assertEquals(rt, map.put("2",2));
    }
```

**说明**：anyInt(),anyString() ...可以匹配任意参数



### 踩坑记录

1. 打桩失败；在Test层可以debug看见打桩，但是进入被测函数就发现打桩失败了

   原因：被测试类有一个final字段，并且使用了构造函数传入了此final字段；

   解决：去掉final修饰，将构造函数改为无参构造函数，使用Java注解进行注入