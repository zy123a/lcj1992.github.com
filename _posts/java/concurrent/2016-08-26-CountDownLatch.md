---
layout: post
tilte: CountDownLatch源码分析
categories: java
tags: countDownLatch juc aqs
---

## CountDownLatch构造器 {#init}

    public CountDownLatch(int count) {
        if (count < 0) throw new IllegalArgumentException("count < 0");
        this.sync = new Sync(count);
    }


    Sync(int count) {
        // 设置state为count
        setState(count);
    }

## CountDownLatch#countDown {#countDown}

    public void countDown() {
        sync.releaseShared(1);
    }

### AbstractQueuedSynchronizer#releaseShared {#aqs_releaseShared}

    public final boolean releaseShared(int arg) {
        if (tryReleaseShared(arg)) {
            // 如果state为0，走不到这里；如果cas进行state-1后state不为0，也走不到这里。详看tryReleaseShared
            doReleaseShared();
            return true;
        }
        return false;
    }

### CountDownLatch#tryReleaseShared {#CountDownLatch#tryReleaseShared}

    protected boolean tryReleaseShared(int releases) {
        // Decrement count; signal when transition to zero
        for (;;) {
            int c = getState();
            if (c == 0)
                // 如果线程持有锁
                return false;
            int nextc = c-1;
            // 可能并发的改state的值，所以这里需要cas
            if (compareAndSetState(c, nextc))
                // 如果nextc不为0，返回false
                return nextc == 0;
        }
    }

### AbstractQueuedSynchronizer#doReleaseShared {#aqs_doReleaseShared}

如果tryReleaseShared返回true

    private void doReleaseShared() {
       for (;;) {
           Node h = head;
           if (h != null && h != tail) {
               int ws = h.waitStatus;

               // signal : waitStatus value to indicate successor's thread needs unparking    
               // 后继节点等待被触发
               if (ws == Node.SIGNAL) {
                   // 如果节点状态为signal，cas重置节点状态为0
                   if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                       continue;            // loop to recheck cases
                   // 如果重置成功，唤醒后继节点       
                   unparkSuccessor(h);
               }
               // 如果头节点为状态为初始状态，尝试设置waitStatus为propagate
               // propagate: waitStatus value to indicate the next acquireShared should unconditionally propagate
               else if (ws == 0 &&
                        !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                   continue;                // loop on failed CAS
           }
           if (h == head)                   // loop if head changed
               break;
       }
   }

### AbstractQueuedSynchronizer#unparkSuccessor {#aqs_unparkSuccessor}

    private void unparkSuccessor(Node node) {
       int ws = node.waitStatus;
       // 唤醒后继节点，需要当前节点的waitStatus置为0或1
       if (ws < 0)
           compareAndSetWaitStatus(node, ws, 0);

       Node s = node.next;
       if (s == null || s.waitStatus > 0) {
           // 如果s为null或者s为取消状态    
           s = null;
           // 从后向前遍历node，直到第一个waitStatus不是cancel的node
           for (Node t = tail; t != null && t != node; t = t.prev)
               if (t.waitStatus <= 0)
                   s = t;
       }
       然后unpark之。
       if (s != null)
           LockSupport.unpark(s.thread);
   }
