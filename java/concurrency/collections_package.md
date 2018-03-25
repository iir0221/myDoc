# Collections.synchronizedXXX
在 Collections 类中有多个静态方法，它们可以获取通过同步方法封装非同步集合而得到的集合：
```
public static Collection synchronizedCollention(Collection c)

public static List synchronizedList(list l)

public static Map synchronizedMap(Map m)

public static Set synchronizedSet(Set s)

public static SortedMap synchronizedSortedMap(SortedMap sm)

public static SortedSet synchronizedSortedSet(SortedSet ss)
```
```
synchronizedCollection
public static <T> Collection<T> synchronizedCollection(Collection<T> c)
返回指定 collection 支持的同步（线程安全的）collection。为了保证按顺序访问，必须通过返回的 collection 完成 所有对底层实现 collection 的访问。
在返回的 collection 上进行迭代时，用户必须手工在返回的 collection 上进行同步：

  Collection c = Collections.synchronizedCollection(myCollection);
     ...
  synchronized(c) {
      Iterator i = c.iterator(); // Must be in the synchronized block
      while (i.hasNext())
         foo(i.next());
  }
 
不遵从此建议将导致无法确定的行为。
返回的 collection 不会 将 hashCode 和 equals 操作传递给底层实现 collection，但这依赖于 Object 的 equals 和 hashCode 方法。在底层实现 collection 是一个 set 或一个列表的情况下，有必要遵守这些操作的协定。

如果指定 collection 是可序列化的，则返回的 collection 也将是可序列化的。

参数：
c - 被“包装”在同步 collection 中的 collection。
返回：
指定 collection 的同步视图。
```
```java
static class SynchronizedList<E>
    extends SynchronizedCollection<E>
    implements List<E> {
    private static final long serialVersionUID = -7754090372962971524L;

    final List<E> list;

    SynchronizedList(List<E> list) {
        super(list);
        this.list = list;
    }
    SynchronizedList(List<E> list, Object mutex) {
        super(list, mutex);
        this.list = list;
    }

    public boolean equals(Object o) {
        if (this == o)
            return true;
        synchronized (mutex) {return list.equals(o);}
    }
    public int hashCode() {
        synchronized (mutex) {return list.hashCode();}
    }

    public E get(int index) {
        synchronized (mutex) {return list.get(index);}
    }
    public E set(int index, E element) {
        synchronized (mutex) {return list.set(index, element);}
    }
    public void add(int index, E element) {
        synchronized (mutex) {list.add(index, element);}
    }
    public E remove(int index) {
        synchronized (mutex) {return list.remove(index);}
    }

    public int indexOf(Object o) {
        synchronized (mutex) {return list.indexOf(o);}
    }
    public int lastIndexOf(Object o) {
        synchronized (mutex) {return list.lastIndexOf(o);}
    }

    public boolean addAll(int index, Collection<? extends E> c) {
        synchronized (mutex) {return list.addAll(index, c);}
    }

    public ListIterator<E> listIterator() {
        return list.listIterator(); // Must be manually synched by user
    }

    public ListIterator<E> listIterator(int index) {
        return list.listIterator(index); // Must be manually synched by user
    }

    public List<E> subList(int fromIndex, int toIndex) {
        synchronized (mutex) {
            return new SynchronizedList<>(list.subList(fromIndex, toIndex),
                                        mutex);
        }
    }

    @Override
    public void replaceAll(UnaryOperator<E> operator) {
        synchronized (mutex) {list.replaceAll(operator);}
    }
    @Override
    public void sort(Comparator<? super E> c) {
        synchronized (mutex) {list.sort(c);}
    }

    /**
        * SynchronizedRandomAccessList instances are serialized as
        * SynchronizedList instances to allow them to be deserialized
        * in pre-1.4 JREs (which do not have SynchronizedRandomAccessList).
        * This method inverts the transformation.  As a beneficial
        * side-effect, it also grafts the RandomAccess marker onto
        * SynchronizedList instances that were serialized in pre-1.4 JREs.
        *
        * Note: Unfortunately, SynchronizedRandomAccessList instances
        * serialized in 1.4.1 and deserialized in 1.4 will become
        * SynchronizedList instances, as this method was missing in 1.4.
        */
    private Object readResolve() {
        return (list instanceof RandomAccess
                ? new SynchronizedRandomAccessList<>(list)
                : this);
    }
}
```