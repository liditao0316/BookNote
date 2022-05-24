## 源码分析

```java
 		/*
 		传递一个comparator参数构造TreeMap类
 		*/
		public TreeMap(Comparator<? super K> comparator) {
        this.comparator = comparator;
    }
		/*
    Associates the specified value with the specified key in this map. If the map previously contained a 						mapping for the key, the old value is replaced.
    Params:
      key – key with which the specified value is to be associated
      value – value to be associated with the specified key
    Returns:
    	the previous value associated with key, or null if there was no mapping for key. (A null return can 							also indicate that the map previously associated null with key.)
    Throws:
    	ClassCastException – if the specified key cannot be compared with the keys currently in the map
    	NullPointerException – if the specified key is null and this map uses natural ordering, or its comparator 				does not permit null keys
 			*/
			public V put(K key, V value) {
        Entry<K,V> t = root;
        if (t == null) {
            compare(key, key); // type (and possibly null) check

            root = new Entry<>(key, value, null);
            size = 1;
            modCount++;
            return null;
        }
        int cmp;
        Entry<K,V> parent;
        // split comparator and comparable paths
        Comparator<? super K> cpr = comparator;
        if (cpr != null) {
            do {
                parent = t;
                cmp = cpr.compare(key, t.key);
                if (cmp < 0)
                    t = t.left;
                else if (cmp > 0)
                    t = t.right;
                else
                    return t.setValue(value);
            } while (t != null);
        }
        else {
            if (key == null)
                throw new NullPointerException();
            @SuppressWarnings("unchecked")
                Comparable<? super K> k = (Comparable<? super K>) key;
            do {
                parent = t;
                cmp = k.compareTo(t.key);
                if (cmp < 0)
                    t = t.left;
                else if (cmp > 0)
                    t = t.right;
                else
                    return t.setValue(value);
            } while (t != null);
        }
        Entry<K,V> e = new Entry<>(key, value, parent);
        if (cmp < 0)
            parent.left = e;
        else
            parent.right = e;
        fixAfterInsertion(e);
        size++;
        modCount++;
        return null;
    }
```

### 注意事项：

#### 1、TreeMap添加元素的机制

- 因此，TreeMap元素的重复性问题，还是要看程序员创建的comparator是如何实现的

```java
				/*
				* 元素的添加，关键码的判别机制
				* 1、有比较器comparator
				* 2、无比较器
				* 无论是有比较器，还是无比较器，关键还是看返回的值大小；如果返回值是0，则认为存在相同的key，返回 t.setValue(value);
				*/
				Comparator<? super K> cpr = comparator;
        if (cpr != null) {
            do {
                parent = t;
                cmp = cpr.compare(key, t.key);
                if (cmp < 0)
                    t = t.left;
                else if (cmp > 0)
                    t = t.right;
                else
                    return t.setValue(value);
            } while (t != null);
        }
        else {
            if (key == null)
                throw new NullPointerException();
            @SuppressWarnings("unchecked")
                Comparable<? super K> k = (Comparable<? super K>) key;
            do {
                parent = t;
                cmp = k.compareTo(t.key);
                if (cmp < 0)
                    t = t.left;
                else if (cmp > 0)
                    t = t.right;
                else
                    return t.setValue(value);
            } while (t != null);
        }
```

#### 2、TreeMap的key不能为null