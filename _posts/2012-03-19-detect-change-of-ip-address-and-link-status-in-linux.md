---
layout: post
title: "리눅스 IP 주소 / 링크 상태 변경 여부 감지하기"
tags: [ GLib,  glibc,  Linux,  Network ]
---

리눅스에서 IP 주소가 변경되었거나 링크 상태 변경 여부(예를 들어 랜선이 꽂히거나 빠졌을때)를 자동으로 감지하는 C 코드입니다. `ifconfig` 명령등의 결과를 파싱하는 방법이 아닌 리눅스 커널 `rtnetlink(7)` 프로토콜과 `getifaddrs()` 함수를 이용해 직접 처리합니다. 참조한 소스는 여러군데가 있는데 모두 구글링이 가능하므로 결과물만 기록으로 남겨둡니다.

```c
#include <stdio.h>
#include <string.h>
#include <errno.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <ifaddrs.h>
#include <net/if.h>
#include <netdb.h>
#include <netinet/in.h>
#include <linux/netlink.h>
#include <linux/rtnetlink.h>

static int
create_sock (const char *nic)
{
  struct sockaddr_nl addr;
  int                sock;

  memset (&addr, 0, sizeof (addr));
  addr.nl_family = AF_NETLINK;
  addr.nl_groups = RTMGRP_LINK | RTMGRP_IPV4_IFADDR | RTMGRP_IPV6_IFADDR;

  sock = socket (PF_NETLINK, SOCK_RAW, NETLINK_ROUTE);
  if (sock < 0)
    {
      fprintf (stderr, "failed to open NETLINK_ROUTE socket for %s - %s(%d)",
               nic, strerror (errno), errno);
      return -1;
    }

  if (bind (sock, (struct sockaddr *)&addr, sizeof(addr)) < 0)
    {
      fprintf (stderr, "failed to bind NETLINK_ROUTE socket for %s - %s(%d)",
                 nic, strerror (errno), errno);
      close (sock);
      return -1;
    }

  return sock;
}

static int
ip_changed (int         sock,
            const char *nic)
{
  struct nlmsghdr   *nlh;
  char               buffer[4096];
  int                len;
  int                idx;
  int                found;

  len = recv (sock, buffer, sizeof (buffer), 0);
  if (len <= 0)
    {
      fprintf (stderr, "NETLINK_ROUTE socket recv() failedn");
      return -1;
    }

  found = 0;
  idx = if_nametoindex (nic);

  for (nlh = (struct nlmsghdr *)buffer;
       NLMSG_OK (nlh, len);
       nlh = NLMSG_NEXT (nlh, len))
    {
      if (nlh->nlmsg_type == NLMSG_DONE)
        break;
      if (nlh->nlmsg_type == NLMSG_ERROR)
        continue;
      if (!(NLMSG_OK (nlh, len)))
        continue;

      switch (nlh->nlmsg_type)
        {
        case RTM_NEWADDR:
          {
            struct ifaddrmsg *ifa = (struct ifaddrmsg *)NLMSG_DATA (nlh);

            if (ifa->ifa_index == idx)
              found = 1;
          }
          break;
        case RTM_NEWLINK:
          {
            struct ifinfomsg *ifi = (struct ifinfomsg *)NLMSG_DATA (nlh);

            if (ifi->ifi_index == idx)
              found = 1;
          }
          break;
        default:
          break;
        }
    }

  return found;
}

static int
get_nic_addr (const char     *nic,
              struct ifaddrs *ifaddr,
              int             wanted_family,
              char           *host,
              int             host_len,
              int            *active)
{
  struct ifaddrs *ifa;

  for (ifa = ifaddr; ifa != NULL; ifa = ifa->ifa_next)
    {
      int family;
      int s;

      if (ifa->ifa_addr == NULL)
        continue;

      if (strcmp (ifa->ifa_name, nic))
        continue;

      /* Skip unwanted families. */
      family = ifa->ifa_addr->sa_family;
      if (family != wanted_family)
        continue;

      *active = (ifa->ifa_flags & IFF_RUNNING) ? 1 : 0;

      s = getnameinfo (ifa->ifa_addr,
                       family == AF_INET ? sizeof (struct sockaddr_in) :
                                           sizeof (struct sockaddr_in6),
                       host,
                       host_len,
                       NULL,
                       0,
                       NI_NUMERICHOST);
      if (s != 0)
        {
          fprintf (stderr, "failed to getnameinfo() for '%s - %s(%d)",
                   ifa->ifa_name, strerror (errno), errno);
          continue;
        }

      /* Get the address of only the first network interface card. */
      return 1;
    }

  return 0;
}

static void
print_ip (const char *nic)
{
  struct ifaddrs *ifaddr;
  char            addr[NI_MAXHOST];
  int             active;

  if (getifaddrs (&ifaddr) == -1)
    {
      fprintf (stderr, "failed to getifaddrs() - %s(%d)", strerror (errno), errno);
      return;
    }

  if (!get_nic_addr (nic, ifaddr, AF_INET, addr, sizeof (addr), &active))
    if (!get_nic_addr (nic, ifaddr, AF_INET6, addr, sizeof (addr), &active))
      {
        strcpy (addr, "127.0.0.1");
        active = 0;
      }

  freeifaddrs (ifaddr);

  fprintf (stdout, "%s is %s (link %s)n",
           nic, addr, active ? "active" : "inactive");
}

int
main (void)
{
  char *nic = "eth0";
  int   sock;

  print_ip (nic);

  sock = create_sock (nic);
  if (sock < 0)
    return -1;

  while (1)
    {
      int ret;

      ret = ip_changed (sock, nic);
      if (ret < 0)
        return -1;

      if (ret)
        print_ip (nic);
    }

  close (sock);

  return 0;
}

/*
  Local Variables:
   mode:c
   c-file-style:"gnu"
   indent-tabs-mode:nil
  End:
  vim:autoindent:filetype=c:expandtab:shiftwidth=2:softtabstop=2:tabstop=8
*/
```

참고로 위 소스에서 네트웍 인터페이스 설정 변경을 감지하기 위해 사용한 소켓 파일 디스크립터(socket file descriptor)는 `select()` / `poll()` 등을 이용해 비동기적으로 감시하는 것도 가능합니다. 당연하지만, [GLib 메인루프](/2009/09/21/using-glib-mainloop/)의 `g_io_add_watch()` 등을 이용해도 됩니다.

**[UPDATE 2012-03-21]** `rtnetlink(7)` 프로토콜의 기반이 되는 `netlink(7)` 프로토콜에 대해 더 자세히 알고 싶다면 [Netlink 라이브러리의 Netlink 프로토콜 기초 문서](http://www.infradead.org/~tgr/libnl/doc/core.html#core_netlink_fundamentals)를 참고하기 바랍니다.
