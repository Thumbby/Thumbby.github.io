---
layout: post
title:  "实习日记59(工作任务)"
date:   2021-10-19
categories: jekyll update
---

## Day59

- Adobe acrobat过期了，去学校软件中心重新搞一个破解的

- 我现在算是真正懂了，相比于技术，业务问题才是最难的。

- Java使用PriorityQueue实现最大/小堆

  - **一个基于优先级堆的无界优先级队列。**

    实际上是一个堆（不指定Comparator时默认为最小堆），通过传入自定义的Comparator函数可以实现大顶堆。

    ```java
    PriorityQueue<Integer> minHeap = new PriorityQueue<Integer>(); //小顶堆，默认容量为11
    PriorityQueue<Integer> maxHeap = new PriorityQueue<Integer>(11,new Comparator<Integer>(){ //大顶堆，容量11
        @Override
        public int compare(Integer i1,Integer i2){
            return i2-i1;
        }
    });
    ```

  - PriorityQueue的常用方法有：poll(),offer(Object),size(),peek()等。

    - 插入方法（offer()、poll()、remove() 、add() 方法）时间复杂度为O(log(n)) ；
    - remove(Object) 和 contains(Object) 时间复杂度为O(n)；
    - 检索方法（peek、element 和 size）时间复杂度为常量。

  -  **PriorityQueue的API文档说明：**

    | 构造方法摘要                                                 |
    | ------------------------------------------------------------ |
    | `**PriorityQueue**()`      使用默认的初始容量（11）创建一个 `PriorityQueue`，并根据其自然顺序对元素进行排序。 |
    | `**PriorityQueue**(Collection<? extends E> c)`      创建包含指定 collection 中元素的 `PriorityQueue`。 |
    | `**PriorityQueue**(int initialCapacity)`      使用指定的初始容量创建一个 `PriorityQueue`，并根据其自然顺序对元素进行排序。 |
    | `**PriorityQueue**(int initialCapacity, Comparator<? super E> comparator)`      使用指定的初始容量创建一个 `PriorityQueue`，并根据指定的比较器对元素进行排序。 |
    | `**PriorityQueue**(PriorityQueue<? extends E> c)`      创建包含指定优先级队列元素的 `PriorityQueue`。 |
    | `**PriorityQueue**(SortedSet<? extends E> c)`      创建包含指定有序 set 元素的 `PriorityQueue`。 |

    | 方法摘要                 |                                                              |
    | ------------------------ | ------------------------------------------------------------ |
    | ` boolean`               | `**add**(E e)`      将指定的元素插入此优先级队列。           |
    | ` void`                  | `**clear**()`      从此优先级队列中移除所有元素。            |
    | ` Comparator<? super E>` | `**comparator**()`      返回用来对此队列中的元素进行排序的比较器；如果此队列根据其元素的自然顺序进行排序，则返回 `null`。 |
    | ` boolean`               | `**contains**(Object o)`      如果此队列包含指定的元素，则返回 `true`。 |
    | ` Iterator<E>`           | `**iterator**()`      返回在此队列中的元素上进行迭代的迭代器。 |
    | ` boolean`               | `**offer**(E e)`      将指定的元素插入此优先级队列。         |
    | ` E`                     | `**peek**()`      获取但不移除此队列的头；如果此队列为空，则返回 `null`。 |
    | ` E`                     | `**poll**()`      获取并移除此队列的头，如果此队列为空，则返回 `null`。 |
    | ` boolean`               | `**remove**(Object o)`      从此队列中移除指定元素的单个实例（如果存在）。 |
    | ` int`                   | `**size**()`      返回此 collection 中的元素数。             |
    | ` Object[]`              | `**toArray**()`      返回一个包含此队列所有元素的数组。      |
    | `<T> T[]`                | `**toArray**(T[] a)`      返回一个包含此队列所有元素的数组；返回数组的运行时类型是指定数组的类型。 |

    | **从类 java.util.AbstractQueue 继承的方法** |
    | ------------------------------------------- |
    | `addAll, element, remove`                   |

    | **从类 java.util.AbstractCollection 继承的方法**       |
    | ------------------------------------------------------ |
    | `containsAll, isEmpty, removeAll, retainAll, toString` |

    | **从类 java.lang.Object 继承的方法**                         |
    | ------------------------------------------------------------ |
    | `clone, equals, finalize, getClass, hashCode, notify, notifyAll, wait, wait, wait` |

    | **从接口 java.util.Collection 继承的方法**                   |
    | ------------------------------------------------------------ |
    | `containsAll, equals, hashCode, isEmpty, removeAll, retainAll` |

