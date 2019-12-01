---
title: Semaphores
tags: Module, CS2106
---

## Basic synchronization patterns

### Rendezvous

设定一个等待点，两个人都到齐之后再一起出发

```C
Semaphore aArrived = 0;
Semaphore bArrived = 0;

// Person A

statement a1;
signal(aArrived);
wait(bArrived);
statement a2;

// Person B

statement b1;
signal(bArrived);
wait(aArrived);
statement b2;
```

### Mutex

一个Critical Section，里面同时最多有一个人

```C
Semaphore mutex = 1;
wait(mutex);
/// Critical Section
signal(mutex);
```

### Multiplex

把Mutex的Semaphore数字改大就可以允许最多有$N$个人在CS里共存

### Barrier


#### One-time barrier

```C
mutex = 1
block = 0
count = 0

wait(mutex)
  count += 1
signal(mutex)

if (count == N) signal(block)

wait(block)
signal(block)
```

#### Reusable barrier

```C
mutex = 1
turn1 = 0
turn2 = 1
count = 0

wait(mutex)
count += 1
if (count == n)
	wait(turn2)
	signal(turn1)
signal(mutex)

wait(turn1)
signal(turn1)

// CS

wait(mutex)
count -= 1
if (count == 0)
	wait(turn1)
	signal(turn2)
signal(mutex)

wait(turn2)
signal(turn2)
```

#### Preloaded turnstile

![](https://i.imgur.com/xduU3hY.png)


### Queue

#### Normal queue

```C
hasLeader = 0
hasFollower = 0

// leader
signal(hasLeader)
wait(hasFollower)
dance()

// follower
signal(hasFollower)
wait(hasLeader)
dance()
```

#### Exclusive queue

## Classical synchronization problems

### Producer-consumer

### Readers-writers

#### Normal solution (Light switch)

```C
Semaphore mutex = 1;
Semaphore isEmpty = 1;
int nReaders = 0;
```

Writers

```C
wait(isEmpty);
write();
signal(isEmpty);
```

Readers

```C
wait(mutex);
nReaders = nReaders + 1;
if(nReaders == 1) wait(isEmpty);
signal(mutex);

read();

wait(mutex);
nReaders = nReaders - 1;
is(nReaders == 0) signal(isEmpty);
signal(mutex);
```

LightSwitch

第一个进门的开灯，之后随便进（lock），最后一个出门的关灯。

```C
int count = 0;
Semaphore mutex = 1;

void lock(Semaphore sem) {
  wait(mutex);
  count = count + 1;
  if(count == 1) wait(sem);
  signal(mutex);
}

void unlock(Semaphore sem) {
  wait(mutex);
  count = count - 1;
  if(count == 0) signal(sem);
  signal(mutex);
}
```

#### No-starve

```C
Semaphore mutex = 1;
Semaphore isEmpty = 1;
Semaphore turnstile = 1;
int nReaders;
```

Writers

```C
wait(turnstile); // added
wait(isEmpty);
write();
signal(isEmpty);
signal(turnstile);
```

Readers

```C
wait(turnstile); // added
signal(turnstile); // added
wait(mutex);
nReaders = nReaders + 1;
if(nReaders == 1) wait(isEmpty);
signal(mutex);

read();

wait(mutex);
nReaders = nReaders - 1;
is(nReaders == 0) signal(isEmpty);
signal(mutex);
```

#### Writer-priority

```C
Lightswitch writerSwitch;
Lightswitch readerSwitch;
Semaphore canWrite;
Semaphore canRead;
```

Writers

```C
writerSwitch.lock(canRead);
wait(canWrite);
write();
signal(canWrite);
writerSwitch.unlock(canRead);
```

Readers

```C
wait(canRead);
readerSwitch.lock(canWrite);
signal(canRead);
read();
readerSwitch.unlock(canWrite);
```