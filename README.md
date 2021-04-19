# Bloom_filter
## 1.项目描述：

​	实现一个简单的布隆过滤器

## 2.布隆过滤器简介:

布隆过滤器是一个非常神奇的数据结构，通过它我们可以非常方便地**判断一个给定数据是否存在于海量数据中**

## 3.特点：

- 有一定误差，如果判定在，小概率可能不在；
- 如果判定不在，那么一定不在；

## 4.常见使用场景：

- 网页黑名单系统
- 垃圾邮件过滤系统
- 爬虫的网址判重系统
- 解决redis缓存穿透问题

## 5.布隆过滤器的判断流程：

### 当一个元素加入布隆过滤器中的时候，会进行如下操作：

1.    使用布隆过滤器中的哈希函数对元素值进行计算，得到**几个哈希值**（有几个哈希函数得到几个哈希值）。
2.    根据得到的哈希值，在**位数组**中把对应**下标**的值置为 1。

### 需要判断一个元素是否存在于布隆过滤器的时候，会进行如下操作：

1. ​	对给定元素再次**进行相同的哈希计算**；
2. ​    得到值之后判断**位数组中的每个元素是否都为 1，如果值都为 1，那么说明这个值在布隆过滤器中**，如果存在一个值不为 1，说明该元素不在布隆过滤器中。

### 误判原因：

- 可能存在不同的字符串 哈希出来的位置相同（可以适当增加位数组大小或者调整我们的哈希函数来降低概率）

## 6.布隆过滤器简单实现：

- 如果使用布隆过滤器：那么需要提供允许的**误差率p**和**样本容量n**，算出布隆过过滤器的大小m，然后再通过m和n算出需要的几个hash函数，最后算出失误率q。

- 例如：n=100亿，p=0.01%，算出m=2000亿bit，需要25G，14个hash函数，误差率q = 0.006%

  

```java
import java.util.BitSet;
/**
* @ClassName MyBloomFilter
* @Description
* @Author wq
* @Date 
* @Version 1.0
**/
public class MyBloomFilter {
    /**
    * 位数组的大小
    */
    private static final int DEFAULT_SIZE = 2 << 24;
    /**
    * 通过这个数组可以创建 6 个不同的哈希函数
    */
    private static final int[] SEEDS = new int[]{3, 13, 46, 71, 91, 134};
    /**
    * 位数组。数组中的元素只能是 0 或者 1
    */
    private BitSet bits = new BitSet(DEFAULT_SIZE);
    /**
    * 存放包含 hash 函数的类的数组
    */
    private SimpleHash[] func = new SimpleHash[SEEDS.length];
    /**
    * 初始化多个包含 hash 函数的类的数组，每个类中的 hash 函数都不一样
    */
    public MyBloomFilter() {
        // 初始化多个不同的 Hash 函数
        for (int i = 0; i < SEEDS.length; i++) {
        	func[i] = new SimpleHash(DEFAULT_SIZE, SEEDS[i]);
        }
    }
    /**
    * 添加元素到位数组
    */
    public void add(Object value) {
        for (SimpleHash f : func) {
        	bits.set(f.hash(value), true);
        }
    }
    /**
    * 判断指定元素是否存在于位数组
    */
    public boolean contains(Object value) {
        boolean ret = true;
        for (SimpleHash f : func) {
            ret = ret && bits.get(f.hash(value));
        }
        return ret;
    }
    /**
    * 静态内部类。用于 hash 操作！
    */
    public static class SimpleHash {
        private int cap;
        private int seed;
        public SimpleHash(int cap, int seed) {
            this.cap = cap;
            this.seed = seed;
        }
        /**
        * 计算 hash 值
        */
        public int hash(Object value) {
            int h;
            return (value == null) ? 0 : Math.abs(seed * (cap - 1) & ((h = value.hashCode()) ^ (h >>> 16)));
        }
	}
}
```

