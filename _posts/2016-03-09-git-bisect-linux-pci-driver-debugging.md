---
layout: post
title: git bisect 이용한 리눅스 PCI 드라이버 디버깅
tags: [ Git, Linux ]
---

리눅스 커널 4.3 버전으로 업그레이드하면서부터 [RocketRAID][rocketraid] RAID 장치가 동작하지 않았다. 컴파일 오류도 없고 로딩에도 문제가 없는데, 응답이 너무 느려서 거의 동작하지 않는 거나 마찬가지다. 구글링을 해도 해결 방법을 찾을 수 없었는데, 아마도 업체에서 제공하는 커널 드라이버라서 사용자가 적어서일 수도 있다. 이렇게 심각한 문제라면 4.3.x 안정 버전이나 4.4 버전에서 해결될 것이라 생각하고 기다렸다. 하지만 4.4 버전이 출시되어도 증상은 동일했다. 결국 직접 원인을 찾기로 했다. 분명한 건 리눅스 커널 4.2 버전까지 정상적으로 동작했는데 4.3 버전부터 동작하지 않는다는 사실이다.

정상적으로 동작하는 리눅스 커널 4.2.5 버전에서는 `rr272x_1x` 모듈이 인터럽트 16을 할당받아 동작하는데,

```
$ cat /proc/interrupts
...
 16:      13629       3699       2326       1683  IR-IO-APIC  16-fasteoi   ehci_hcd:usb1, rr272x_1x
...

$ dmesg | grep rr272x
...
[    1.619829] rr272x_1x:adapter at PCI 1:0:0, IRQ 16
...
```

오동작하는 리눅스 커널 4.4.1 버전에서는 인터럽트 11을 할당받지만 실제 인터럽트xi는 발생하지 않고,

```
$ cat /proc/interrupts
...
 11:          0          0          0          0  IR-IO-APIC  11-edge      rr272x_1x
 16:     199999          0          1          1  IR-IO-APIC  16-fasteoi   ehci_hcd:usb1
...

$ dmesg | grep rr272x
...
[    1.639672] rr272x_1x:adapter at PCI 1:0:0, IRQ 11
...
```

인터럽트 16을 아무도 처리하지 않아서 커널이 경고 메시지를 출력한다.

```
[    5.750969] irq 16: nobody cared (try booting with the "irqpoll" option)
[    5.750973] CPU: 0 PID: 0 Comm: swapper/0 Tainted: G     U     O    4.4.1-1-ARCH #1
[    5.750974] Hardware name: Supermicro C7Z97-OCE/C7Z97-OCE, BIOS 2.0 06/22/2015
[    5.750975]  0000000000000000 bf85cf29362dfeed ffff880856c03e68 ffffffff812c7f39
[    5.750977]  ffff88082e06ae00 ffff880856c03e90 ffffffff810d1053 ffff88082e06ae00
[    5.750978]  0000000000000000 0000000000000010 ffff880856c03ec8 ffffffff810d13de
[    5.750980] Call Trace:
[    5.750981]  <IRQ>  [<ffffffff812c7f39>] dump_stack+0x4b/0x72
[    5.750988]  [<ffffffff810d1053>] __report_bad_irq+0x33/0xc0
[    5.750990]  [<ffffffff810d13de>] note_interrupt+0x23e/0x280
[    5.750992]  [<ffffffff810ce6d7>] handle_irq_event_percpu+0xa7/0x1b0
[    5.750994]  [<ffffffff810ce819>] handle_irq_event+0x39/0x60
[    5.750995]  [<ffffffff810d1be9>] handle_fasteoi_irq+0x89/0x150
[    5.750997]  [<ffffffff81018c7a>] handle_irq+0x1a/0x30
[    5.751000]  [<ffffffff815944db>] do_IRQ+0x4b/0xd0
[    5.751002]  [<ffffffff815925c2>] common_interrupt+0x82/0x82
[    5.751002]  <EOI>  [<ffffffff81447c14>] ? cpuidle_enter_state+0x124/0x290
[    5.751006]  [<ffffffff81447db7>] cpuidle_enter+0x17/0x20
[    5.751008]  [<ffffffff810b79b2>] call_cpuidle+0x32/0x60
[    5.751010]  [<ffffffff81447d93>] ? cpuidle_select+0x13/0x20
[    5.751012]  [<ffffffff810b7c72>] cpu_startup_entry+0x292/0x370
[    5.751014]  [<ffffffff815858c9>] rest_init+0x89/0x90
[    5.751015]  [<ffffffff81906013>] start_kernel+0x483/0x4a4
[    5.751017]  [<ffffffff81905120>] ? early_idt_handler_array+0x120/0x120
[    5.751019]  [<ffffffff81905339>] x86_64_start_reservations+0x2a/0x2c
[    5.751020]  [<ffffffff81905485>] x86_64_start_kernel+0x14a/0x16d
[    5.751021] handlers:
[    5.751025] [<ffffffffa0093410>] usb_hcd_irq [usbcore]
[    5.751026] Disabling IRQ #16
```

인터럽트 번호가 잘못 할당된 것 같은데 원인을 찾아야 했다. 결국 무식하지만 가장 확실한 방법을 사용하기로 결심하고, `git bisect` 명령을 이용해 리눅스 커널 4.2 버전과 4.3 버전 사이에서 문제를 일으키는 커밋을 찾아야 했다. 물론 매번 커널을 다시 컴파일하고 테스트하는 작업은 오래 걸리지만 다행히 성능 좋은 개발 서버에서 돌려서 시간을 많이 단축할 수 있었고 약 하루 정도 만에 문제의 커밋을 찾았다.

```sh

$ git bisect start 6a13feb9c82803e2b815eca72fa7a9f5561d7861 64291f7db5bd8150a74ad2036f1037e6a0428df2
$ git bisect log
# bad: [6a13feb9c82803e2b815eca72fa7a9f5561d7861] Linux 4.3
# good: [64291f7db5bd8150a74ad2036f1037e6a0428df2] Linux 4.2
git bisect start '6a13feb9c82803e2b815eca72fa7a9f5561d7861' '64291f7db5bd8150a74ad2036f1037e6a0428df2'
# bad: [807249d3ada1ff28a47c4054ca4edd479421b671] Merge branch 'upstream' of git://git.linux-mips.org/pub/scm/ralf/upstream-linus
git bisect bad 807249d3ada1ff28a47c4054ca4edd479421b671
# bad: [102178108e2246cb4b329d3fb7872cd3d7120205] Merge tag 'armsoc-drivers' of git://git.kernel.org/pub/scm/linux/kernel/git/arm/arm-soc
git bisect bad 102178108e2246cb4b329d3fb7872cd3d7120205
# good: [c8192ba416397ad6ce493f186da40767ce086c3b] Merge tag 'for-v4.3' of git://git.kernel.org/pub/scm/linux/kernel/git/sre/linux-power-supply
git bisect good c8192ba416397ad6ce493f186da40767ce086c3b
# bad: [7073bc66126e3ab742cce9416ad6b4be8b03c4f7] Merge branch 'core-rcu-for-linus' of git://git.kernel.org/pub/scm/linux/kernel/git/tip/tip
git bisect bad 7073bc66126e3ab742cce9416ad6b4be8b03c4f7
# bad: [26f8b7edc9eab56638274f5db90848a6df602081] Merge tag 'pci-v4.3-changes' of git://git.kernel.org/pub/scm/linux/kernel/git/helgaas/pci
git bisect bad 26f8b7edc9eab56638274f5db90848a6df602081
# good: [cf9d615f7f5842ca1ef0f28ed9f67a97d20cf6fc] Merge tag 'regulator-v4.3' of git://git.kernel.org/pub/scm/linux/kernel/git/broonie/regulator
git bisect good cf9d615f7f5842ca1ef0f28ed9f67a97d20cf6fc
# good: [edc837da4b54a01ba6fa3c29b411e35d1a8430ca] Merge tag 'leds_for_4.3' of git://git.kernel.org/pub/scm/linux/kernel/git/j.anaszewski/linux-leds
git bisect good edc837da4b54a01ba6fa3c29b411e35d1a8430ca
# bad: [5a4f3cf0d1f02884c0a64488d22b3bb4bce31b44] Merge branches 'pci/irq', 'pci/misc', 'pci/resource' and 'pci/virtualization' into next
git bisect bad 5a4f3cf0d1f02884c0a64488d22b3bb4bce31b44
# good: [932c435caba8a2ce473a91753bad0173269ef334] PCI: Add dev_flags bit to access VPD through function 0
git bisect good 932c435caba8a2ce473a91753bad0173269ef334
# good: [cd66d5c3df7c96cbf75010b964b94032ceca8889] Merge branches 'pci/host-designware', 'pci/host-xgene' and 'pci/host-xilinx' into next
git bisect good cd66d5c3df7c96cbf75010b964b94032ceca8889
# bad: [5f2269916b0e509f2926346b58209abfa8316143] PCI/MSI: Free legacy IRQ when enabling MSI/MSI-X
git bisect bad 5f2269916b0e509f2926346b58209abfa8316143
# bad: [991de2e59090e55c65a7f59a049142e3c480f7bd] PCI, x86: Implement pcibios_alloc_irq() and pcibios_free_irq()
git bisect bad 991de2e59090e55c65a7f59a049142e3c480f7bd
# good: [890e4847587fcff5eb0438e90992ad7d2a261f33] PCI: Add pcibios_alloc_irq() and pcibios_free_irq()
git bisect good 890e4847587fcff5eb0438e90992ad7d2a261f33
# first bad commit: [991de2e59090e55c65a7f59a049142e3c480f7bd] PCI, x86: Implement pcibios_alloc_irq() and pcibios_free_irq()
```

바로 이 커밋이 원인이었다.

```
commit 991de2e59090e55c65a7f59a049142e3c480f7bd
Author: Jiang Liu <jiang.liu@linux.intel.com>
Date:   Wed Jun 10 16:54:59 2015 +0800

    PCI, x86: Implement pcibios_alloc_irq() and pcibios_free_irq()

    To support IOAPIC hotplug, we need to allocate PCI IRQ resources on demand
    and free them when not used anymore.

    Implement pcibios_alloc_irq() and pcibios_free_irq() to dynamically
    allocate and free PCI IRQs.

    Remove mp_should_keep_irq(), which is no longer used.

    [bhelgaas: changelog]
    Signed-off-by: Jiang Liu <jiang.liu@linux.intel.com>
    Signed-off-by: Bjorn Helgaas <bhelgaas@google.com>
    Acked-by: Thomas Gleixner <tglx@linutronix.de>

:040000 040000 765e2d5232d53247ec260b34b51589c3bccb36ae f680234a27685e94b1a35ae2a7218f8eafa9071a M	arch
:040000 040000 d55a682bcde72682e883365e88ad1df6186fd54d f82c470a04a6845fcf5e0aa934512c75628f798d M	drivers
```

이 커밋에 대한 자세한 정보를 얻기 위해 인터넷을 검색하니, 약간 허무하지만, 동일한 문제가 이미 보고되어 있었다.

* [PCI device driver broken between 4.2 and 4.3][bug-ml]  
* [TA1-PCI card worked in v4.2, fails in v4.3][bug-bz]

해당 [버그질라][bug-bz]에 간단한 댓글을 달면서 공식 해결 방법을 기다렸지만, 공식 커널에 적용되려면 너무 시간이 오래 걸릴 것 같아서 직접 드라이버 [패치][bug-patch]를 만들어 적용했더니 문제없이 동작했다. 일종의 편법이긴 하지만, `pci_register_driver()` 함수를 미리 호출한 다음에 `pci_get_device()` 함수를 이용하는 원래 초기화 함수를 호출하도록 변경했다.

그런데 조금 성급했는지, 아니면 심각한 버그라고 판단했는지 리눅스 4.5 RC 버전에 바로 [반영][bug-commit]되었다.

```
commit 6c777e8799a93e3bdb67bec622429e1b48dc90fb
Author: Bjorn Helgaas <bhelgaas@google.com>
Date:   Wed Feb 17 12:26:42 2016 -0600

    Revert "PCI, x86: Implement pcibios_alloc_irq() and pcibios_free_irq()"

    991de2e59090 ("PCI, x86: Implement pcibios_alloc_irq() and
    pcibios_free_irq()") appeared in v4.3 and helps support IOAPIC hotplug.

    Олег reported that the Elcus-1553 TA1-PCI driver worked in v4.2 but not
    v4.3 and bisected it to 991de2e59090.  Sunjin reported that the RocketRAID
    272x driver worked in v4.2 but not v4.3.  In both cases booting with
    "pci=routirq" is a workaround.

    I think the problem is that after 991de2e59090, we no longer call
    pcibios_enable_irq() for upstream bridges.  Prior to 991de2e59090, when a
    driver called pci_enable_device(), we recursively called
    pcibios_enable_irq() for upstream bridges via pci_enable_bridge().

    After 991de2e59090, we call pcibios_enable_irq() from pci_device_probe()
    instead of the pci_enable_device() path, which does *not* call
    pcibios_enable_irq() for upstream bridges.

    Revert 991de2e59090 to fix these driver regressions.

    Link: https://bugzilla.kernel.org/show_bug.cgi?id=111211
    Fixes: 991de2e59090 ("PCI, x86: Implement pcibios_alloc_irq() and pcibios_free_irq()")
    Reported-and-tested-by: Олег Мороз <oleg.moroz@mcc.vniiem.ru>
    Reported-by: Sunjin Yang <fan4326@gmail.com>
    Signed-off-by: Bjorn Helgaas <bhelgaas@google.com>
    Acked-by: Rafael J. Wysocki <rafael@kernel.org>
    CC: Jiang Liu <jiang.liu@linux.intel.com>

 arch/x86/include/asm/pci_x86.h |  2 ++
 arch/x86/pci/common.c          | 20 +++++++++++---------
 arch/x86/pci/intel_mid_pci.c   |  7 ++-----
 arch/x86/pci/irq.c             | 15 ++++++++++++++-
 drivers/acpi/pci_irq.c         |  9 ++++++++-
 5 files changed, 37 insertions(+), 16 deletions(-)
```

참 좋은 세상이다 ;)

[rocketraid]: http://www.highpoint-tech.com/USA_new/series_rr272x_configuration.htm
[bug-ml]: http://www.spinics.net/lists/linux-pci/msg48507.html
[bug-bz]: https://bugzilla.kernel.org/show_bug.cgi?id=111211
[bug-patch]: https://gist.github.com/lethean/e2e53ca15b1c97cee45a
[bug-commit]: https://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/commit/?id=6c777e8799a93e3bdb67bec622429e1b48dc90fb
