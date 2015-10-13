# Project 2: Sandboxing

## Computer Security Fall 2015

## DUE: Monday 19th Oct at 11:59am KST

이번 과제는 Linux상에서 Sandbox를 만들어보는 과제이다. 이번 과제의 목표는 수업 시간에 학습한 sandbox mechanism을 이해하고 이를 구현해보는 것이다. Sandbox는 내부의 동작을 monitoring하고 restrict하는 isolated environment로서 내부에서 수행되는 application이 시스템에 가하는 피해를 막는데에 목적이 있다. 이 과제에서는 application이 사용하는 system call들을 interpose하는 Sandbox를 만든다. Sandbox안에서 sandboxed application 호출하는 OS와의 모든 interaction들은 system call을 통해 이루어진다. process가 system call을 호출하면, 이는 우선 sandbox에 의해 검사되고 그 결과에 따라 허용 여부가 결정된다. Sandbox가 이를 결정할 동안 sandboxed application은 실행을 멈춘채로 기다린다.

1.  **Systemcall Interposition. (30pts)**

    우리는 대상 process가 사용하는 system call을 interpose하기 위해 Linux에 기본적으로 존재하는 ptrace라는 process tracing facility를 활용한다. ptrace는 외부 process (tracer)가 특정 process (tracee)의 runtime behavior를 trace하고 대상 process (tracee)의 register 및 memory의 값들을 참조하거나 변경할 수 있는 system interface를 제공한다. tracer는 ptrace를 사용하 tracee가 사용하는 모든 system call에 대해 entry와 exit point를 intercept할 수 있다. 아래의 3 step을 따라 ptrace의 기능을 활용하여 대상 process가 실행되는 동안 호출하는 system call을 interpose해보자. (ptrace 사용법: [ptrace manual](http://man7.org/linux/man-pages/man2/ptrace.2.html), [ptrace tutorial](http://www.linuxjournal.com/article/6100?page=0,0), [ptrace guide](http://www.howzatt.demon.co.uk/articles/SimplePTrace.html))

    1.  PTRACE_GETREGS: register value를 읽어 system call number와 argument를 출력해보자. (참고: [Linux system call convention.](http://man7.org/linux/man-pages/man2/syscall.2.html))
    2.  PTRACE_SETREGS: register에 담긴 system call argument를 변경하여 원래와 다른 system call이 불려지도록 해보자.
    3.  PTRACE_POKEDATA: argument가 address인 경우, 해당 pointer가 가리키는 memory address를 overwrite하여 system call이 다른 동작을 하도록 해보자. (Linux 3.2부터는 다른 process로의 data transfer를 위한 [process_vm_readv / process_vm_writev](http://man7.org/linux/man-pages/man2/process_vm_readv.2.html)가 제공된다. 이는 특히 많은 data를 한번에 transfer하는 경우에 ptrace를 통한 memory access보다 효율적이다.)
    
    _Requirements>_
    위의 step 2, 3에 해당하는 기능을 사용해볼 수 있는 commandline interface를 제공한다. 사용자는 option이나 policy file을 input으로 주어 system call을 control할 수 있다. option 및 policy의 형태는 사용자가 편히 사용할 수 있게 정의한다.

    Suggestion: strace tool을 사용해보길 추천한다. strace는 ptrace에 기반한 system call tracing tool로서 다양한 기능들을 제공한다. strace를 간단히 사용해보면 1번 과제의 올바른 결과를 예측해볼 수 있다. (strace: [strace manual](http://man7.org/linux/man-pages/man1/strace.1.html), [strace source code](http://sourceforge.net/projects/strace/))

2.  **Sandboxing. (45pts)**

    Linux의 user-level process들은 kernel interaction을 위해 system call을 사용한다. 특히 다른 process, file 또는 network로 통신을 위해서는 system call을 사용해야 한다. 1번에서 구현한 system call interposition을 활용하면 system call을 원하는대로 control하여 (e.g., whether to allow or disallow, change syscall arguments, change memory space of tracee) 원하는 sandbox를 만들 수 있다. Sandbox가 허용하거나 금지하는 system call의 list는 sandbox의 목적에 달라진다. 이 과제에서 우리가 목표로 하는 sandbox의 property는 아래와 같다.

    _Goal>_ sandboxed process가 외부로 data를 유출할 수 없도록 차단하라.
    
    과제의 목표는 명시적으로 data를 유출하는 system call들을 차단하는 sandbox를 설계 및 구하는 것이다. 다만, data 유출과는 연관이 없는 정상적인 동작을 방해하여 기능성을 저해하지 않도록 주의해야 한다. timing-attack등의 side-channel이나 covert-channel을 사용하는 공격은 가정하지 않는다. 이번 과제에서 차단해야 할 data 유출의 위험이 있는 system call의 유형은 아래와 같다.
    - system calls for filesystem operation (e.g., open(), create(), write())
    - system calls for network (e.g., socket(), connect(), bind(), listen(), accept())
    - system calls for IPC
    - system calls for memory mapping
    - **[Extra credit] - 추가적인 data leakage vector 탐색 및 차단 (10pts)**

    TIP: strace의 _-e trace=xxx_ option을 사용하면 특정 카테고리의 system call 종류들을 확인할 수 있다.

    _Requirements>_
    Sandbox program은 실행시 binary의 path를 input으로 받아 이를 sandbox안에서 수행시킨다. Sandbox program은 sandboxed process가 호출하는 system call 정보를 user가 보기 쉽게 출력한다 (strace 참조). Sandbox program은 user에게 두가지 실행 option을 제공하여 sandboxed application이 허용되지 않은 system call을 호출했을 때에 1) 해당 system call 정보를 출력하고 계속 process를 진행시키거나 2) 정보 출력과 동시에 sandboxed process를 kill하도록 한다. 그 외의 유용한 추가 option들은 자율에 맡긴다.

    **[Extra credit] - seccomp/BPF (10pts)**
    ptrace를 사용한 user-level system call interposition은 context switching 등으로 인해 overhead가 크다는 단점이 있다. Kernel-level system call interposition을 사용하면 보다 효율적인 system call interposition이 가능하다. 특히, 특정 종류의 systemcall을 단순히 allow/disallow하는 것은 kernel-level에 맡기는 것이 보다 효율적이다.
    Kernel-level system call interposition tool로서는 seccomp/BPF가 존재한다. 이는 process가 호출하는 system call로부터 사용자가 정의한 a set of system call을 kernel-level에서 filter하는 역할을 한다. 위의 sandbox에 seccomp을 적용하여, 모든 system call이 아닌 특정 systemcall들만을 interpose하고 context switching로 인한 overhead를 제거함으로써 효율적인 sandbox를 구현해보자. Sandbox가 달성해야 하는 목표는 위와 동일하다. (seccomp의 사용법: [seccomp](seccomp https://www.kernel.org/doc/Documentation/prctl/seccomp_filter.txt), [seccomp guide](http://events.linuxfoundation.org/sites/events/files/slides/limiting_kernel_attack_surface_with_seccomp-LPC_2015-Kerrisk.pdf)) (seccomp를 활용한 system call interposition: [Mbox](https://people.csail.mit.edu/nickolai/papers/kim-mbox.pdf) 논문 참조)

How to submit:
source code는 학번 및 이름을 포함한 20xx-xxxxx_name.tar.gz 로 압축하여 보고서와 함께 sec-tas@cmslab.snu.ac.kr로 제출해주시기 바랍니다. 보고서 및 이메일에 program의 build 및 사용 방법을 명시해주시기 바랍니다. 채점자가 1번과 2번의 기능을 모두 사용해볼 수 있어야 합니다.

Development environment:

채점 환경은 아래 두가지가 준비되어 있으므로, 이 중 하나의 환경을 선택하여 과제를 수행해주시고 결과물 제출 시 선택 환경을 기입해주시기 바랍니다.

- Ubuntu 12.04 LTS 64bit, Kernel: Linux 3.13.0-63-generic
- Ubuntu 14.04 LTS 64bit, Kernel: Linux 3.19.0-25-generic

References:

- Goldberg, Ian, et al. "A secure environment for untrusted helper applications: Confining the wily hacker." Proceedings of the 6th conference on USENIX Security Symposium, Focusing on Applications of Cryptography. Vol. 6\. 1996.
- Kim, Taesoo, and Nickolai Zeldovich. "Practical and Effective Sandboxing for Non-root Users." USENIX Annual Technical Conference. 2013.
