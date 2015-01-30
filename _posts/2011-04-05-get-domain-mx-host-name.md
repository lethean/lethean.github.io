---
layout: post
title: "도메인 메일 호스트(MX) 주소 얻기"
tags: [ glibc,  Linux ]
---

예를 들어 nobody@hades.net이라는 메일 주소의 서버는 hades.net인 것 같지만 실제로 메일을 호스팅하는 서버는 해당 도메인 서버에 질의해서 MX 레코드에 기록된 호스트를 찾아야 합니다. 그리고 이 작업을 위해 DNS 관련 프로토콜을 직접 구현하거나. [djbdns](http://cr.yp.to/djbdns.html) 등과 같은 라이브러리를 이용합니다.

그런데, 요즘 기존 코드를 리팩토링하면서 가능한 오래된(?) 라이브러리에 대한 의존성을 없애고 있는데 위에서 설명한 작업을 하는 함수가 리눅스 기본 glibc 라이브러리가 *당연히* 제공하는 걸 알게 되어 잠시 허탈했습니다.

다음은 도메인 이름을 인수로 주면 해당 도메인의 MX 레코드, 즉 메일서버 호스트를 glibc API를 이용해 작성한 코드입니다.

```c
#include <netinet/in.h>
#include <arpa/nameser.h>
#include <resolv.h>

static char *
lookup_mx (const char *name)
{
  unsigned char response[NS_PACKETSZ];  /* big enough, right? */
  ns_msg        handle;
  int           ns_index;
  int           len;

  len = res_search (name, C_IN, T_MX, response, sizeof (response));
  if (len < 0)
    {
      /* failed to search MX records */
      return strdup (name);
    }
  if (ns_initparse (response, len, &handle) < 0)
    {
      /* failed to parse MX records for '%s'", name); */
      return strdup (name);
    }
  len = ns_msg_count (handle, ns_s_an);
  if (len <= 0)
    {
      /* no mx records */
      return strdup (name);
    }
  for (ns_index = 0; ns_index < len; ns_index++)
    {
      ns_rr rr;
      char  dispbuf[4096];

      if (ns_parserr (&handle, ns_s_an, ns_index, &rr))
        {
          /* WARN: ns_parserr failed */
          continue;
        }
      ns_sprintrr (&handle, &rr, NULL, NULL, dispbuf, sizeof (dispbuf));
      if (ns_rr_class (rr) == ns_c_in && ns_rr_type (rr) == ns_t_mx)
        {
          char mxname[MAXDNAME];

          dn_expand (ns_msg_base (handle),
                     ns_msg_base (handle) + ns_msg_size (handle),
                     ns_rr_rdata(rr) + NS_INT16SZ,
                     mxname,
                     sizeof (mxname));
          return strdup (mxname);
        }
    }
  return strdup (name);
}
```

관련 자료는 [Stack Overflow](http://stackoverflow.com/)에서 본 것 같기도 하고... 아무튼, 명색이 전문 리눅스 C 프로그래머로서 15년 넘게 버티고 있으면서도 아직까지도 기본 C 라이브러리가 제공하는 함수도 제대로 알지 못하는 스스로를 돌아보게 됩니다. :(
