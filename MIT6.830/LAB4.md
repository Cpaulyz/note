# LAB4

## exercise1&2

1和2其实可以放一起讨论，2是在1基础上对以往代码进行修正

实现一个页级的锁管理器

二阶段锁，规则：

* 在一个transaction能够读资源前，必须获得资源的 shared lock。
* 在一个transaction能够写资源前，必须获得资源的 exclusive locks。
* 多个transaction可以同时获得资源的 shared lock。
* 同一时间，只有一个transaction可以获得资源的 exclusive locks。
* 如果transaction t 是唯一一个拥有资源 o shared lock 的transaction，t可以将它对资源o的锁更新为exclusive locks

主要实现：

* Modify getPage() to block and acquire the desired lock before returning a page.
* Implement releasePage(). This method is primarily used for testing, and at the end of transactions.
* Implement holdsLock() so that logic in Exercise 2 can determine whether a page is already locked by a transaction.

需要引入一个`LockManager`，进行锁管理即可。下面仅展示实现所需的核心代码，方法补全上只需要调用`LockManager`提高的服务即可

```java
    enum LockType{
        SHARED_LOCK, EXCLUSIVE_LOCK;
    }
    private class Lock{
        TransactionId tid;
        LockType lockType;   // 0 for shared lock and 1 for exclusive lock

        public Lock(TransactionId tid,LockType lockType){
            this.tid = tid;
            this.lockType = lockType;
        }
    }
    private class LockManager {
        ConcurrentHashMap<PageId, Vector<Lock>> pageLocks = new ConcurrentHashMap<>();
        ConcurrentHashMap<PageId, Object> txLocks = new ConcurrentHashMap<>();
        /**
         * 获取锁
         * 场景：已持有锁、尝试获取读锁、尝试获取写锁、读锁升级写锁
         * 注意：别的事务持有锁，导致当前事务无法获取锁之后需要进行等待和通知
         * @return 是否获取锁成功
         */
        public synchronized boolean acquireLock(PageId pageId,Lock lock){
            if(pageLocks.get(pageId)==null){ // 没人有锁，成功
                Vector<Lock> locks = new Vector<>();
                locks.add(lock);
                pageLocks.put(pageId,locks);
                return true;
            }
            Vector<Lock> locks = pageLocks.get(pageId);
            for(Lock l:locks){
                if(l.tid.equals(lock.tid)){
                    if(l.lockType==lock.lockType){
                        return true;  // 已持有锁
                    }else{
                        if(l.lockType==LockType.EXCLUSIVE_LOCK){
                            return true; // 已有排它锁，可以有共享锁
                        }else if(l.lockType==LockType.SHARED_LOCK){
                            if(locks.size()==1){
                                // 读锁升级写锁
                                l.lockType = LockType.EXCLUSIVE_LOCK;
                                return true;
                            }else {
                                return false;
                            }
                        }

                    }
                }
            }
            // 如果没有过，并且其他都是共享锁，可以加锁
            if(lock.lockType==LockType.SHARED_LOCK&&locks.get(0).lockType==LockType.SHARED_LOCK){
                locks.add(lock);
                return true;
            }
            return false;
        }

        /**
         * 释放锁
         * @return true: 成功 false: 没锁
         */
        public synchronized boolean releaseLock(TransactionId tid){
            for(PageId pageId:pageLocks.keySet()){
                if(holdsLock(pageId,tid)){
                    boolean res = releaseLock(pageId,tid);
                    if(!res){
                        return false;
                    }
                    // TODO：先全部唤醒，可优化
                    Object object = getTxLock(pageId);
                    object.notifyAll();
                }
            }
            return true;
        }

        /**
         * 释放锁
         * @return true: 成功 false: 没锁
         */
        public synchronized boolean releaseLock(PageId pageId,TransactionId tid){
            assert pageLocks.get(pageId) != null : "page not locked!";
            Vector<Lock> locks = pageLocks.get(pageId);
            for(Lock l:locks){
                if(l.tid.equals(tid)){
                    locks.remove(l);
                    if(locks.size()==0){
                        pageLocks.remove(pageId);
                    }
                    return true;
                }
            }
            return false;
        }

        /**
         * 判断tix是否持有锁
         */
        public synchronized boolean holdsLock(PageId pageId,TransactionId tid){
            Vector<Lock> locks = pageLocks.get(pageId);
            if(locks==null){
                return false;
            }
            for(Lock l:locks){
                if(l.tid.equals(tid)){
                    return true;
                }
            }
            return false;
        }

        public Object getTxLock(PageId pageId){
            txLocks.computeIfAbsent(pageId, k -> new Object());
            return txLocks.get(pageId);
        }
    }

}

```



