---
title: Traffic Test (iperf,locust)
categories: [ETC., Tools]
tags: [Docker, Kali]
image:
    path: 
---





iftop

네트워크 트래픽 양(pps)을 확인하고자 할 경우 `iftop`을 설치하여 실행하면 아래와 같이 inbound/outbound에 대한 각 트래픽 양을 보여준다.
``` bash
                      12.5Kb                 25.0Kb                37.5Kb                 50.0Kb           62.5Kb
└─────────────────────┴──────────────────────┴─────────────────────┴──────────────────────┴──────────────────────
test-ubuntu22                              => 211.34.57.129                               1.36Kb  2.00Kb  2.00Kb
                                           <=                                              160b    456b    456b
test-ubuntu22                              => kns.kornet.net                                 0b    390b    390b
                                           <=                                                0b    700b    700b

─────────────────────────────────────────────────────────────────────────────────────────────────────────────────
TX:             cum:   2.38KB   peak:   3.77Kb                                   rates:   1.36Kb  2.38Kb  2.38Kb 
RX:                    1.13KB           3.05Kb                                             160b   1.13Kb  1.13Kb 
TOTAL:                 3.51KB           6.82Kb                                            1.52Kb  3.51Kb  3.51Kb
```