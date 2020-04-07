#### 4月7日 早

遇见你，我变得很低很低，一直低到尘埃里去，但我的心是欢喜的，并且在那里开出一朵花来。

#### LRU算法是什么？
1. Least Recently Used，最近最久未使用法。
2. 源自一个理论：最近使用的页面数据会在未来一段时间内仍然被使用，已经很久没有使用的数据页面很可能在未来较长时间内仍然不会被使用。
3. 所以存在一个缓存淘汰机制：每次内存中找到最久未使用的数据然后置换出去，从而存入新的数据。
4. 主要指标: 使用时间；附加指标：使用次数。

#### LRU算法实现
1. 利用双向链表实现。基于利用先进先出（FIFO），最近被放入的数据会最早被获取。其中主要涉及到添加、访问、修改、删除操作。
2. 添加：如果是新元素，直接放在链表头上面，其他的元素顺序往下移动；
3. 访问：在头节点的可以不用管，如果是在中间位置或者尾巴，就要将数据移动到头节点；
4. 修改：修改原值之后，再将数据移动到头部
5. 删除：直接删除，其他元素顺序移动；

#### 代码实现（线程安全）
```
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.atomic.AtomicInteger;
import java.util.concurrent.atomic.AtomicLong;
import java.util.concurrent.locks.ReentrantLock;

public class ConcurrentLRUCache<K,V> {

    private final ConcurrentHashMap<K, CacheEntry<K,V>> map;
    private final int upperWaterMark, lowerWaterMark;
    private final int acceptableWaterMark;
    private ReentrantLock markAndSweepLock = new ReentrantLock(true);
    private Stats stats = new Stats();
    private volatile boolean isCleaning = false;
    private long oldestEntry;

    public ConcurrentLRUCache(int size, int lowerWaterMark) {
        this.upperWaterMark = size;
        this.lowerWaterMark = lowerWaterMark;
        this.acceptableWaterMark = (int) Math.floor((lowerWaterMark + size) / 2);
        this.map = new ConcurrentHashMap<>((int) Math.ceil(0.75 * size));
    }

    public V get(K key) {
        CacheEntry<K,V> cacheEntry = map.get(key);
        if (cacheEntry == null) {
            return null;
        }
        return cacheEntry.value;
    }

    public V put(K key, V val) {
        if (val == null) return null;
        CacheEntry<K,V> cacheEntry = new CacheEntry<>(key, val, stats.accessCounter.incrementAndGet());
        CacheEntry<K,V> oldEntry = map.put(key, cacheEntry);
        if (oldEntry == null) {
            int currentSize = stats.size.incrementAndGet();
            if (currentSize > upperWaterMark && !isCleaning) {
                new Thread(this::markAndSweep).start();
            }
        }
        return oldEntry == null ? null : oldEntry.value;
    }

    private void markAndSweep() {
        if (!markAndSweepLock.tryLock()) return;
        try {
            doMarkAndSweep();
        } finally {
            isCleaning = false;
            markAndSweepLock.unlock();
        }
    }

    private void doMarkAndSweep() {
        isCleaning = true;

        int sz = stats.size.get();
        long timeCurrent = stats.accessCounter.longValue();

        int numRemoved = 0;
        int numKept = 0;
        long newestEntry = timeCurrent;
        long newNewestEntry = -1;
        long newOldestEntry = Long.MAX_VALUE;

        int wantToKeep = lowerWaterMark;
        int wantToRemove = sz - lowerWaterMark;

        CacheEntry<K,V>[] eset = new CacheEntry[sz];
        int eSize = 0;

        for (CacheEntry<K,V> ce : map.values()) {
            // set lastAccessedCopy to avoid more volatile reads
            ce.lastAccessedCopy = ce.lastAccessed;
            long thisEntry = ce.lastAccessedCopy;

            // since the wantToKeep group is likely to be bigger than wantToRemove, check it first
            if (thisEntry > newestEntry - wantToKeep) {
                // this entry is guaranteed not to be in the bottom
                // group, so do nothing.
                numKept++;
                newOldestEntry = Math.min(thisEntry, newOldestEntry);
            } else if (thisEntry < oldestEntry + wantToRemove) { // entry in bottom group?
                // this entry is guaranteed to be in the bottom group
                // so immediately remove it from the map.
                evictEntry(ce.key);
                numRemoved++;
            } else {
                // This entry *could* be in the bottom group.
                // Collect these entries to avoid another full pass... this is wasted
                // effort if enough entries are normally removed in this first pass.
                // An alternate impl could make a full second pass.
                if (eSize < eset.length-1) {
                    eset[eSize++] = ce;
                    newNewestEntry = Math.max(thisEntry, newNewestEntry);
                    newOldestEntry = Math.min(thisEntry, newOldestEntry);
                }
            }
        }

        // if we didn't remove enough entries, then make more passes
        // over the values we collected, with updated min and max values.
        while (sz - numRemoved > acceptableWaterMark) {
            oldestEntry = newOldestEntry == Long.MAX_VALUE ? oldestEntry : newOldestEntry;
            newOldestEntry = Long.MAX_VALUE;
            newestEntry = newNewestEntry;
            newNewestEntry = -1;
            wantToKeep = lowerWaterMark - numKept;
            wantToRemove = sz - lowerWaterMark - numRemoved;

            // iterate backward to make it easy to remove items.
            for (int i=eSize-1; i>=0; i--) {
                CacheEntry<K,V> ce = eset[i];
                long thisEntry = ce.lastAccessedCopy;

                if (thisEntry > newestEntry - wantToKeep) {
                    // this entry is guaranteed not to be in the bottom
                    // group, so do nothing but remove it from the eset.
                    numKept++;
                    // remove the entry by moving the last element to its position
                    eset[i] = eset[eSize-1];
                    eSize--;

                    newOldestEntry = Math.min(thisEntry, newOldestEntry);

                } else if (thisEntry < oldestEntry + wantToRemove) { // entry in bottom group?

                    // this entry is guaranteed to be in the bottom group
                    // so immediately remove it from the map.
                    evictEntry(ce.key);
                    numRemoved++;

                    // remove the entry by moving the last element to its position
                    eset[i] = eset[eSize-1];
                    eSize--;
                } else {
                    // This entry *could* be in the bottom group, so keep it in the eset,
                    // and update the stats.
                    newNewestEntry = Math.max(thisEntry, newNewestEntry);
                    newOldestEntry = Math.min(thisEntry, newOldestEntry);
                }
            }
        }

        oldestEntry = newOldestEntry == Long.MAX_VALUE ? oldestEntry : newOldestEntry;
        this.oldestEntry = oldestEntry;
    }

    private void evictEntry(K key) {
        CacheEntry<K,V> entry = map.remove(key);
        if (entry == null) return;
        stats.size.decrementAndGet();
    }

    public static class CacheEntry<K,V> implements Comparable<CacheEntry<K,V>> {
        K key;
        V value;
        volatile long lastAccessed = 0;
        long lastAccessedCopy = 0;

        public CacheEntry(K key, V value, long lastAccessed) {
            this.key = key;
            this.value = value;
            this.lastAccessed = lastAccessed;
        }

        public void setLastAccessed(long lastAccessed) {
            this.lastAccessed = lastAccessed;
        }

        @Override
        public int compareTo(CacheEntry<K, V> that) {
            if (this.lastAccessedCopy == that.lastAccessedCopy) return 0;
            return this.lastAccessedCopy < that.lastAccessedCopy ? 1 : -1;
        }
    }

    public static class Stats {
        AtomicLong accessCounter = new AtomicLong(0);
        AtomicInteger size = new AtomicInteger(0);
    }
}
```