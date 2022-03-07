---
layout: post
title:  "StampedLock"
date:   2022-03-07 15:13:00 +0800
parent: Java锁
categories: Java锁
---
## StampedLock
https://medium.com/double-pointer/guide-to-stampedlock-in-java-7c41f9a4a987

StampedLock提供三种模式控制读写的并发执行。
* Reading
通过#readLock() 获取一个读锁，返回一个long stamp，可以用来释放 read lock;
当一个Thread 获取一个read lock后，任何 write lock的尝试获取都是阻塞的。
* Writing
通过#writeLock() 获取一个独占锁， 返回一个long stamp, 可以用来释放 write lock;
当一个Thread 获取一个write lock后，任何 read lock的尝试获取都是阻塞的。
* Optimistic Read
tryOptimisticRead() 返回一个stamp，如果write lock没有被独占。这个线程可以read 但是必须做个检查 validate(long)，
* 判断这段时间是否有write lock进入。

## sample
```
import java.util.concurrent.locks.StampedLock;

class Point {
    private double x, y;
    private final StampedLock sl = new StampedLock();

    void move(double deltaX, double deltaY) {
        long stamp = sl.writeLock(); // 获取写锁
        try {
            x += deltaX;
            y += deltaY;
        } finally {
            sl.unlockWrite(stamp);
        }
    }

    double distanceFromOrigin() {
        long stamp = sl.tryOptimisticRead(); // 乐观读
        double currentX = x, currentY = y;
        if (!sl.validate(stamp)) { // 检查读的版本是否发生变化（期间有没有写)）
            stamp = sl.readLock(); // 获取读锁
            try {
                currentX = x;
                currentY = y;
            } finally {
                sl.unlockRead(stamp); // 释放读锁
            }
        }
        return Math.sqrt(currentX * currentX + currentY * currentY);
    }

    void moveIfAtOrigin(double newX, double newY) { // upgrade
        // Could instead start with optimistic, not read mode
        long stamp = sl.readLock();
        try {
            while (x == 0.0 && y == 0.0) {
                long ws = sl.tryConvertToWriteLock(stamp);
                if (ws != 0L) {
                    stamp = ws;
                    x = newX;
                    y = newY;
                    break;
                }
                else {
                    sl.unlockRead(stamp);
                    stamp = sl.writeLock();
                }
            }
        } finally {
            sl.unlock(stamp);
        }
    }
}
```