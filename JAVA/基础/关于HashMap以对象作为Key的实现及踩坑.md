# 关于HashMap以对象作为Key的实现及踩坑

[TOC]



## 1 场景

今天遇到了这样一串代码

![image-20211123001950011](https://cyzblog.oss-cn-beijing.aliyuncs.com/macimg/image-20211123001950011.png)

对于画圈的部分，看起来很奇怪，我一开始想改为：

```java
result.get(groupPath).add(indexes);
groupPath.addMeasurements(exactPath.getMeasurementList());
groupPath.addSchemas(exactPath.getSchemaList());
```

但是却发生了大量的空指针异常

## 2 原因

查看源码后发现，HashMap以对象作为Key的时候（假设我们重写了hashcode以及equals方法），他在put和get的时候对于key部分会处理：

* `hash(key)`
* `key`

![image-20211123002248654](https://cyzblog.oss-cn-beijing.aliyuncs.com/macimg/image-20211123002248654.png)

![image-20211123002303759](https://cyzblog.oss-cn-beijing.aliyuncs.com/macimg/image-20211123002303759.png)

换言之，在执行put的时候，HashMap会保存当前key对象的**hash值**和**引用**，在`get(key)`的时候，会根据执行get时刻的key对象hash值进行查找，再根据引用判断相等。

## 3 一个实验

```java
import java.util.*;

public class HashMapTest {
    public static void main(String[] args) {
        Map<MyObject, List> map = new HashMap<>();
        MyObject object = new MyObject("a");
        map.put(object,new ArrayList());
        object.setName("b");
        System.out.println(map.containsKey(object));
        for (MyObject myObject : map.keySet()) {
            System.out.println(myObject.hashCode());
        }
        System.out.println(object.hashCode());
        MyObject object1 = new MyObject("a");
        System.out.println(map.containsKey(object1));
        System.out.println(object1.hashCode());
        object.setName("a");
        System.out.println(map.containsKey(object));
    }

    public static class MyObject{
        String name;
        MyObject (String name){
            this.name = name;
        }

        public void setName(String name) {
            this.name = name;
        }

        @Override
        public boolean equals(Object o) {
            if (this == o) return true;
            if (o == null || getClass() != o.getClass()) return false;
            MyObject myObject = (MyObject) o;
            return Objects.equals(name, myObject.name);
        }

        @Override
        public int hashCode() {
            return Objects.hash(name);
        }
    }
}
```

执行结果为：

```
false
129
129
false
128
true
```

