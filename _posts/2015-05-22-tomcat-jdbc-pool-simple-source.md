---
layout: post
title: tomcat-jdbc-pool 实现简单分析
description: 在现稍微大一点的软件系统开发中，都会接触到池。有时，并不是没有用到，而是没有去注意到。例如：内存池，线程池，连接池等各种各样的池（pool）
category: blog
---

在现稍微大一点的软件系统开发中，都会接触到池。有时，并不是没有用到，而是没有去注意到。例如：内存池，线程池，连接池等各种各样的池（pool）。

### 先聊聊池

池，不由自主的会想到水池。  
小时候，我们都要去远处的水井挑水，倒进家中的水池里面。这样，每次要用水时，直接从水池中“取”就行了。不用大老远跑去水井打水。

数据库连接池就如此，我们预先准备好一些连接，放到池中。当需要时，就直接获取。而不要每次跟数据库建立一个新的连接。特别对数据库连接这类耗时，耗资源的操作。当连接用完后，再放回池中，供后续使用。

从上可以简单看些，池的一些基本特征：  
。池会有一定的容量，及已经创建好的对象  
。有“借”有“还”操作的接口  

我先前有简单看过dbcp，c3po连接池的实现。相对于简单，精湛tomcat-jdbc-pool，还是复杂不少。
tomcat-jdbc-pool基于jdk1.5后的线程池实现。所以你懂得。

### 有借有还，再借不难
俗话说：“有借有还，再借不难”。
我们刨去各种初始化，各种花枝招展的包装以及一些简单逻辑的卫语句。直接去看看tomcat-jdbc-pool是怎么管理“借”的操作。
`org.apache.tomcat.jdbc.pool.ConnectionPool borrowConnection()`

![image]({{ site.url }}/images/posts/20150524-tomcat-jdbc-pool-borrowConnection-2.png)  

### 借了后怎么还
在连接池中，是不会去关闭真实的数据库连接的。只将归还至可用的池中。  
如果真实关闭数据库连接了，那连接池的又有什么用咧。。。

![image]({{ site.url }}/images/posts/20150524-tomcat-jdbc-pool-returnConnection.png) 

### 清除不良账务
当访问高峰过时，我们会创建不少新的连接。   
高峰过后，我们需要去清理可能暂不再会使用的连接，释放些资源。（如有需要，可再创建嘛。有借有还，这再借当然不难落。）   
_不用去找那么多类，连接的管理都在`ConnectionPool`中。_     
`PoolCleaner extends TimerTask`   
在池的初始化时就进行注册，内部采用`scheduleAtFixedRate`方式，定时扫描`idle`队列中所有的空闲连接，进行释放（当然每个连接有标志其创建和最后将使用完成的时间。通过这些时间判断是否可以释放了）。

#### 贴源码
技术文章写得比较少，正在不断练习中。  
还是直接贴源码，这些源码注释还不错，最好的技术文档了。有空可以多去读读。   

```js
    var body = 'hello world';
    response.writeHead(200, {
        'Content-Length': body.length,
        'Content-Type': 'text/plain'
    });
```

#### borrowConnection
```java
/**
 * Thread safe way to retrieve a connection from the pool
 * @param wait - time to wait, overrides the maxWait from the properties,
 * set to -1 if you wish to use maxWait, 0 if you wish no wait time.
 * @return PooledConnection
 * @throws SQLException
 */
private PooledConnection borrowConnection(int wait, String username, String password) throws SQLException {

    if (isClosed()) {
        throw new SQLException("Connection pool closed.");
    } //end if

    //get the current time stamp
    long now = System.currentTimeMillis();
    //see if there is one available immediately
    PooledConnection con = idle.poll();

    while (true) {
        if (con!=null) {
            //configure the connection and return it
            PooledConnection result = borrowConnection(now, con, username, password);
            //null should never be returned, but was in a previous impl.
            if (result!=null) return result;
        }

        //if we get here, see if we need to create one
        //this is not 100% accurate since it doesn't use a shared
        //atomic variable - a connection can become idle while we are creating
        //a new connection
        if (size.get() < getPoolProperties().getMaxActive()) {
            //atomic duplicate check
            if (size.addAndGet(1) > getPoolProperties().getMaxActive()) {
                //if we got here, two threads passed through the first if
                size.decrementAndGet();
            } else {
                //create a connection, we're below the limit
                return createConnection(now, con, username, password);
            }
        } //end if

        //calculate wait time for this iteration
        long maxWait = wait;
        //if the passed in wait time is -1, means we should use the pool property value
        if (wait==-1) {
            maxWait = (getPoolProperties().getMaxWait()<=0)?Long.MAX_VALUE:getPoolProperties().getMaxWait();
        }

        long timetowait = Math.max(0, maxWait - (System.currentTimeMillis() - now));
        waitcount.incrementAndGet();
        try {
            //retrieve an existing connection
            con = idle.poll(timetowait, TimeUnit.MILLISECONDS);
        } catch (InterruptedException ex) {
            if (getPoolProperties().getPropagateInterruptState()) {
                Thread.currentThread().interrupt();
            } else {
                Thread.interrupted();
            }
            SQLException sx = new SQLException("Pool wait interrupted.");
            sx.initCause(ex);
            throw sx;
        } finally {
            waitcount.decrementAndGet();
        }
        if (maxWait==0 && con == null) { //no wait, return one if we have one
        if (jmxPool!=null) {
            jmxPool.notify(org.apache.tomcat.jdbc.pool.jmx.ConnectionPool.POOL_EMPTY, "Pool empty - no wait.");
            }
            throw new PoolExhaustedException("[" + Thread.currentThread().getName()+"] " +
                    "NoWait: Pool empty. Unable to fetch a connection, none available["+busy.size()+" in use].");
        }
        //we didn't get a connection, lets see if we timed out
        if (con == null) {
            if ((System.currentTimeMillis() - now) >= maxWait) {
                if (jmxPool!=null) {
                    jmxPool.notify(org.apache.tomcat.jdbc.pool.jmx.ConnectionPool.POOL_EMPTY, "Pool empty - timeout.");
                }
                throw new PoolExhaustedException("[" + Thread.currentThread().getName()+"] " +
                    "Timeout: Pool empty. Unable to fetch a connection in " + (maxWait / 1000) +
                    " seconds, none available[size:"+size.get() +"; busy:"+busy.size()+"; idle:"+idle.size()+"; lastwait:"+timetowait+"].");
            } else {
                //no timeout, lets try again
                continue;
            }
        }
    } //while
}
```

#### returnConnection
```java
/**
 * Returns a connection to the pool
 * If the pool is closed, the connection will be released
 * If the connection is not part of the busy queue, it will be released.
 * If {@link PoolProperties#testOnReturn} is set to true it will be validated
 * @param con PooledConnection to be returned to the pool
 */
protected void returnConnection(PooledConnection con) {
    if (isClosed()) {
        //if the connection pool is closed
        //close the connection instead of returning it
        release(con);
        return;
    } //end if

    if (con != null) {
        try {
            con.lock();

            if (busy.remove(con)) {

                if (!shouldClose(con,PooledConnection.VALIDATE_RETURN)) {
                    con.setStackTrace(null);
                    con.setTimestamp(System.currentTimeMillis());
                    if (((idle.size()>=poolProperties.getMaxIdle()) && !poolProperties.isPoolSweeperEnabled()) || (!idle.offer(con))) {
                        if (log.isDebugEnabled()) {
                            log.debug("Connection ["+con+"] will be closed and not returned to the pool, idle["+idle.size()+"]>=maxIdle["+poolProperties.getMaxIdle()+"] idle.offer failed.");
                        }
                        release(con);
                    }
                } else {
                    if (log.isDebugEnabled()) {
                        log.debug("Connection ["+con+"] will be closed and not returned to the pool.");
                    }
                    release(con);
                } //end if
            } else {
                if (log.isDebugEnabled()) {
                    log.debug("Connection ["+con+"] will be closed and not returned to the pool, busy.remove failed.");
                }
                release(con);
            }
        } finally {
            con.unlock();
        }
    } //end if
} //checkIn
```

#### checkIdle
```java
public void checkIdle(boolean ignoreMinSize) {

    try {
        if (idle.size()==0) return;
        long now = System.currentTimeMillis();
        Iterator<PooledConnection> unlocked = idle.iterator();
        while ( (ignoreMinSize || (idle.size()>=getPoolProperties().getMinIdle())) && unlocked.hasNext()) {
            PooledConnection con = unlocked.next();
            boolean setToNull = false;
            try {
                con.lock();
                //the con been taken out, we can't clean it up
                if (busy.contains(con))
                    continue;
                long time = con.getTimestamp();
                if (shouldReleaseIdle(now, con, time)) {
                    release(con);
                    idle.remove(con);
                    setToNull = true;
                } else {
                    //do nothing
                } //end if
            } finally {
                con.unlock();
                if (setToNull)
                    con = null;
            }
        } //while
    } catch (ConcurrentModificationException e) {
        log.debug("checkIdle failed." ,e);
    } catch (Exception e) {
        log.warn("checkIdle failed, it will be retried.",e);
    }

}


protected boolean shouldReleaseIdle(long now, PooledConnection con, long time) {
    if (con.getConnectionVersion() < getPoolVersion()) return true;
    else return (con.getReleaseTime()>0) && ((now - time) > con.getReleaseTime()) && (getSize()>getPoolProperties().getMinIdle());
}

private static volatile Timer poolCleanTimer = null;
private static HashSet<PoolCleaner> cleaners = new HashSet<>();

private static synchronized void registerCleaner(PoolCleaner cleaner) {
    unregisterCleaner(cleaner);
    cleaners.add(cleaner);
    if (poolCleanTimer == null) {
        ClassLoader loader = Thread.currentThread().getContextClassLoader();
        try {
            Thread.currentThread().setContextClassLoader(ConnectionPool.class.getClassLoader());
            poolCleanTimer = new Timer("PoolCleaner["+ System.identityHashCode(ConnectionPool.class.getClassLoader()) + ":"+
                                       System.currentTimeMillis() + "]", true);
        }finally {
            Thread.currentThread().setContextClassLoader(loader);
        }
    }
    poolCleanTimer.scheduleAtFixedRate(cleaner, cleaner.sleepTime,cleaner.sleepTime);
}

private static synchronized void unregisterCleaner(PoolCleaner cleaner) {
    boolean removed = cleaners.remove(cleaner);
    if (removed) {
        cleaner.cancel();
        if (poolCleanTimer != null) {
            poolCleanTimer.purge();
            if (cleaners.size() == 0) {
                poolCleanTimer.cancel();
                poolCleanTimer = null;
            }
        }
    }
}

protected static class PoolCleaner extends TimerTask {
    protected WeakReference<ConnectionPool> pool;
    protected long sleepTime;
    protected volatile long lastRun = 0;

    PoolCleaner(ConnectionPool pool, long sleepTime) {
        this.pool = new WeakReference<>(pool);
        this.sleepTime = sleepTime;
        if (sleepTime <= 0) {
            log.warn("Database connection pool evicter thread interval is set to 0, defaulting to 30 seconds");
            this.sleepTime = 1000 * 30;
        } else if (sleepTime < 1000) {
            log.warn("Database connection pool evicter thread interval is set to lower than 1 second.");
        }
    }

    @Override
    public void run() {
        ConnectionPool pool = this.pool.get();
        if (pool == null) {
            stopRunning();
        } else if (!pool.isClosed() &&
                (System.currentTimeMillis() - lastRun) > sleepTime) {
            lastRun = System.currentTimeMillis();
            try {
                if (pool.getPoolProperties().isRemoveAbandoned())
                    pool.checkAbandoned();
                if (pool.getPoolProperties().getMinIdle() < pool.idle
                        .size())
                    pool.checkIdle();
                if (pool.getPoolProperties().isTestWhileIdle())
                    pool.testAllIdle();
            } catch (Exception x) {
                log.error("", x);
            }
        }
    }

    public void start() {
        registerCleaner(this);
    }

    public void stopRunning() {
        unregisterCleaner(this);
    }
}
```






