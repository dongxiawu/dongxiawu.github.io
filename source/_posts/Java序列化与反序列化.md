---
title: Java序列化与反序列化
date: 2018-05-15 14:10:41
tags:
- Java
- Serializable
- Externalizable
layout:
updated:
comments: true
categories:
- Java
permalink:
---
{% asset_img Java序列化与反序列化.jpg %}

> 版权声明：本文为 *冬夏* 原创文章，可以随意转载，但请注明出处。

<!-- more -->

# 前言 #
Java 的序列化相信大家都不陌生。当我们需要讲对象存储起来或进行传输时，就需要对 Java 对象进行序列化，那么在序列化过程中有什么需要注意的呢？

# Serializable #

## 基础用法 ##

当我们要序列化一个类时，可以选择实现 *Serializable* 接口，注意，这是一个 *标记接口*，也就是说这个接口并没有任何方法或者成员变量，仅仅告诉 JVM 该类可以被序列化。

例：

    public class Animal implements Serializable{
      public int age;

      public Animal(int age) {
        this.age = age;
      }

      @Override
      public String toString() {
        return "Animal [age=" + age + "]";
      }
    }

此时，该类就可以被存储或者传输了。

例：

    public class Main{
        public static final void main(String[] args) {

            ObjectOutputStream objectOutputStream = null;
            try {
                Animal animal = new Animal(2);
                System.out.println(animal);
                objectOutputStream = new ObjectOutputStream(
                        new FileOutputStream(new File("file1")));
                objectOutputStream.writeObject(animal);
            } catch (FileNotFoundException e) {
                e.printStackTrace();
            } catch (IOException e) {
                e.printStackTrace();
            }finally {
                try {
                    if (objectOutputStream != null) {
                        objectOutputStream.flush();
                        objectOutputStream.close();
                    }
                } catch (IOException e) {
                    e.printStackTrace();
                }

            }

            ObjectInputStream objectInputStream = null;
            try {
                objectInputStream = new ObjectInputStream(
                        new FileInputStream(new File("file1")));
                Object object = objectInputStream.readObject();
                System.out.println(object);
            } catch (FileNotFoundException e) {
                e.printStackTrace();
            } catch (ClassNotFoundException e) {
                e.printStackTrace();
            } catch (IOException e) {
                e.printStackTrace();
            }finally {
                try {
                    if (objectInputStream != null) {
                        objectInputStream.close();
                    }
                } catch (IOException e) {
                    e.printStackTrace();
                }

            }

        }

    }

输出结果：

    Animal [age=2]
    Animal [age=2]

可以看到，成功的实现了对象的保存与恢复，有的时候，我们并不希望所有成员变量都进行保存，比如有一些敏感信息等。这时候就可以将不保存的成员变量声明为 *transient*,如：`transient public int age`，这样在序列化的时候就不会保存该成员变量，相应的反序列化的时候该成员变量将不会被赋值。

## 自定义序列化逻辑 ##
默认情况下，JVM会自动帮我们做序列化和反序列化的工作，但有的时候我们想自己在序列化和反序列化的时候添加上自己的逻辑，如加解密等，这就涉及到了以下的方法：

* private void writeObject(java.io.ObjectOutputStream out) throws IOException;
* private void readObject(java.io.ObjectInputStream in) throws IOException, ClassNotFoundException;
* private void readObjectNoData() throws ObjectStreamException;

其中，writeObject()方法 用于保存对象的状态以便 readObject()方法进行恢复，可以调用out.defaultWriteObject 进行默认保存。写入的对象可以是有效的成员变量或者DataOutput中支持的基本类型


readObject()方法用于从数据流中恢复类的非静态和非瞬时成员变量，in.defaultReadObject()为默认的恢复方法。

*注意* readObject()方法中的恢复顺序必须与与writeObject()方法中的保存顺序一致。

以上的两个方法均可以不关注属于父类或子类的成员变量。

readObjectNoData()方法比较特殊，其作用为当序列化流没有将给定的类认定为反序列化的对象的超类时，该方法可以用于初始化一个反序列化对象。其原理为调用反序列化对象的无参构造函数，并进行对应初始化。这就要求反序列化对象必须有无参构造函数。

这种情况在当接收方与发送方的反序列化类版本不同，*并且* 接收方的类继承了某些发送方没有继承的类(而没有改动类的内容);或者序列化流被篡改了，变得不完整的时候会发生。

例子：

    public class Main{
        public static final void main(String[] args) {

            ObjectOutputStream objectOutputStream = null;
            try {
                Persion persion = new Persion("jack");
                System.out.println(persion);
                objectOutputStream = new ObjectOutputStream(
                        new FileOutputStream(new File("file1")));
                objectOutputStream.writeObject(persion);
            } catch (FileNotFoundException e) {
                e.printStackTrace();
            } catch (IOException e) {
                e.printStackTrace();
            }finally {
                try {
                    if (objectOutputStream != null) {
                        objectOutputStream.flush();
                        objectOutputStream.close();
                    }
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
    public class Persion implements Serializable {
        public String name;

        public Persion() {

        }

        public Persion (String name) {
            this.name = name;
        }

        @Override
        public String toString() {
            return "Persion [" + "name=" + name +"]";
        }
    }

先将 Person 对象保存起来，然后修改 Person 类，使其继承 Animal 类，并在 Animal 类中实现 readObjectNoData() 方法，最后尝试恢复对象。

    public class Persion extends Animal implements Serializable {
        public String name;

        public Persion() {
        }

        public Persion (String name) {
            this.name = name;
        }

        @Override
        public String toString() {
            return "Persion [" + "name=" + name + ", age=" + age +"]";
        }
    }
    public class Animal implements Serializable{
        transient public int age;

        public Animal() {
        }

        public Animal(int age) {
            this.age = age;
        }

        private void readObjectNoData() throws ObjectStreamException {
            this.age = 20;
        }

        @Override
        public String toString() {
            return "Animal [age=" + age + "]";
        }
    }
    public class Main{
        public static final void main(String[] args) {
            ObjectInputStream objectInputStream = null;
            try {
                objectInputStream = new ObjectInputStream(
                        new FileInputStream(new File("file1")));
                Object object = objectInputStream.readObject();
                System.out.println(object);
            } catch (FileNotFoundException e) {
                e.printStackTrace();
            } catch (ClassNotFoundException e) {
                e.printStackTrace();
            } catch (IOException e) {
                e.printStackTrace();
            }finally {
                try {
                    if (objectInputStream != null) {
                        objectInputStream.close();
                    }
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }

可以发现 Animal 类的 readObjectNoData() 方法被调用了，输出结果为：`Persion [name=jack, age=20]`

## 替换序列化对象 ##
有的时候，我们想将其他对象而不是当前对象进行序列化保存或传输，比如代理类的情况，这个时候就可以通过实现
`Object writeReplace() throws ObjectStreamException`
方法达到，该方法可以是任何访问属性，子类也遵循访问规则。

值得提出的是，反序列化的对象和原来的对象并不是同一个对象，如果这个时候对两个对象进行 `==` 测试，则结果为 `false`。有的时候，我们希望反序列化的对象和原来的对象是同一个对象，比如在单例模式的时候，这个时候可以通过实现
`Object readResolve() throws ObjectStreamException;`方法进行反序列化对象替换。该方法可以是任何访问属性。

例：

    public class Animal implements Serializable{
        transient public int age;

        public Animal() {
        }

        public Animal(int age) {
            this.age = age;
        }

        public Object readResolve() throws ObjectStreamException {
            return new Animal(3);
        }

        @Override
        public String toString() {
            return "Animal [age=" + age + "]";
        }
    }

    public class Main{
        public static final void main(String[] args) {

            ObjectOutputStream objectOutputStream = null;
            try {
                Animal animal = new Animal(2);
                System.out.println(animal);
                objectOutputStream = new ObjectOutputStream(
                        new FileOutputStream(new File("file1")));
                objectOutputStream.writeObject(animal);
            } catch (FileNotFoundException e) {
                e.printStackTrace();
            } catch (IOException e) {
                e.printStackTrace();
            }finally {
                try {
                    if (objectOutputStream != null) {
                        objectOutputStream.flush();
                        objectOutputStream.close();
                    }
                } catch (IOException e) {
                    e.printStackTrace();
                }

            }

            ObjectInputStream objectInputStream = null;
            try {
                objectInputStream = new ObjectInputStream(
                        new FileInputStream(new File("file1")));
                Object object = objectInputStream.readObject();
                System.out.println(object);
            } catch (FileNotFoundException e) {
                e.printStackTrace();
            } catch (ClassNotFoundException e) {
                e.printStackTrace();
            } catch (IOException e) {
                e.printStackTrace();
            }finally {
                try {
                    if (objectInputStream != null) {
                        objectInputStream.close();
                    }
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }

此时的输出结果为：

    Animal [age=2]
    Animal [age=3]

## serialVersionUID ##
序列化运行时分配给每一个序列化类一个版本号，称为serialVersionUID，
在反序列化过程中使用该版本号来验证序列化对象的发送者和接收者是否已加载该对象的与序列化相容的类。
如果接收者为与对应的发送者类具有不同serialVersionUID的对象加载类，则反序列化将导致InvalidClassException异常。
一个可序列化的类可以通过声明一个名为serialVersionUID的字段声明自己的serialVersionUID，该字段必须是static final long
类型，可以是任何访问属性。

如果可序列化类没有显式声明serialVersionUID，则序列化运行时将基于该类的各个方面计算该类的默认serialVersionUID值，
如Java（TM）对象序列化规范中所述。然而，强烈建议所有可序列化的类显式声明serialVersionUID值，因为默认的serialVersionUID计算对类详细信息高度敏感，
可能因编译器实现而异，因此可能会导致在反序列化过程中抛出InvalidClassException。
因此，要确保跨不同的java编译器实现保持一致的serialVersionUID值，可序列化的类必须声明显式的serialVersionUID值。
还强烈建议显式serialVersionUID声明尽可能使用 *private* 修饰符，因为这些声明仅适用于立即声明的类
serialVersionUID字段作为继承成员是无用的。
数组类无法声明显式的serialVersionUID，因此它们始终具有默认的计算值，
但是对于数组类而言，不需要匹配serialVersionUID值。


# Externalizable #
序列化的接口除了 Serializable 还有 Externalizable，二者的不同在于 Externalizable 有以下的两个方法：

* void writeExternal(ObjectOutput out) throws IOException;
* void readExternal(ObjectInput in) throws IOException, ClassNotFoundException;

通过实现 `Externalizable` 接口实现序列化的类除了需要控制该类成员变量的的保存与恢复，还需要控制其父类成员变量的保存与恢复。 其通过上述的两个方法实现。

例子：

    public class Animal implements Serializable{
        transient public int age;

        public Animal() {
        }

        public Animal(int age) {
            this.age = age;
        }

        @Override
        public String toString() {
            return "Animal [age=" + age + "]";
        }
    }

    public class Persion extends Animal implements Externalizable {
        public String name;

        public Persion() {

        }

        public Persion (String name, int age) {
            this.name = name;
            this.age = age;
        }

        @Override
        public void writeExternal(ObjectOutput out) throws IOException {
            out.writeObject(name);
        }

        @Override
        public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException{
            name = (String )in.readObject();
        }

        @Override
        public String toString() {
            return "Persion [" + "name=" + name + ", age=" + age +"]";
        }
    }

    public class Main{
        public static final void main(String[] args) {

            ObjectOutputStream objectOutputStream = null;
            try {
                Persion persion = new Persion("Jack", 25);
                System.out.println(persion);
                objectOutputStream = new ObjectOutputStream(
                        new FileOutputStream(new File("file1")));
                objectOutputStream.writeObject(persion);
            } catch (FileNotFoundException e) {
                e.printStackTrace();
            } catch (IOException e) {
                e.printStackTrace();
            }finally {
                try {
                    if (objectOutputStream != null) {
                        objectOutputStream.flush();
                        objectOutputStream.close();
                    }
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }

            ObjectInputStream objectInputStream = null;
            try {
                objectInputStream = new ObjectInputStream(
                        new FileInputStream(new File("file1")));
                Object object = objectInputStream.readObject();
                System.out.println(object);
            } catch (FileNotFoundException e) {
                e.printStackTrace();
            } catch (ClassNotFoundException e) {
                e.printStackTrace();
            } catch (IOException e) {
                e.printStackTrace();
            }finally {
                try {
                    if (objectInputStream != null) {
                        objectInputStream.close();
                    }
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }

其输出结果为：

    Persion [name=Jack, age=25]
    Persion [name=Jack, age=0]

故表明通过实现 `Externalizable` 接口实现序列化的类除了需要控制该类成员变量的的保存与恢复，还需要控制其父类成员变量的保存与恢复。
