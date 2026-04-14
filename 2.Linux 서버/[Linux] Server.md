## 설치
- ISO 파일 가져가서 설치합니다. (현재 실습 환경에서는 선택하면 자동 설치되니 제외)
- 실행 환경 Link : https://192.168.10.15/
- administrator@vsphere.local / Iworks@1qw2#ER$


# 기본 구조와 원리
## 커널 Kernel
하드웨어 장치 지원 여부에 관한 정보
하드웨어 성능
하드웨어 제어 코드
- Prepatch : 
	- 리누스 토르발트 직접 관리. RC 버전 (Release Candidate) 이라고도 부름
	- 신기능 포함되나 안정성 떨어질 수 있음.
- Mainline:
	- 실제 운영 버전. Prepatch 구현 신기능을 안정적으로 사용 가능
	- 9~10주 간격으로 발표됨
- Stable:
	- Mainline 대부분의 버그가 잡힌 안정화된 커널
	- 1주 간격 발표. 실사용에 큰 문제가 없음
- Longterm:
	- 장기 지원 (Longterm manintenance) 커널
	- 장기간 업데이트 지원


## 리눅스 배포판 종류
- Red Hat Linux:
	- 2019년 IBM에 인수됨
	- GPL 라이선스를 따라야 해서 소스 코드 공개되어 있음
		- 공개된 소스코드를 로고 변경 후 컴파일한 클론 리눅스가 있음
		- 클론 리눅스 AlmaLinux, Rocky Linux... CentOS는 8버전까지
- Fedora Linux:
	- Red Hat 후원하는 페도라 프로젝트
	- 최근에는 Red Hat의 베타 버전 성격이 강함
- CentOS (센토스):
	- 레드햇 클론 리눅스
	- RedHat 본사의 문제 해결 기술 지원을 받을 수 없음
	- 8버전 이후로 만들어지지 않을 예정이며, CentOS Stream으로 전환.
		- 이는 RHEL 포함 예정인 실험적 기능을 테스트하는 리눅스
- Rocky Linux
	- 레드햇 클론 리눅스. CentOS 원년 개발자 중 한명인 그레고리 커처가 CentOS를 대체하는 리눅스 개발 프로젝트를 진행함


> **리눅스 간 관계**
> fedora에서 실험 신기능 반영해 제작 후 Red Hat에 안정화 반영. 이렇게 제작된 Red Hat의 소스 코드를 그대로 재컴파일 해서 Rocky Linux가 제작된다. CentOS Stream은 RHEL 업그레이드를 위해 출시 이후에 신기능이  추가된 베타 버전으로 개발된다.


> **Rocky Linux 9 하드웨어 요구사항**
> CPU : 64bit, 하드디스크 여유 공간 20GB 이상, 권장 메모리 4GB 이상 (최소 2)


## Partitioning
Patition 요구사항에 맞춰 정의
root 아래에 mount된 내역을 확인한다.

디스크 단위 식별자
- sda
- sdb
- sdc

![[Pasted image 20260410111310.png]]


![[Pasted image 20260410131301.png]]

