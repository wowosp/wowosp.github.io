---
layout: post
title:  "[Project] 이온풍 UAM 개발기 - 4만 볼트와 FPU 없는 MCU의 사투"
date:   2023-12-12
categories: [Dev, Project]
tags: [UAM, Arduino, BMS, Troubleshooting, 회로설계]
---

<table width="100%">
  <tr>
    <td align="center" style="border: none;">
      <img src="assets/img/uam-bms/비행체.jpg" alt="비행체" width="100%">
      <br>
      <sub>(위풍당당 비행기)</sub>
    </td>
  </tr>
</table>

대학교 3학년 2학기, 교내 '알파 프로젝트'로 진행했던 **이온풍 추진 UAM(Ion-Wind Thrust UAM)** 개발 과정을 정리해 보려 한다.

이온풍을 이용해 프로펠러 없이 비행체를 띄운다는 목표는 낭만적이었지만, 막상 구현을 시작해보니 회로 설계부터 펌웨어 제어까지 예상치 못한 부분에서 꽤나 많은 삽질이 필요했다.
오늘은 그중에서도 가장 골치 아팠던, **4만 볼트의 고전압 속에서 단 하나뿐인 배터리를 지켜낸 과정**을 기록해 본다.

## 🌧️ 낭만과 현실의 괴리: 40kV와 No FPU

이 프로젝트의 핵심은 **40kV(4만 볼트)**라는 초고전압을 만드는 것이다. 문제는 우리에게 주어진 자원이 너무나 열악했다는 점이다.

1.  **배터리는 딱 하나:** 54셀 리튬 배터리 팩이 단 한 개뿐이었다. 실험 중 실수로 완전 방전(Deep Discharge)되거나 과열되면 즉시 프로젝트 종료다. 예비는 없다.
2.  **MCU는 거북이:** 배터리 보호(BMS)를 위해 온도를 감시해야 하는데, 사용한 아두이노 MCU에는 **FPU(부동소수점 연산 장치)**가 없었다.

### "계산하다가 배터리 터지겠는데요?"
서미스터(온도 센서) 값을 온도로 바꾸려면 복잡한 로그(log) 연산이 필요하다.
FPU도 없는 MCU한테 이 무거운 계산을 시켰더니, 시스템 전체가 버벅(Latency)거리기 시작했다. 0.1초가 급한 고전압 제어 상황에서 이런 딜레이는 치명적이었다. 하드웨어를 바꿀 수도 없는 노릇, 소프트웨어로 어떻게든 뚫어야 했다.

## 🛠️ 삽질의 결과: 머리가 나쁘면 정답지를 주자

결국 "MCU에게 계산을 시키지 말자"는 결론에 도달했다.

### 1. 데이터 분석으로 '족보(Lookup Table)' 만들기
PC에서 Python(`Scikit-Learn`)과 Matlab을 돌려 서미스터 특성 곡선을 미리 분석했다. "저항이 몇 옴이면 몇 도"라는 정답지를 미리 다 계산해서 배열(Array) 형태로 펌웨어에 박아버렸다.

<table>
  <tr>
    <td align="center" width="50%">
      <img src="assets/img/uam-bms/서미스터.png" alt="데이터 분석 및 시뮬레이션 결과" width="100%">
    </td>
    <td align="center" width="50%">
      <img src="assets/img/uam-bms/모니터링.png" alt="모니터링" width="100%">
    </td>
  </tr>
  <tr>
    <td colspan="2" align="center">
      <sub>Python과 Matlab을 활용한 서미스터 온도-저항 특성 분석, 모니터링</sub>
    </td>
  </tr>
</table>

여기에 테이블 사이의 값은 가벼운 정수 연산인 **선형 보간법(Linear Interpolation)**으로 채워 넣었다.
결과는 대성공. 무거운 로그 연산이 사라지니 아두이노가 날아다녔고, 온도가 튀는 즉시 0.01초 만에 전원을 차단할 수 있게 됐다.

### 2. 무거운 변압기 대신 '사다리' 타기
4만 볼트를 만들려면 보통 무거운 변압기를 쓰지만, 날아야 하는 UAM에겐 사치다.
대신 다이오드와 커패시터를 사다리처럼 쌓아 전압을 뻥튀기하는 **Cockcroft-Walton 회로**를 직접 땜질해서 설계했다. 가벼우면서도 40kV를 안정적으로 뽑아내는, 그야말로 경량화의 끝판왕이었다.

<table width="100%" style="border: none;">
  <tr>
    <td width="20%" style="border: none;"></td>
    
    <td width="60%" align="center" style="border: none;">
      <img src="assets/img/uam-bms/회로.png" alt="회로" width="100%">
      <br>
      <sub>(직접 설계한 배전압 회로도와 제작된 고전압 발생 장치)</sub>
    </td>
    
    <td width="20%" style="border: none;"></td>
  </tr>
</table>

## 🚀 마치며

이런 하드웨어 맞춤형 최적화(라고 쓰고 삽질이라 읽는) 덕분에, 프로젝트가 끝날 때까지 배터리는 무사했고 이온풍 비행체는 성공적으로 실험을 마쳤다. (교내 경진대회 수상은 덤이었다.)

단순히 코드를 짜는 것을 넘어, **"내 코드가 하드웨어를 살릴 수도, 죽일 수도 있다"**는 임베디드의 살벌한 매력을 제대로 느낀 프로젝트였다. 

<details>
<summary>마지막은 프로젝트 끝나고 만들었던 포스터 투척! </summary>

<img src="assets/img/uam-bms/포스터.png" alt="포스터" width="100%">

</details>
