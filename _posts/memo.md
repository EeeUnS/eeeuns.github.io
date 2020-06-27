

1. open // 

system call is handled by binder_open function.
binder_open allocates binder_proc data structure and assigns it to the filp->private_data


1.  epoll_create // eventpoll 할당, 초기화 그리고 이를 파일에 담고 파일정보를 담은 fd(int)를 반환
2.  epoll_ctl // binder_threade 할당
3.  ioctl // binder_thread 해제
4.  프로그램 종료시에 해제된 binder_thread를 사용하면서 use after free 발생 
