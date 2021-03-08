---
layout: post
title: "[DB] Disk Drive"
subtitle: "[DB] Disk Drive"
categories: DB DBMS Disk HDD SSD
tags: Python Simulation
comments: true


---

DBMS는 `디스크기반`에서 `인메모리 기반`으로 발전함.
DBMS의 기능을 알기 위해서는 disk drive를 알아둬야 함.

## 디스크란?

Secondary Storage 역할을 함.

- 1차 저장장치는 "휘발성" 메모리인 RAM
- 디스크는 Secondary Storage로 장기간의 영구적 저장공간으로 이용된다.

디스크 내 R/W

- Read : 데이터를 디스크에서 RAM(메인 메모리)으로 전달
- WRITE : 데이터를 RAM에서 디스크로 전달

모든 데이터를 RAM에 올리지 않는 이유

- RAM 은 volatile함. (저장이 안됨)
- 비용이 많이 든다. (128MB of RAM == 15GB of disk)

Tape storage와 비교했을때의 장점

- 랜덤 액세스를 통한 신속한 백업 및 복구

## 디스크 구조

![disk_drive1](https://yunsikus.github.io/assets/img/post_img/disk_drive1.jpg)

- 헤드가 플래터의 표면에 움직이면서 데이터를 쓰거나 읽어옴

    → 데이터의 location이 dbms 성능에 영향을 끼침.

- 트랙 : 플래터 표면 내에 동심원에 데이터를 저장하는 공간
- 섹터 : 트랙의 일부를 피자조각처럼 쪼갠 가장 작은 단위(보통 512byte)

`ZBR(Zone Bit Recording)`이란?

- 예전에는 트랙 당 섹터 수가 고정되어 있었음. 그래서 안쪽 트랙과 바깥쪽 트렉의 섹터 수가 동일하여 공간의 낭비가 심했음. 이를 개선하여 바깥쪽 트랙으로 갈수록 트랙 당 섹터 수를 늘리는 기법이 ZBR.

## Access time은 다음 3가지로 구성됨

`탐색시간(seek time)` : read/write 헤드를 데이터가 저장되어 있는 트랙위치로 이동시키는데 소요되는 시간

`회전지연시간(rotational delay time)`: read/write 헤드를 데이터가 위치하는 트랙으로 이동시킨 순간부터 원하는 섹터에 헤드가  다다를때까지 소요되는 시간.

`전송시간(transfer time)` : reas/write 헤드가 찾은 데이터를 실제로 디스크로부터 사용자의 버퍼로 보내지는데 소요되는 시간

그 중에서도 `seek time` 과 `rotational delay time` 을 줄이는 것이 핵심.

→ 작은 지름의 디스크를 사용하여 head가 이동하는 거리를 줄일 수 있다. (seek time 개선)

→ rotational delay time은 RPM을 늘리는 방법이 있다. (rotational delay time 개선)

## Disk 내 Cache

`Disk cache`란

디스크로부터 읽은 데이터를 보관해두는 메모리 안의 영역. 이후에 같은 데이터를 읽어야할 경우가 생기면, 실제의 디스크가 아닌 디스크 캐시의 내용을 읽으면 된다.

`Read ahead cache`란

I/O 와 CPU 연산을 동시에 수행해 그 수행 성능을 최대로 만들어 보겠다라는 최적화 알고리즘. 상식적으로 읽고 연산 해야 하지만, "미리 쓰일것 같아 먼저 읽어 두겠다"는 것. 그로 인해 연산을 먼저 시작할 수 있으므로 성능이 더 나빠질수도 있음.

`Command Queing`이란

명령어 대기열로 주어진 작업을 모두 처리한 다음 명령을 받아들이면, 대기시간이 길어지다 보니, 하드디스크에 미리 명령을 쌓아 놓았다가 바로 다음 명령을 수행하는 기술로 작업 시간을 줄일 수 있다. CQ를 확장한 NCQ(Native Command Queing)와 TCQ(Tagged Command Queue)가 있다.

엘리베이터가 층을 움직일 때 최소 경로를 찾듯 Disk도 여러 커맨드를 입력 받았을때 지연시간을 가장 줄일 수 있는 순서를 찾는 알고리즘이 있다.

## Read & Write Processed Differently

read write는 상당히 다르게 다뤄짐

`Read requests` →동기(작업의 흐름이 순차적으로 진행되며 블록킹 방식이기 때문에 request가 끝날때까지 다른 작업을 동시에 진행할 수 없다)

→ Read latency는 performance에 영향을 끼친다.

`write requests` → 비동기

## Bandwidth vs Latency

`Bandwidth` : Data Transfer Rate

`Latency` : seek and rotation time

disk technology의 개발이 bandwidth가 2배 발전할때 latency는 1.2 ~ 1.4배정도 발전해서 gap 이 벌어짐. latency는 동기처리(read)에서 성능에 영향을 많이 주므로 문제가 됨.
