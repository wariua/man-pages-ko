# 스레드

* [[pthreads(7)]], [[nptl(7)]]
* [[pthread_create(3)]], [[pthread_exit(3)]], [[pthread_detach(3)]], [[pthread_join(3)]], [[pthread_tryjoin_np(3)]]
* [[pthread_equal(3)]], [[pthread_self(3)]]
* [[pthread_attr_init(3)]], [[pthread_attr_setdetachstate(3)]], [[pthread_attr_setguardsize(3)]], [[pthread_attr_setschedpolicy(3)]], [[pthread_attr_setschedparam(3)]], [[pthread_attr_setinheritsched(3)]], [[pthread_attr_setaffinity_np(3)]], [[pthread_attr_setscope(3)]], [[pthread_attr_setstack(3)]], [[pthread_attr_setstackaddr(3)]], [[pthread_attr_setstacksize(3)]], [[pthread_getattr_np(3)]], [[pthread_getattr_default_np(3)]]
* [[pthread_kill(3)]], [[pthread_kill_other_threads_np(3)]], [[pthread_sigmask(3)]], [[pthread_sigqueue(3)]]
* [[pthread_cancel(3)]], [[pthread_setcancelstate(3)]], [[pthread_testcancel(3)]], [[pthread_cleanup_push(3)]], [[pthread_cleanup_push_defer_np(3)]]
* [[pthread_once(3p)]], [[pthread_atfork(3)]]
* [[pthread_getspecific(3p)]], [[pthread_key_create(3p)]], [[pthread_key_delete(3p)]]
* [[pthread_setname_np(3)]]
* [[pthread_getcpuclockid(3)]]
* [[pthread_barrier_destroy(3p)]], [[pthread_barrier_wait(3p)]], [[pthread_barrierattr_destroy(3p)]], [[pthread_barrierattr_getpshared(3p)]]
* [[pthread_cond_destroy(3p)]], [[pthread_cond_timedwait(3p)]], [[pthread_cond_broadcast(3p)]], [[pthread_condattr_destroy(3p)]], [[pthread_condattr_getpshared(3p)]], [[pthread_condattr_getclock(3p)]]
* [[pthread_spin_init(3)]], [[pthread_spin_lock(3)]]
* [[pthread_mutex_destroy(3p)]], [[pthread_mutex_lock(3p)]], [[pthread_mutex_timedlock(3p)]], [[pthread_mutex_getprioceiling(3p)]], [[pthread_mutex_consistent(3)]], [[pthread_mutexattr_init(3)]], [[pthread_mutexattr_getpshared(3)]], [[pthread_mutexattr_gettype(3p)]], [[pthread_mutexattr_setrobust(3)]], [[pthread_mutexattr_getprotocol(3p)]], [[pthread_mutexattr_getprioceiling(3p)]]
* [[pthread_rwlock_destroy(3p)]], [[pthread_rwlock_rdlock(3p)]], [[pthread_rwlock_timedrdlock(3p)]], [[pthread_rwlock_trywrlock(3p)]], [[pthread_rwlock_timedwrlock(3p)]], [[pthread_rwlock_unlock(3p)]], [[pthread_rwlockattr_destroy(3p)]], [[pthread_rwlockattr_getpshared(3p)]], [[pthread_rwlockattr_setkind_np(3)]]
* [[pthread_setconcurrency(3)]], [[pthread_setschedparam(3)]], [[pthread_setschedprio(3)]], [[pthread_setaffinity_np(3)]], [[pthread_yield(3)]]
* [[gettid(2)]], [[tkill(2)]]
* [[membarrier(2)]]

참고: 3p 섹션 페이지들은 기본적으로 인터페이스 명세 문서임. 하지만 다른 페이지들과 어감을 맞추기 위해 "shall"을 "~할 것인다"/"~해야 한다"가 아니라 "~한다"로 번역함.
