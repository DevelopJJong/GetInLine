# 통합 온라인 예약 장소 줄서기 프로젝트


## 개발 환경

* Intellij IDEA Ultimate 2022.1.4
* Java 17
* Spring Boot 3.0.4


## 기술 스택

* Spring Boot
* Spring Boot Devtools
* Spring Web
* Spring Data JPA
* Spring Security
* Spring HATEOAS
* Rest Repositories HAL Explorer
* Thymeleaf
* Lombok
* Ant Design
* Tailwind CSS
* Heroku


## 기획

### 줄서기 서비스의 기본 개요

* 일반 적인 사업자의 서비스 기업은 예약 시스템을 갖춘 경우가 많음
* 그러나 공공 기관은 예약 시스템을 갖추지 않고 현장에서 처리하는 경우가 있음
* 코로나로 모든 공공 시설에 입장 인원 제한 정책이 생기면서, 현장에서 입장 인원을 수기 처리
* 방문객은 현재 원하는 시설에 입장 인원 현황이 어떤지, 인원이 차서 입장이 불가능한지를 알 수 없음
* 현장에 가야 확인이 되고, 지인의 연락을 통해 현황을 전달받는 정도
* 이 불편을 해소한다.

### 예약 vs 줄서기

* 예약이란 스케쥴 + 회원제이다. 누가 언제 예약하는지가 핵심이다.
* 줄서기란 현장 + 익명이다. 미리 줄설 수 없으며 누가 줄섰는지 중요하지 않다.
* 이 차이를 구현에서 분명하게 구분할 것 -> 서비스의 캐릭터가 됨

### 서비스 핵심 가치

* 장소 관리자가 줄서기 현황을 사용자들에게 더 쉽고 빠르게 공유해주는 것이 목표
* 현재 이 서비스 기획에서 “줄서기”란 "줄세우기" 나 “입장객수 카운트" 와 동일한 의미
* 워딩을 “줄서기” 로 한 이유는
  * 좀 더 친숙하고 피동적이지 않은, 거부감이 덜한 이름 선택
  * 나중에 능동적인 줄서기 서비스 (회원 + 예약) 로 확장을 고려
* 타겟 관리자는 IT 서비스에 친숙하지 않은 사람을 커버한다.
* 다른 기능은 몰라도 현장에서 사용하는 기능의 UX는 가능한 한 단순해야 한다. 관리자의 머리를 아프게 하는 요소가 조금만 있어도 서비스를 이용하지 않을 것
  * 줄서기 이벤트 열기 (빠른 이벤트)
  * 줄서기, 줄서기 취소
  * 줄서기 이벤트 닫기
* 관리자는 줄서기 핵심 정보를 가능한 한 단순하고 확실하게 파악할 수 있어야 한다.
  * 현재 이벤트 상태
  * 줄 선 사람 인원수
  * 이벤트 정원 등

### 도메인 개념

#### 관리자

* 장소를 관리하고 줄서기 이벤트를 열 수 있는 사람
* 1 관리자는 N 장소를 가진다.

#### 장소

* 줄서기 이벤트를 열고 줄서기를 받을 수 있는 시설
* 1 장소는 N 줄서기 이벤트를 가진다. (N <= 동일한 일시, 기간의 줄서기 이벤트들의 최대 참석 가능 인원의 합)
* 1 장소는 N 관리자를 둔다.
* 1 장소는 N 줄서기 이벤트를 가진다.

#### 관리자 장소 매핑

* 관리자 : 장소 = N : M 매핑 테이블

#### 줄서기 이벤트

* 단위 줄서기 이벤트
* 열림, 진행중 상태의 이벤트는 줄서기가 가능하다. 종료, 취소된 이벤트는 줄서기 불가.
* 동일 이벤트 일시와 기간에도 N개의 줄서기 이벤트를 만들 수 있으나, 동 시간대 이벤트들 총 입장 인원 합이 장소의 최대 수용 인원을 넘으면 안된다.
* 이벤트 작성 방법은 편의성 극대화를 위해 일반 이벤트, 빠른 이벤트로 나눈다.
  * 일반 이벤트: 이벤트 작성 페이지를 제공. 날짜, 기간, 정원 정보를 넣고 확인한다.
  * 빠른 이벤트: 원터치로 이벤트 생성. 클릭 당시 시간과 기본값 정원 정보로 이벤트 생성
    * 이벤트의 기간, 종료 등 관련 설정을 가능한한 자동으로 세팅하고 처리

### 서비스 UI

* PC, 모바일에 대응 (반응형)
* 메인 페이지(비로그인): 장소, 줄서기 이벤트, 관리자 로그인
  * 장소: 장소들 조회 가능
  * 줄서기 이벤트: 현재 열린 줄서기 이벤트와 정보 조회 가능
* 메인 페이지(관리자 로그인): 장소, 줄서기 이벤트, 설정 (my page), 로그아웃
  * 장소: 관리자가 관리하는 장소 조회
  * 줄서기 이벤트: 현재 관리자가 만든 이벤트 조회, 생성, 관리


## 도메인 설계

* 관리자: 닉네임, 비밀번호, 이메일, 전화번호
* 장소: 장소 종류(체육관, 음식 등), 장소 이름, 주소, 전화번호, 최대 수용 인원, 설명
* 관리자 장소 매핑: 관리자, 장소
* 줄서기 이벤트: 이벤트 상태(열림, 진행중, 완료, 취소), 이벤트 일시, 기간, 현재 인원, 최대 참석 가능 인원, 메모
* 공통 메타데이터: 생성일시, 수정일시, 생성자, 수정자


## api

* 관리자 조회, 등록, 수정, 삭제
  * 관리자 가입, 마이 페이지 정보 조회, 정보 수정, 탈퇴, ID 찾기, 비번 초기화
* 장소 조회, 등록, 수정, 삭제
  * 관리자 장소 목록 조회, 장소 상세 정보 확인
* 줄서기 이벤트 조회, 등록, 수정
  * 줄서기, 줄서기 취소, 줄서기 세부 정보 (현황) 조회, 줄서기 가능한 이벤트 목록 조회
  * 줄서기 이벤트는 기본적으로 삭제하지 않고 상태를 바꾸며 누적 (종료, 취소)
    * 나중에는 히스토리 테이블을 분리하는 것도 고려 가능


## 뷰

* 시작 페이지, 메인 페이지, 관리자 마이 페이지, 장소 페이지, 줄서기 이벤트 페이지