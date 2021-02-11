---
layout: post
title:  "pintos-Priority-Scheduling[작성중]"
date:   2021-02-08 02:26:01 +0900
tag: etc
---

busy wait는 내가 코드 짜는데 손을 아예 안댔기에 생략.

```
Implement priority scheduling in Pintos. When a thread is added to the ready list that has a higher priority than the currently running thread, the current thread should immediately yield the processor to the new thread. 
Similarly, when threads are waiting for a lock, semaphore, or condition variable, the highest priority waiting thread should be awakened first. A thread may raise or lower its own priority at any time, but lowering its priority such that it no longer has the highest priority must cause it to immediately yield the CPU.

Thread priorities range from PRI_MIN (0) to PRI_MAX (63). Lower numbers correspond to lower priorities, so that priority 0 is the lowest priority and priority 63 is the highest. The initial thread priority is passed as an argument to thread_create(). If there's no reason to choose another priority, use PRI_DEFAULT (31). The PRI_ macros are defined in threads/thread.h, and you should not change their values.

One issue with priority scheduling is "priority inversion". Consider high, medium, and low priority threads H, M, and L, respectively. If H needs to wait for L (for instance, for a lock held by L), and M is on the ready list, then H will never get the CPU because the low priority thread will not get any CPU time. A partial fix for this problem is for H to "donate" its priority to L while L is holding the lock, then recall the donation once L releases (and thus H acquires) the lock.

Implement priority donation. You will need to account for all different situations in which priority donation is required. Be sure to handle multiple donations, in which multiple priorities are donated to a single thread. You must also handle nested donation: if H is waiting on a lock that M holds and M is waiting on a lock that L holds, then both M and L should be boosted to H's priority. If necessary, you may impose a reasonable limit on depth of nested priority donation, such as 8 levels.

You must implement priority donation for locks. You need not implement priority donation for the other Pintos synchronization constructs. You do need to implement priority scheduling in all cases.

Finally, implement the following functions that allow a thread to examine and modify its own priority. Skeletons for these functions are provided in threads/thread.c.


Function: void thread_set_priority (int new_priority)
Sets the current thread's priority to new_priority. If the current thread no longer has the highest priority, yields.

Function: int thread_get_priority (void)
Returns the current thread's priority. In the presence of priority donation, returns the higher (donated) priority.
You need not provide any interface to allow a thread to directly modify other threads' priorities.

The priority scheduler is not used in any later project.

FAQ

Doesn't priority scheduling lead to starvation?
Yes, strict priority scheduling can lead to starvation because a thread will not run if any higher-priority thread is runnable. The advanced scheduler introduces a mechanism for dynamically changing thread priorities.

Strict priority scheduling is valuable in real-time systems because it offers the programmer more control over which jobs get processing time. High priorities are generally reserved for time-critical tasks. It's not "fair," but it addresses other concerns not applicable to a general-purpose operating system.

What thread should run after a lock has been released?
When a lock is released, the highest priority thread waiting for that lock should be unblocked and put on the list of ready threads. The scheduler should then run the highest priority thread on the ready list.

If the highest-priority thread yields, does it continue running?
Yes. If there is a single highest-priority thread, it continues running until it blocks or finishes, even if it calls thread_yield(). If multiple threads have the same highest priority, thread_yield() should switch among them in "round robin" order.

What happens to the priority of a donating thread?
Priority donation only changes the priority of the donee thread. The donor thread's priority is unchanged. Priority donation is not additive: if thread A (with priority 5) donates to thread B (with priority 3), then B's new priority is 5, not 8.

Can a thread's priority change while it is on the ready queue?
Yes. Consider a ready, low-priority thread L that holds a lock. High-priority thread H attempts to acquire the lock and blocks, thereby donating its priority to ready thread L.

Can a thread's priority change while it is blocked?
Yes. While a thread that has acquired lock L is blocked for any reason, its priority can increase by priority donation if a higher-priority thread attempts to acquire L. This case is checked by the priority-donate-sema test.

Can a thread added to the ready list preempt the processor?
Yes. If a thread added to the ready list has higher priority than the running thread, the correct behavior is to immediately yield the processor. It is not acceptable to wait for the next timer interrupt. The highest priority thread should run as soon as it is runnable, preempting whatever thread is currently running.

How does thread_set_priority() affect a thread receiving donations?
It sets the thread's base priority. The thread's effective priority becomes the higher of the newly set priority or the highest donated priority. When the donations are released, the thread's priority becomes the one set through the function call. This behavior is checked by the priority-donate-lower test.

Doubled test names in output make them fail.
Suppose you are seeing output in which some test names are doubled, like this:

 	
(alarm-priority) begin
(alarm-priority) (alarm-priority) Thread priority 30 woke up.
Thread priority 29 woke up.
(alarm-priority) Thread priority 28 woke up.
What is happening is that output from two threads is being interleaved. That is, one thread is printing "(alarm-priority) Thread priority 29 woke up.\n" and another thread is printing "(alarm-priority) Thread priority 30 woke up.\n", but the first thread is being preempted by the second in the middle of its output.

This problem indicates a bug in your priority scheduler. After all, a thread with priority 29 should not be able to run while a thread with priority 30 has work to do.

Normally, the implementation of the printf() function in the Pintos kernel attempts to prevent such interleaved output by acquiring a console lock during the duration of the printf call and releasing it afterwards. However, the output of the test name, e.g., (alarm-priority), and the message following it is output using two calls to printf, resulting in the console lock being acquired and released twice.



```

Pintos에서 우선 순위 예약을 구현합니다.
현재 실행 중인 스레드보다 우선 순위가 높은 스레드가 준비 목록에 추가되면 현재 스레드는 즉시 새 스레드에 프로세서를 제공해야 합니다.

마찬가지로, 스레드가 잠금, 세마포어 또는 조건 변수를 대기할 때 가장 높은 우선 순위 대기 스레드를 먼저 깨워야 합니다. 스레드는 언제든지 자체 우선 순위를 올리거나 낮출 수 있지만 더 이상 가장 높은 우선 순위를 갖지 않도록 우선 순위를 낮추면 CPU를 즉시 산출해야 합니다.

스레드 우선 순위는 PRI_MIN(0)부터 PRI_MAX(63)까지입니다. 숫자가 낮을수록 우선순위가 낮아지므로 우선 순위 0이 가장 낮고 우선 순위 63이 가장 높습니다. 초기 스레드 우선 순위가 thread_create()에 인수로 전달됩니다. 다른 우선 순위를 선택할 이유가 없으면 PRI_DEFAULT(31)를 사용하십시오. PRI_ 매크로는 스레드/스레드.h로 정의되며 값을 변경하면 안 됩니다.

우선 순위 스케줄링의 한 가지 문제는 "우선 순위 반전"입니다. 각각 높음, 중간 및 낮음 우선 순위 스레드 H, M 및 L을 고려합니다. H가 L(예: L에 의해 고정되는 잠금)을 기다려야 하고 M이 준비 목록에 있으면 우선 순위가 낮은 스레드는 CPU 시간을 얻지 못하기 때문에 H가 CPU를 가져올 수 없습니다. 이 문제에 대한 부분적인 해결책은 L이 자물쇠를 잡고 있는 동안 H가 우선권을 L에게 "기부"한 다음, L이 자물쇠를 풀면 (따라서 H가) 기부금을 회수하는 것이다.

우선 기부금을 집행합니다. 우선적인 기부가 필요한 모든 상황을 고려해야 할 것이다. 여러 개의 우선 순위를 하나의 실에 기부하는 여러 개의 기부금을 처리하십시오. 내포된 기부도 처리해야 합니다. H가 M이 잡고 있는 자물쇠에 대기하고 M이 L이 잡고 있는 자물쇠에 대기하고 있다면 M과 L이 모두 H의 우선순위로 올라가야 합니다. 필요한 경우 8단계와 같이 중첩된 우선 순위 기부 깊이에 적절한 제한을 둘 수 있습니다.

자물쇠에 대한 우선적인 기부를 실시해야 한다. 다른 Pintos 동기화 구성에는 우선 기부금을 구현할 필요가 없습니다. 모든 경우 우선 순위 예약을 구현해야 합니다.

마지막으로 스레드가 자체 우선 순위를 검사하고 수정할 수 있는 다음 기능을 구현합니다. 이러한 기능을 위한 골격은 스레드/스레드.c로 제공됩니다.


함수: void thread_set_priority(new_priority에서)
현재 스레드의 우선 순위를 new_priority로 설정합니다. 현재 스레드의 우선 순위가 더 이상 높지 않으면 산출합니다.

함수: int thread_get_priority(void)
현재 스레드의 우선 순위를 반환합니다. 우선 순위 기부가 있을 경우 더 높은(기부된) 우선 순위를 반환합니다.
스레드가 다른 스레드의 우선 순위를 직접 수정할 수 있도록 허용하는 인터페이스는 제공할 필요가 없습니다.

우선 순위 스케줄러는 이후 프로젝트에서 사용되지 않습니다.



```
FAIL tests/threads/priority-change
FAIL tests/threads/priority-donate-one
FAIL tests/threads/priority-donate-multiple
FAIL tests/threads/priority-donate-multiple2
FAIL tests/threads/priority-donate-nest
FAIL tests/threads/priority-donate-sema
FAIL tests/threads/priority-donate-lower
FAIL tests/threads/priority-fifo
FAIL tests/threads/priority-preempt
FAIL tests/threads/priority-sema
FAIL tests/threads/priority-condvar
FAIL tests/threads/priority-donate-chain
```



list_insert_ordered(&sleeping_list,&cur_thread -> elem,order_by_wakeup_tick,NULL);

요구사항을 정리했을때 다음과 같이 정리된다.

1. When a thread is added to the ready list that has a higher priority than the currently running thread, the current thread should immediately yield the processor to the new thread
2. when threads are waiting for a lock, semaphore, or condition variable, the highest priority waiting thread should be awakened first.
3. One issue with priority scheduling is "priority inversion". Consider high, medium, and low priority threads H, M, and L, respectively. If H needs to wait for L (for instance, for a lock held by L), and M is on the ready list, then H will never get the CPU because the low priority thread will not get any CPU time. A partial fix for this problem is for H to "donate" its priority to L while L is holding the lock, then recall the donation once L releases (and thus H acquires) the lock.
4. void thread_set_priority (int new_priority) Sets the current thread's priority to new_priority. If the current thread no longer has the highest priority, yields.
5.  Function: int thread_get_priority (void) Returns the current thread's priority. In the presence of priority donation, returns the higher (donated) priority.


-----------
가장먼저 든생각은 `ready_list`를 우선순위 큐 자료구조로 만드는 것이다. 이렇게하기에는 너무 번거롭기에 busy wait에서 해결한 방식인 insert_sort를 그대로 사용해보도록 한다.

1. 새로운 thread가 만들어질때를 추적해서 그부분을 고치면 될것같다. 
2. lock, semaphore, condition variable 이 풀릴때 모두 priority 순으로 풀리길 원한다.
3. 은 너무 막막하다. 일단 스킵
4. 는 1에서의 코드를 그대로 적용시키면될것같다
5. 3와같이 해결해야될듯하다 스킵

ready_list에 thread가 들어가는 부분을 모두 삽입정렬로 들어가게 바꾸었다.
```c
/* list_less_function for ready_list_insert_ordered. Inserting thread in order by larger priority values. */
bool order_by_priority(struct list_elem* a, struct list_elem* b)
{
  struct thread* thread_a = list_entry(a,struct thread,elem);
  struct thread* thread_b = list_entry(b,struct thread,elem);

  return thread_a->priority > thread_b->priority;
}


thread_unblock...
thread_yield
  list_insert_ordered(&ready_list, &t->elem, order_by_priority, NULL);


/* Sets the current thread's priority to NEW_PRIORITY. */
void
thread_set_priority (int new_priority) 
{
  struct thread* front_ready_thread = list_entry(list_begin(&ready_list), struct thread, elem);

  thread_current ()->priority = new_priority;
  if (front_ready_thread->priority > new_priority)
    thread_yield();
}
```


```c
tid_t
thread_create (const char *name, int priority,
               thread_func *function, void *aux) 
{
...
  /* Add to run queue. */
  thread_unblock (t);
  if (t->priority > thread_get_priority ())
    thread_yield ();
  return tid;
}

```



1,4를 먼저 해결했다.

```
pass tests/threads/priority-change
FAIL tests/threads/priority-donate-one
FAIL tests/threads/priority-donate-multiple
FAIL tests/threads/priority-donate-multiple2
FAIL tests/threads/priority-donate-nest
FAIL tests/threads/priority-donate-sema
FAIL tests/threads/priority-donate-lower
pass tests/threads/priority-fifo
pass tests/threads/priority-preempt
FAIL tests/threads/priority-sema
FAIL tests/threads/priority-condvar
FAIL tests/threads/priority-donate-chain
```
donate관련 /sema conditionvariable 으로 딱 나눠진 두부분만 통과를 못한것을 알수있다.
donate가 가장 복잡해 보이므로 sema cond쪽을 먼저 해결하려했다.

2.를 해결하려 보니 synch 에 sema, lock, condi 가 모두 들어있더라. 또 구조를 보니 sema 쪽에 코드를 잘 짜면 lock, condi가 모두 sema를 응용하기에 같이 해결될거라 생각했다. 
각각 구조와 함수가 간단한편이라 이해하는거 어렵진않았다.
내부에 waiting하는 리스트가 있기에 이 list를 priority로 정렬되게 하면 우선순위순으로 알아서 풀리지 않을까란 생각을 했다. 앞 과제와 똑같은 방식이 었다.
따라서 해당 리스트가 push_back하던 코드를 ready_list가 정렬된상태로 들어가듯이 모두 고쳐줬다.


이런다고 추가적으로 pass하는 코드는 없더라.


하루동안 골머리를 썩다가 무엇이 문제인지 알았는데.
unblock이 되면서 ready_list에 올라가는데 이때 ready_list에만 올라가는 것이 아닌 thread_yield 또한 호출하여 priority에 따른 스레드를 실행시켜야한다.

```c
void
sema_up (struct semaphore *sema) 
{ ...
  intr_set_level (old_level);
  thread_yield();
}
```
thread_yield 에서 추가적인 intr_disable()을 호출하기에 이것이 이미 disable 된 상태에서 호출하는것이 문제가 없는가 했지만 fail을 하는등하는 일은 없지만 세마포어는 확실하게 up된상태에서 thread_yield를 호출해야한다 그렇지않으면 영원히 기다리기만한다.
사실 unblock 된 상태에서 priority가 현재 스레트보다 더 큰게 확실할때만 thread_yield를 불러야 되는거 아닌가 생각했지만 내부 함수를 뜯어보니 로직이 현재 스레드를 ready_list로 집어넣는다(위에서 고쳤으므로 priority로 정렬됨이 보장된다) ready_list에서 새로 꺼낸다.
따라서 unblock여부에 상관없이 적절한것이 알아서 튀어나온다.
조금더 효율을 중시한다면 다음과 같이 짤 수도있겠지만 코드의 간편화를 아래가아닌 위해서 위와같이 코드를 하기로 정했다.

```c
void
sema_up (struct semaphore *sema) 
{
  enum intr_level old_level;

  ASSERT (sema != NULL);

  old_level = intr_disable ();
  if (!list_empty (&sema->waiters)) 
  {  
    struct thread* blocked = list_entry (list_pop_front (&sema->waiters),
                                struct thread, elem);
    thread_unblock (blocked);
    sema->value++;
    if(blocked->priority > thread_current ()->priority)
    {
      thread_yield();
    }
  }
  else
  {
    sema->value++;
  }
  intr_set_level (old_level);
}
```
재밌는 부분은 if(block...)를 제외하고 if내에서 무조건 thread_yield를 한번씩 불렀을때는 tests/threads/mlfqs-block를 통과한다.

```shell
pass tests/threads/priority-sema
```

priority-condvar 쪽이 통과가 되지 않는게 좀 이해가 안갔다.
result 기록을 보니 완전히 insert 삽입이 제대로 이루어 지지않는듯했다.

생각없이 코드만 바꿔넣었을뿐인데 order_by_priority의 내부 상 thread이라고 임의로 변환한 부분에서 아마 문제가 있는것으로 추측해보았다. 따라서 해당 코드를 더 빠삭하게 이해할 필요가 있다.

```c
void
cond_wait (struct condition *cond, struct lock *lock) 
{
  struct semaphore_elem waiter;

  ASSERT (cond != NULL);
  ASSERT (lock != NULL);
  ASSERT (!intr_context ());
  ASSERT (lock_held_by_current_thread (lock));
  
  sema_init (&waiter.semaphore, 0);
  list_insert_ordered(&cond->waiters, &waiter.elem, order_by_priority, NULL);
  lock_release (lock);
  sema_down (&waiter.semaphore);
  lock_acquire (lock);
}


/* comapre_function for ready_list_insert_ordered. Inserting thread in order by larger priority values. */
bool 
order_by_priority(struct list_elem* a, struct list_elem* b)
{
  struct thread* thread_a = list_entry(a,struct thread,elem);
  struct thread* thread_b = list_entry(b,struct thread,elem);

  return thread_a->priority > thread_b->priority;
}
```


cond_signal을 봤는데 list_entry로 튀어나오는 구조체가 `struct semaphore_elem`이더라. 따라서 넣을때도 이 구조체에 맞게넣어야지 thread로 변환한다고 제대로 나올리 없다.

priority를 이제 어디서 구해서 어디로 넣는가에 대한 문제가 남았는데 넣을곳이 `semaphore_elem`에다가 직접 변수하나 박아 넣는 방법말고는 도저히 모르겠다. priority 값 자체는 thread_get_priority로 불러오면 별로 문제가 없어보인다. semaphore_elem는 일단 C언어 전체에서는 사용하는 부분이 cond쪽 함수를 사용하기위해서 말고는 없다. 
따라서 여기에다가 추가적인 선언을 하기로했다.

하단은 비교코드 구조체가 thread에서 바뀐것을 제외하곤 차이가 없다.  
```
/* comapre_function for semaphore_elem waiters_insert_ordered. Inserting larger priority values. */
bool 
semaphore_elem_order_by_priority(struct list_elem* a, struct list_elem* b)
{
  struct semaphore_elem* sema_elem_a = list_entry(a,struct semaphore_elem,elem);
  struct semaphore_elem* sema_elem_b = list_entry(b,struct semaphore_elem, elem);
  return sema_elem_a->priority > sema_elem_b->priority;
}
```

이제 세마포어 리스트의 각각의 원소에는 스레드가 대응할수있다고 볼수있다. 아마 도네이션쪽의 문제 해결도 이와 밀접한 연관이 있지 않을까 기대해본다.


```shell
pass tests/threads/priority-condvar
```

# donate

이제 donate쪽을 건드려보자.
솔직히 고려할게 너무 많아서 어디서부터 손을 대야할지 모르겠다.












```shell
pass tests/threads/priority-change
pass tests/threads/priority-donate-one
pass tests/threads/priority-donate-multiple
FAIL tests/threads/priority-donate-multiple2
FAIL tests/threads/priority-donate-nest
pass tests/threads/priority-donate-sema
FAIL tests/threads/priority-donate-lower
pass tests/threads/priority-fifo
pass tests/threads/priority-preempt
pass tests/threads/priority-sema
pass tests/threads/priority-condvar
```