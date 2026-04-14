
/검색할거
해놓고 n 누르면 그 다음거로 넘어간다.


### U-01 root 계정 원격 접속 제한
root가 원격 접속 불가능하도록 설정
/etc/ssh/sshd_config
- 포트를 바꿀 수 있다
- 로그인 설정 변경이 가능하다.

> 9버전 이상 보다는 permit log 옵션이 다른 파일로 바뀌었다.


**- Linux –**
```
‘/etc/securetty’ 파일에서 pts/0 ~ pts/x(숫자) 설정 제거 또는, 주석 처리 후
‘/etc/pam.d/login’ 파일을 수정(없으면 생성) 하여
‘auth required /lib/security/pam_securetty.so’ 라인 추가
#vi /etc/pam.d/login
 auth required /lib/security/pam_securetty.so
```

※ ‘/etc/securetty’ 파일은 기본으로 존재하지 않으므로 /etc 디렉터리 내에 ‘securetty’파일이 존재하지 않는 경우 새로 생성한 후 적용해야 함

### U-02 패스워드 복잡성 설정
**- Linux -**
Fedora & Gentoo & Red Hat의 경우 /etc/pam.d/system-auth 파일에서 아래와 같이 설정
(Ubuntu & Suse & Debian의 경우 /etc/pam.d/common-password)
(대/소문자 중 하나, 숫자/특수문자 중 하나는 선택적으로 적용)
password requisite /lib/security/$ISA/pam_cracklib.so retry=3 minlen=8 **lcredit=-2** ucredit=0 dcredit=0 **ocredit=-1**
- lcredit : 포함되어야 할 알파벳 소문자의 수( ‘-‘ 로 입력하면 최소)
- ucredit : 포함되어야 할 알파벳 대문자의 수( ‘-‘ 로 입력하면 최소)
- dcredit : 포함되어야 할 숫자 수( ‘-‘ 로 입력하면 최소)
- ocredit : 포함되어야 할 특수문자 수( ‘-‘ 로 입력하면 최소)

### U-03 계정 잠금 임계값 설정
**- Linux -**
Fedora & Gentoo & Red Hat의 경우 /etc/pam.d/system-auth 파일에서 아래와 같이 설정 (Ubuntu & Suse & Debian의 경우 /etc/pam.d/common-auth)

**32**비트
auth required pam_tally.so deny=5 unlock_time=120 no_magic_root
account required pam_tally.so no_magic_root reset

**64**비트
auth required pam_tally2.so deny=5 unlock_time=120 no_magic_root
account required pam_tally2.so no_magic_root
- no_magic_root : root에게는 패스워드 잠금 설정을 하지 않음
- deny=5 : 5회 입력 실패 시 패스워드 잠금
- unlock_time : 계정이 잠긴 후 풀릴 때 까지의 시간(초단위)
- reset : 접속시도 성공 시 실패한 횟수 리셋

임계 값 관련 PAM 모듈 설정은 "**auth  sufficient  pam_unix.so**" 설정 위에 존재해야 모듈 설정이 유효함. pam_unix는 인증 관련 모듈로, sufficient 설정 시, 인증 성공 시에는 아래의 모듈을 확인하지 않고 인증을 성공시킴.

※ 접속 실패 횟수 리셋
pam_tally2 –u <계정이름> -r
pam_tally –user <계정이름> --reset


> root는 로그인 실패 시 반드시 재부팅이 필요하므로 root를 잠기게 설정하진 않는다. 일반적으로 아예 패스워드를 안 주고 로그인을 못하게 하거나 등등의 다른 대안을 쓰기도 한다.



### U-04 패스워드 파일 보호
종종 패스워드 하드코딩 된 경우 있지만 지금은 없다.


....

### U-15 world writable 파일 점검

|            |                                                                                                                                                                                    |
| ---------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **취약점 개요** | 모든 사용자가 접근 및 수정할 수 있는 권한으로 설정된 파일이 존재할 경우 일반 사용자의 실수 또는 악의적인 행위로 인해 주요 파일 정보가 노출되거나 시스템 장애를 유발할 수 있음<br><br>의도적으로 변경된 스크립트 파일을 root가 확인하지 않고 실행시켰을 경우 시스템 권한 노출을 비롯해 다양한 보안 위험이 있음 |
| **위협**     | 공격자에 의하여 World-writable 파일 변경                                                                                                                                                      |
|            |                                                                                                                                                                                    |

world writable 파일 존재 여부를 확인하고 불필요한 경우 제거

- 전 시스템 공통 –
```
일반 사용자 쓰기 권한 제거 방법
#chmod o-w <filename>

파일 삭제 방법
rm –rf <filename>
```


....

### U-44 root 이외의 UID가 ‘0’ 금지
UID가 0이면 루트 권한을 갖게 된다. 일반적으론 없음

### U-45 root 계정 su 제한

**- Solaris, HP-UX, Linux-**

su 명령을 실행하기 위한 별도의 그룹 생성
```
#groupadd wheel

su 명령어 사용이 필요한 계정을 wheel 그룹에 추가

#usermod –G wheel <username>

/usr/bin/su 파일의 권한을 변경(4750)

#chmod 4750 /usr/bin/su

/usr/bin/su 의 그룹을 wheel로 변경

#chgrp wheel /usr/bin/su

infraadm 사용자 계정을 wheel 그룹에 넣어준 후, wheel 그룹에게 /usr/bin/su 권한을 부여한다.
chmod 4750 /usr/bin/su
chgrp wheel /usr/bin/su
```




### U-46 패스워드 최소 길이 설정
**- Linux -**
/etc/login.defs 파일에서 PASS_MIN_LEN 값을 수정
```
#vi /etc/login.defs

   PASS_MIN_LEN=8
```

### U-47 패스워드 최대 사용 기간 설정
**- Linux -**
/etc/login.defs 파일에서 PASS_MAX_DAYS 값을 수정(단위: 일)
```

 # vi /etc/login.defs

   PASS_MAX_DAYS 90

```

### U-48 패스워드 최소 사용 기간 설정
...


### U-49 불필요한 계정 제거

**- Solaris, HP-UX, Linux -**

서버에 등록된 불필요한 사용자 계정 확인
userdel 명령으로 불필요한 사용자 계정 삭제
`#userdel <username>`

※ /etc/passwd 파일에서 계정 앞에 `#`을 삽입하여도 주석처리가 되지 않으므로 조치 시에는 반드시 계정을 삭제하도록 권고함.

|                                                                                        |
| -------------------------------------------------------------------------------------- |
| **기본적으로 차단하는 Default 계정**                                                              |
| adm, lp, sync, shutdown, halt, news, uucp, operator, games, gopher, nfsnobody, squid 등 |

### U-50 관리자 그룹에 최소한의 계정 포함
**- Solaris, HP-UX, Linux -**
vi 편집기를 이용하여 “/etc/group” 파일을 연 후
root 그룹에 등록된 불필요한 계정 삭제

> 운영 중에는 발생할 수 있으나 지금은 없다

### U-51 계정이 존재하지 않는 GID 금지
구성원이 존재하지 않는 그룹이 있을 경우 관리자와 검토하여 제거.
`#groupdel <groupname>`

### U-52 동일한 UID 금지
**- Solaris, HP-UX, Linux -**
vi /etc/passwd 명령으로 UID가 중복되지 않도록 변경
같은 방법으로 다음과 같은 명령어를 통해 uid를 변경할 수 있음
`# usermod –u 501 username (username의 UID를 501로 변경)`

### U-53 사용자 shell 점검
로그인이 필요하지 않은 계정에 대해 /bin/false(nologin) 쉘 부여.

vi 편집기를 이용하여 ‘/etc/passwd’ 파일을 연 후
로그인 쉘 부분인 계정 맨 마지막에 /bin/false(nologin) 부여 및 변경

|   |
|---|
|**일반적으로 로그인이 불필요한 계정**|
|daemon, bin, sys, adm, listen, nobody, nobody4, noaccess, diag, listen, operator, games, gopher 등 일반적으로 UID 100이하 60000이상의 시스템 계정이 해당|
> nologin을 주면 ssh 접속 자체가 불가능해진다.
> userdel을 진행해 지우곤 함

### U-54 Session Timeout 설정

**일정 시간이 지날 경우 접속이 끊어질 수 있도록 Timeout을 설정**
**단, 모니터링 용도 계정의 경우 예외(해당 계정 설정에서 600초 이상 입력)**

**(****안행부 취약점 분석평가 기준 600초(10분)이하로 설정)**

**
**- HP-UX, AIX, Linux -**

sh(born shell)와 ksh(korn shell) 에서는 “/etc/profile” 또는 “$HOME/.profile” 파일의 “TMOUT” 값을 설정(초)

`# vi /etc/profile`
`TMOUT = 600`
``  export TMOUT

csh 에서는 “/etc/csh.login 또는 /etc/csh.cshrc” 파일 내에 set autologout=10(분) 을 추가


### U-05 root 홈, 패스 디렉터리 권한 및 패스 설정
### U-06 파일 및 디렉터리 소유자 설정

|                                                                                                                                                                                                                                        |
| -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **-** **전 시스템 공통 -**<br><br>**vi** **편집기를 이용하여 root 계정의 설정파일(~/.profile과 /etc/profile)을 열어**<br><br>**아래와 같이 수정**<br><br>**# vi /etc/profile**<br><br>**PATH=.:$PATH:$HOME/bin (****수정 전)**<br><br>**PATH=$PATH:$HOME/bin (****수정 후)** |

> 운영 중에 발생
> 
### U-07 /etc/passwd 파일 소유자 및 권한 설정


/etc/passwd 파일의 소유자와 권한 변경(소유자 root, 권한 644이하)

# chmod 444 /etc/passwd 혹은

  chmod 644 /etc/passwd

(HP-UX는 400 설정)

소유자를 root로 바꾸어 줌

`# chown root /etc/passwd`

### U-08 /etc/shadow 파일 소유자 및 권한 설정
‘/etc/shadow’ 파일은 시스템에 등록된 모든 계정의 패스워드를 암호화된 형태로 저장, 관리하고 있는 중요 파일로 root 계정을 제외한 모든 사용자의 접근을 제한해야 함

권한 관리가 미흡할 경우 ID와 패스워드 정보가 노출될 수 있는 위험이 있음
```# chown root /etc/shadow

# chmod 400 /etc/shadow```

```


### U-09 /etc/hosts 파일 소유자 및 권한 설정

‘/etc/hosts’ 파일은 IP 주소와 호스트네임을 매핑하는데 사용되는 파일이며 이 파일의 접근권한 설정이 잘못 되어 있을 경우 악의적인 시스템을 신뢰하게 되므로 ‘/etc/hosts’ 파일에 대한 접근권한을 제한하고 있는지 확인

|                                                                                                                                                                   |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ```<br>```**‘/etc/hosts’** **파일의 소유자와 권한 변경 (소유자 root, 권한 600)**<br><br>**-** **전 시스템 공통 -**<br><br>**# chown root /etc/hosts**<br><br>**# chmod 600 /etc/hosts** |

### U-10 /etc/(x)inetd.conf 파일 소유자 및 권한 설정
인터넷 슈퍼데몬 서비스 설정파일인 inetd.conf(xinetd.d) 파일에 대한 접근권한 제한 여부를 확인함

inetd.conf(xinetd.d)의 접근 권한이 잘못 설정되어 있을 경우 비인가자가 악의적인 프로그램을 등록하고 root 권한으로 서비스를 실행시켜 기존 서비스에 영향을 줄 수 있음

```
**- Linux -**

#chown root /etc/xinetd.conf

#chmod 600 /etc/xinetd.conf

※ ‘/etc/xinetd.d/’ 하위에 파일도 위와 동일한 방법으로 조치
```

### U-12 /etc/services 파일 소유자 및 권한 설정

‘/etc/services’ 파일의 소유자 및 권한 변경 (소유자 root, 권한 644)

- 전 시스템 공통 -

```
#chown root /etc/services
#chmod 644 /etc/services
```

### U-13 SUID, SGID, Sticky bit 설정 파일 점검
SUID(Set Uid) 와 SGID(Set Gid)가 설정된 파일(특히, root 소유의 파일인 경우)은 사용자가 특정 명령어를 실행하여 root 권한 획득 및 정상서비스 장애를 발생시킬 수 있으며, 로컬 공격에 많이 이용되므로 보안상 철저한 관리가 필요함

root 소유의 SUID 파일의 경우에는 꼭 필요한 파일을 제외하고 SUID, SGID 속성을 제거해줘야 함

* SUID : 설정된 파일 실행 시, 특정 작업을 수행하기 위해 일시적으로 파일 소유자의 권한을 얻게 됨

* SGID : 설정된 파일 실행 시, 특정 작업 수행을 위하여 일시적으로 파일 소유 그룹의 권한을 얻게 됨


/usr/bin/gpasswd 랑 /usr/sbin/unix_chkpwd는 최근 보안 취약점 권고 대상이 되었기때문에 적용필요



...


### U-18 접속 IP 및 포트 제한
**- Solaris, Linux, AIX -**

vi 편집기를 이용하여 ‘/etc/hosts.deny’ 파일을 열어 아래와 같이 수정 또는 신규 삽입(ALL Deny 설정)

ALL:ALL

vi 편집기를 이용하여 ‘/etc/hosts.allow’ 파일을 열어 아래와 같이 수정 또는 신규 삽입(해당 파일이 없을 경우 생성)

ssh:192.168.0.12, 192.168.0.54 (다른 서비스도 동일한 방식으로 설정)

<TCP Wrapper 접근제어 가능 서비스>

-SYSTAT, FINGER, FTP, TELNET, RLOGIN, RSH, TALK, EXEC, TFTP, SSH

> 보통 ALL로 해두면 다 차단되기 때문에 다른 곳에서 별도로 지정해서 조절하며, 방식은 고객사마다 다르다.


### U-55 hosts.lpd 파일 소유자 및 권한 설정
> 파일 없으면 넘어감


### U-56 UMASK 설정 관리
|            |                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **취약점 개요** | 시스템내에서 사용자가 새로 생성하는 파일의 접근권한은 UMASK 값에 따라 정해짐 <br><br>현재 설정된 UMASK는 명령 프롬프트에서 “umask”를 수행하여 확인할 수 있으며<br><br>UMASK 값이 “027” 또는, “022”이기를 권고<br><br>UMASK 값 “027”은 “rw-r-----“ 접근권한으로 파일이 생성됨.<br><br>UMASK 값 “022”는 “rw-r--r--“ 접근권한으로 파일이 생성됨.<br><br>계정의 Start Profile(/etc/profile, /etc/default/login, .cshrc, .kshrc, .bashrc, .login, .profile  등)에 명령을 추가하면, 사용자가 로그인 한 후에도 변경된 UMASK 값을 적용 받게 되며 잘못 설정된 UMASK 값은 잘못된 권한의 파일을 생성시킴. |

**권고 사항**
**파일 및 디렉터리를 생성할 때 umask 의 값을 타 사용자에게 쓰기 권한을 부여하지 않도록 “022”로 설정**

umask 값을 022로 설정할 경우 디렉터리의 권한과 파일 권한은 다음과 같음

|   |   |
|---|---|
|**디렉터리 권한**|755(drwxr-xr-x)|
|**파일 권한**|644(-rw-r--r--)|

계정의 Start Profile(/etc/profile,/etc/default/login, .cshrc .kshrc .bashrc .login .profile등)에 명령을 추가하면, 유저가 로그인 후에도 변경된 umask값을 적용 받음

**※** **각 환경설정 파일은 위에서부터 아래로 적용되므로 맨 아래에 존재하는 “umask” 설정 확인 필요**


**Linux, HP-UX -**

1. ‘/etc/profile’을 이용한 설정

vi 편집기를 이용하여 ‘/etc/profile’파일을 열어 아래와 같이 수정 또는 신규 삽입

```
umask 022
export umask
```


### U-57 홈 디렉터리 소유자 및 권한 설정

|            |                                                                                                                      |
| ---------- | -------------------------------------------------------------------------------------------------------------------- |
| **취약점 개요** | 사용자 홈 디렉터리 내 설정파일이 비인가자에 의해 변조되면 정상적인 사용자 서비스가 제한됨<br><br>해당 홈 디렉터리의 소유자 외 일반 사용자들이 해당 홈 디렉터리를 수정할 수 없도록 제한하고 있는지 점검 |
사용자별 홈 디렉터리 소유주를 해당 계정으로 변경하고 타 사용자(other)의 쓰기 권한 제거

- 전 시스템 공통 -
```
#chown <username> <user_home_directory>
#chmod o-w <user_home_directory>
```



※ 애플리케이션에서 홈 디렉터리를 업로드 등의 용도로 사용할 수 있기 때문에 권한 변경 시 확인 필요

...

### U-20 Anonymous FTP 비활성화
ftp 있으면 계정 삭제

Anonymous FTP를 사용하지 않는 경우 Anonymous FTP 접속 차단 설정 적용

**-** **전 시스템 공통 -**

1. 일반 FTP

‘/etc/passwd’ 파일에서 ftp 또는 anonymous 계정 삭제

 - Solaris, Linux, HP-UX: userdel 명령 사용

 - AIX: rmuser 명령 사용

2. ProFTP

‘/etc/passwd’ 파일에서 ftp 계정 삭제

- Solaris, Linux, HP-UX: userdel 명령 사용

 - AIX: rmuser 명령 사용

3. vsftp

vsftp 설정파일(/etc/vsftpd/vsftpd.conf 또는 /etc/vsftpd.conf)에서

anonymous_enable=NO 설정


### U-21 r 계열 서비스 비활성화
> r 안 쓰니까 pass


### U-22 cron 파일 소유자 및 권한 설정

|   |   |
|---|---|
|**취약점 개요**|Cron 시스템은 cron.allow 파일과 cron.deny 파일을 통하여 명령어 사용자를 제한할 수 있으며 보안상 해당 파일에 대한 접근제한이 필요함<br><br>cron 접근제한 파일의 권한이 잘못되어 있을 경우 권한을 획득한 사용자가 악의적인 목적으로 임의의 계정을 등록하여 불법적인 예약 파일 실행으로 시스템 피해를 일으킬 수 있음<br><br>- cron을 허용하는 계정의 목록은 cron.allow 파일<br><br>- cron을 허용하지 않는 계정의 목록 cron.deny 파일<br><br>- 만약 cron.allow 파일이 존재하면 cron.deny 파일은 유효성이 없음<br><br>cron. deny 파일은 그 안에 포함된 계정으로 하여금 cron 명령을 수행 하지 못하게 함|
**- Linux -**
‘/etc/cron.allow’, ‘/etc/cron.deny’ 파일의 소유자와 권한 변경

```
#chown root /etc/cron.allow
#chmod 640 /etc/cron.allow
#chown root /etc/cron.deny
#chmod 640 /etc/cron.deny
```

### U-24 NFS 서비스 비활성화

|            |                                                                                   |
| ---------- | --------------------------------------------------------------------------------- |
| **취약점 개요** | NFS(Network File System) 서비스는 root 권한 획득을 가능하게 하는 등 침해사고 위험성이 높으므로 사용하지 않는 경우 중지함 |
**사용하지 않는 NFS 서비스는 중지할 것을 권고**

1. /etc/dfs/dfstab의 모든 공유 제거
2. NFS 데몬(nfsd, statd, mountd) 중지
3. 시동 스크립트 삭제 또는 이름 변경

**- Linux, AIX, HP-UX, Solaris 9****이하 -**
NFS 서비스 데몬 중지(nfsd, statd, mountd)

`#kill -9 [PID]`

> 최초 세팅 시 실행중인 데몬 없음



### U-26 automountd 제거
|   |   |
|---|---|
|**취약점 개요**|automountd 데몬에는 로컬 공격자가 데몬에 RPC(Remote Procedure Call)를 보낼 수 있는 취약점이 존재하여 이를 통해 파일 시스템의 마운트 옵션을 변경하여 root 권한을 획득할 수 있으며, 로컬 공격자가 automountd 프로세스 권한으로 임의의 명령을 실행할 수 있음|

automountd 서비스 비활성화

**- Linux, AIX, HP-UX, Solaris 9****이하 -**

automountd 서비스 데몬 중지

`#kill -9 [PID]`

시스템 재시작 시 automount가 시작되지 않도록 함

(정확한 파일 위치는 O/S 마다 다름)

`# mv /etc/rc2.d/S74autofs /etc/rc2.d/_S74autofs`


※ NFS, Samba 서비스에서 사용 시 automountd 사용 여부 확인이 필요하며, 적용 시 CDROM의 자동 마운트는 이뤄지지 않음


... 넘어가고

### U-30 Sendmail 버전 점검
|            |                                                                                                                  |
| ---------- | ---------------------------------------------------------------------------------------------------------------- |
| **취약점 개요** | Sendmail은 널리 쓰이는 만큼 많은 취약점이 알려져 있어 공격에 목표가 되기 쉬우므로 서버에서 Sendmail을 사용하는 목적을 검토하여 사용할 필요가 없는 경우, 서비스를 제거하는 것이 바람직함 |
| **위협**     | 버퍼 오버플로우(Buffer Overflow) 공격에 의한 시스템 권한 획득과 정보 유출                                                                |
Sendmail 서비스를 사용하지 않을 경우 서비스 중지, 재부팅 후 다시 시작하지 않도록 시작 스크립트 변경, 사용할 경우 패치 관리 정책을 수립하여 주기적으로 패치 적용

sendmail 버전 확인
`# `echo \$Z | /usr/sbin/sendmail -bt -d0`

sendmail 업데이트 참조

http://www.sendmail.com/sm/open_source/download/

ftp://ftp.sendmail.org/pub/sendmail

> 없음

### U-35 웹서비스 디렉터리 리스팅 제거
~~ 웹서비스 안 쓰니 패스

... 어플리케이션 싹 설치하고 나서도 보안권고문 취약점 리스트 따로 적용해주면 된다.
(이는 어플리케이션 담당사에서 진행하게 되는 사항)


### U-65 at 파일 소유자 및 권한 설정

|            |                                                                                                                                                              |
| ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **취약점 개요** | at 명령어 사용자 제한은 at.allow 파일과 at.deny 파일에서 할 수 있으므로 보안상 해당 파일에 대한 접근제한이 필요함<br><br>at 파일에 권한이 잘못되어 있을 경우 권한을 획득한 사용자 계정을 등록하여 불법적인 예약 파일 실행으로 시스템 피해를 발생할 수 있음 |
| **위협**     | 불법적인 예약 파일 실행으로 시스템 피해                                                                                                                                       |
at.allow, at.deny 파일 소유자와 권한 변경 (소유자 root, 권한 640 이하)

**- Solaris, Linux, AIX, HP-UX -**
```
#chown root [at.allow 파일]
#chown root [at.deny 파일]
#chmod 640 [at.allow 파일]
#chmod 640 [at.deny 파일]
```

|                                          |                                               |
| ---------------------------------------- | --------------------------------------------- |
| **OS** **종류별 at.allow, at.deny 설정파일 위치** |                                               |
| **Solaris**                              | /etc/cron.d/at.allow, /etc/cron.d/at.deny     |
| **Linux**                                | /etc/at.allow, /etc/at.deny                   |
| **AIX, HP-UX**                           | /var/adm/cron/at.allow, /var/adm/cron/at.deny |

> 파일 없음


### U-66 SNMP 서비스 구동 점검
|            |                                                                                                                                 |
| ---------- | ------------------------------------------------------------------------------------------------------------------------------- |
| **취약점 개요** | SNMP 서비스는 시스템 상태를 실시간으로 파악하거나 설정하기 위하여 사용하는 서비스이며, SNMP 서비스로 인하여 시스템의 주요 정보 유출 및 정보의 불법수정이 발생할 수 있으므로 SNMP 서비스를 사용하지 않을 경우 중지시킴 |
| **위협**     | 시스템의 주요 정보 유출 및 불법 수정                                                                                                           |

**SNMP** **서비스를 사용하지 않을 경우 서비스를 중지하고, 반드시 필요할 경우  community string 의 복잡성을 강화하도록 함(‘U-68** **SNMP** **서비스 커뮤니티스트링의 복잡성 설정’ 참조)**

**<SNMP** **서비스가 불필요한 경우>**

**- Solaris 9** **이하, Linux -**
```
ls –al /etc/rc.d/rc*.d/* | grep snmp로 검색하여 위치 확인 후 이름 변경
# mv /etc/rc3.d/S76snmpdx /etc/rc3.d/_S76snmpdx
```

**<SNMP** **서비스가 필요한 경우>**

**-** **전 시스템 공통 -**

/etc/snmpd.conf (solaris의 경우 /etc/snmp/conf/snmpd.conf 파일)파일의 내용에서 community string 부분을 추측하기 어려운 이름으로 변경 후 서비스 재구동

* NMS(Network Management System)을 사용하는 경우 해당 NMS 장비에서도 변경한 Community string을 변경해주어야 정상적으로 MIB 값을 수신할 수 있음



> SNMP 안 쓰고 있다.



`ls -al /etc/exports`
> 파일 존재유무 및 권한 확인 방법


## 패치 및 로그 관리
> 실행 사항을 안내하진 않고, 그냥 이런 로그들이 기록된다는 안내사항이다. 한 번 읽어볼 것.




### [추가 작업 필요] newgrp / unix_chkpwd S권한 제거 필요
/usr/bin/newgrp
chmod u-s /usr/bin/newgrp
ls -al /usr/bin/newgrp


ls-al /sbin/unix_chkpwd
chmod u-s /sbin/unix_chkpwd



---

## Tip
- KISA 보안취약점과 KAI에서 다른 사항이 있는지 확인 
- pem.d 설정 항목은 버전 별로 다르고 잘못된 변경 시 기본 기능에 이슈가 갈 수 있음

---

# 학습

#  PAM(Pluggable Authentication Module)
PAM은 리눅스 시스템에서 사용하는 '인증 모듈(Pluggable Authentication Modules)'로써 응용 프로그램(서비스)에 대한 사용자의 사용 권한을 제어하는 모듈이다.

---

PAM( _장착형 인증 모듈_ )은 인증 및 권한 부여를 위한 일반적인 프레임워크입니다. Red Hat Enterprise Linux의 대부분의 시스템 애플리케이션은 인증 및 권한 부여를 위한 기본 PAM 구성에 따라 다릅니다.

## [10.1. PAM 정보](https://docs.redhat.com/ko/documentation/red_hat_enterprise_linux/7/html/system-level_authentication_guide/pluggable_authentication_modules#About_PAM) 

PAM(Pluggable Authentication Modules)은 시스템 애플리케이션이 중앙 집중식으로 구성된 프레임워크에 인증을 릴레이하는 데 사용할 수 있는 중앙 집중식 인증 메커니즘을 제공합니다.

PAM은 다양한 유형의 인증 소스(예: Kerberos, SSSD, NIS 또는 로컬 파일 시스템)에 대한 PAM 모듈이 있으므로 연결할 수 있습니다. 서로 다른 인증 소스에 우선 순위를 지정할 수 있습니다.

이 모듈식 아키텍처는 관리자가 시스템에 대한 인증 정책을 설정할 때 상당한 유연성을 제공합니다. PAM은 다음과 같은 몇 가지 이유로 개발자와 관리자에게 유용한 시스템입니다.
- PAM은 다양한 애플리케이션과 함께 사용할 수 있는 공통 인증 체계를 제공합니다.
- PAM은 시스템 관리자에게 상당한 유연성과 인증에 대한 제어를 제공합니다.
- PAM은 개발자가 자체 인증 체계를 만들 필요 없이 프로그램을 작성할 수 있는 완전한 문서화된 단일 라이브러리를 제공합니다.


---

자체적으로 보안 인증 만들다가 passwd 직접 접근하고 보안침해사고가 생겨 이를 해결하기 위해 PAM이 등장하게 되었으며, PAM의 동작 원리는 다음과 같다.

1. 인증이 필요한 응용프로그램은 더 이상 passwd 파일을 열람하지 않고 ‘PAM’ 모듈에 사용자 인증을 요청한다.
2. PAM은 인증을 요청한 사용자의 정보를 가지고 결과를 도출하여 응용프로그램에 전달한다.

![](https://www.igloo.co.kr/files/2019/12/27/20191227091849be997627-4147-466b-a229-7e602e5b052c.png "%EA%B7%B8%EB%A6%BC25.png")

[그림 1] 응용프로그램 자체적으로 사용자 인증하는 과정

![](https://www.igloo.co.kr/files/2019/12/27/201912270919020ee3f79c-5cfb-4774-8339-daf65d9fa6bd.png "%EA%B7%B8%EB%A6%BC26.png")  
[그림 2] PAM 모듈을 통한 사용자 인증 과정

PAM 모듈은 소프트웨어의 개발과 인증 및 안전한 권한 부여 체계를 분리하고자 하는 목적으로 만들어졌기 때문에 이를 통한 인증을 수행할 경우, 응용프로그램에서 직접 인증 로직을 구현하지 않아 개발이 간소화될 뿐만 아니라 passwd 파일 등 시스템 파일을 열람하지 않아도 되는 장점이 있다. 무엇보다도 가장 큰 장점은 시스템 운영자가 응용프로그램의 인증 동작을 제어할 수 있어 더욱 안전하게 시스템을 운영할 수 있다.

