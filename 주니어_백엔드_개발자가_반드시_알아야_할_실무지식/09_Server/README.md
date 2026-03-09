
## 서버
개발자는 코드를 작성한 이후 서버에 애플리케이션을 배포한 후, 서비스가 안정적으로 운영되도록 모니터링하고 장애에 대응해야 한다. 이 과정에서 **_서버에 대한 기본적인 이해도가 낮다면, 문제가 발생했을 때 원인을 찾고 해결하는 데 큰 어려움_** 을 겪게 된다. <br/>
가장 큰 어려움은 장애가 발생하였는데 애플리케이션에 아무러 로그가 남아 있지 않은 경우다. 이때는 서버를 직접 들여다보고 문제를 찾아내야한다. <br/>
예를 들어 메모리가 부족해서 OS가 앱을 강제 종료했거나, 디스크 풀이 발생하거나, 다른 서버와의 네트워크 문제가 있을 수 있다. <br/>
때문에 개발자는 서버 자원과 프로세스, 네트워크, 로그 등 서버의 기본적인 요소들을 이해하고 장애 상황에서 원인을 찾을 수 있는 능력을 갖춰야 한다. <br/>


## OS 계정과 권한
OS는 **_사용자 계정과 권한으로 파일과 프로세스에 접근을 제어_** 한다. 계정과 권한을 최소화하는 것은 보안적인 측면에서 중요하지만, 이로 인해서 애플리케이션의 배포나 실행 오류가 발생할 수 있다. <br/>
뿐만 아니라 애플리케이션이 실행 도중 파일을 읽거나 쓰는 과정에서 권한 문제로 인해 장애가 발생할 수 있다.

> OS 계정과 권한 문제로 인해서 "로컬에서는 됐는데 서버에서 안 돼요"라는 말이 자주 나온다. 이때 OS 계정과 권한에 대한 이해가 없으면, 원인을 찾는데 시간이 오래 걸리고, 잘못된 조치를 취할 수 있다. <br/>
> 예를 들어 문제를 해결하기 위해서 `root` 계정에 대한 권한을 부여하여 문제를 해결했다고 생각할 수 있지만, 이로 인해서 서버 전체에 심각한 보안 위험을 야기시킬 수 있다.


### 파일 권한

파일에 대한 권한은 소유자, 그룹, 기타로 나뉘며, 각 사용자 유형에 대해 읽기(r), 쓰기(w), 실행(x) 권한이 있다. 파일 권한은 `ls -al` 명령어로 확인할 수 있다. 

```bash
$ ls -al /opt/myapp/
drwxr-xr-x 3 root    root     4096  Mar 8 12:00 .
-rwxr-xr-- 1 appuser appgroup 12345 Mar 8 12:01 app.jar
```

가장 왼쪽의 `-rwxr-xr--`가 권한을 나타내며, 각 권한은 3자리로 나뉜다. 첫 번째는 소유자(appuser), 두 번째는 그룹(appgroup), 세 번째는 기타 사용자에 대한 권한이다. 각 자리의 rwx는 읽기, 쓰기, 실행 권한을 나타낸다.

```
-  rwx  r-x  r--
타입 소유자 그룹 기타
```

| 권한        | 숫자 | 의미 |
| ----------- | ---- | ---- |
| r (read)    | 4    | 읽기 |
| w (write)   | 2    | 쓰기 |
| x (execute) | 1    | 실행 |

`chown(Change Owner)` 명령어를 사용하여 **_파일의 소유자와 그룹을 변경_** 할 수 있다. 예를 들어 `chown appuser:appgroup app.jar`는 파일의 소유자를 `appuser`로, 그룹을 `appgroup`으로 변경한다.
`chmod(Change Mode)` 명령어를 사용하여 **_파일 권한을 변경_** 할 수 있다. 예를 들어 `chmod 755 app.jar`는 소유자에게 읽기, 쓰기, 실행 권한을 부여하고, 그룹과 기타 사용자에게 읽기와 실행 권한을 부여한다. <br/>

#### 사용자와 그룹의 차이
사용자는 시스템에 로그인하여 명령어를 실행하는 개별 계정이다. 그룹은 여러 사용자를 묶어서 관리하는 단위다. <br/>
각 사용자는 자신의 고유 번호인 UID(User ID)와 그룹 번호인 GID(Group ID)를 가지고 있다. 또한 하나의 사용자는 여러 그룹에 속할 수 있다. <br/>

> 참고로 UID 0은 `root` 계정이며, 1부터 999까지는 시스템 계정, 1000부터는 일반 사용자 계정으로 할당되는 경우가 많다.

```bash
$ ls -al /opt/myapp/
drwxr-xr-x 3 root    root     4096  Mar 8 12:00 .
-rwxr-xr-- 1 appuser appgroup 12345 Mar 8 12:01 app.jar
```

`app.jar` 파일은 해당 파일의 소유자인 `appuser`에게 읽기, 쓰기, 실행 권한이 있고, 그룹인 `appgroup`과 기타 사용자에게는 읽기 권한과 실행 권한이 있다. <br/>
때문에 `appgroup`에 속한 다른 사용자도 `app.jar` 파일을 읽거나 실행할 수 있지만, 쓰기는 할 수 없다. <br/>

```bash
# app.jar 파일의 소유자와 그룹을 변경하기
$ sudo chown appuser:appgroup app.jar

# appgroup에 다른 사용자 추가하기
$ sudo usermod -aG appgroup otheruser

# appgroup에서 다른 사용자 제거하기
$ sudo deluser otheruser appgroup

# otheruser가 속한 그룹 확인하기
$ groups otheruser

# 현재 사용자의 UID와 GID 확인하기
$ id

# 시스템에 존재하는 모든 그룹 확인하기
$ sudo cat /etc/group
```


### 다른 계정으로 명령어 실행하기
현재 내가 가진 계정의 권한으로 명령어 수행이나 파일을 읽거나 쓰는 것이 안 될 때, `sudo` 명령어를 사용하여 `root` 또는 다른 계정으로 명령어를 실행할 수 있다. <br/>
`sudo` 명령어를 사용하지 않고 `root` 계정으로 로그인하여 명령어를 실행하게 될 경우 누가 어떤 명령어를 실행했는지 로그에 남지 않게 되기 때문에 누가 어떤 명령어를 실행했는지 추적하기 위해서는 발급 받은 계정으로 `sudo` 명령어를 사용하여 명령어를 실행하는 것이 좋다. <br/>

```bash
# appuser 계정으로 명령어 실행
$ sudo -u appuser cat /opt/myapp/application.yml

# root 계정으로 명령어 실행 (계정 지정 없이 sudo만 사용)
$ sudo cat /opt/myapp/application.yml

# 명령어 수행 로그 확인
$ sudo cat /var/log/auth.log
```

아래와 같이 `/etc/sudoers` 파일을 편집하여 특정 계정이 특정 명령어를 `sudo`로 실행할 수 있도록 권한을 부여할 수 있다. <br/>

```bash

# sudo 권한이 있는 계정과 명령어 확인
$ sudo cat /etc/sudoers
```

```bash
# user1 계정이 모든 명령어를 비밀번호 없이 실행할 수 있도록 설정
user1 ALL=(ALL) NOPASSWD: ALL

# user2 계정이 docker 서비스를 시작하고 중지하는 명령어를 비밀번호 없이 실행할 수 있도록 설정
user2 ALL=(ALL) NOPASSWD: /usr/bin/systemctl start docker
user2 ALL=(ALL) NOPASSWD: /usr/bin/systemctl stop docker
```

`sudo` 를 통해서 `root` 권한으로 주로 명령어를 수행하는 경우는 아래와 같다.
- 서비스 관리 (systemctl)
- 시스템 파일 편집(vi)
- 파일 권한 변경(chmod, chown)
- 방화벽 설정(ufw)
- 시스템 로그 확인(journalctl)
- 프로세스 관리(kill, lsof)
- 등등...


### 왜 root로 애플리케이션을 실행하면 안 되는가

서비스 운영 시 root 계정으로 애플리케이션을 하게될 경우, **root로 실행된 앱이 해킹되면 서버 전체가 위험** 해진다. 해커가 애플리케이션의 취약점을 파악하여 코드를 심게될경우 root 계정으로 모든 파일과 프로세스에 접근할 수 있기 때문이다. <br/>
때문에 별도의 서비스 계정을 만들어서 애플리케이션을 실행하는 것이 좋다.

### 패스워드 말고 SSH로 접속하기
SSH(Secure Shell)는 원격 서버에 안전하게 접속하기 위한 프로토콜이다. 패스워드 대신 SSH 키를 사용하여 인증하는 것이 보안적으로 더 안전하다. <br/>
왜냐하면 패스워드를 통한 접속 방식은 무차별 대입 공격에 취약할 수 있지만, SSH 키는 공개키와 개인키로 구성되어 있어, 개인키가 유출되지 않는 한 안전하게 접속할 수 있기 때문이다. <br/>

```bash
# SSH 키 생성하기
$ ssh-keygen -t rsa -b 4096 -C "도메인 또는 이메일 주소"

# 생성된 공개키를 서버의 authorized_keys에 추가하기
$ ssh-copy-id -i [공개키] user@server_ip

# authorized_keys 파일 확인하기
$ ls ~/.ssh/authorized_keys

# SSH 키로 서버에 접속하기
$ ssh -i [개인키] user@server_ip
```

### 장애 시나리오

배포 후 앱 실행은 됐는데 파일을 읽거나 쓰지 못하는 경우가 있다. 이때 로그에 아래와 같은 에러가 찍힌다면 OS 계정과 권한 문제일 가능성이 높다.

```
java.io.FileNotFoundException: /opt/myapp/logs/app.log (Permission denied)
```


**1. 앱이 어떤 계정으로 실행 중인지 확인**

```bash
$ ps aux | grep java
appuser   1234  2.5 15.2 4096000 512000 ?  Sl   12:00   5:23 java -jar app.jar
# 첫 번째 컬럼이 실행 계정 → appuser
```

**2. 디렉토리 소유자 확인**

```bash
$ ls -la /opt/myapp/
drwxr-xr-x 2 root root 4096 Mar 8 12:01 logs   ← 소유자가 root!
# appuser는 이 디렉토리에 쓰기 권한 없음
```

**3. 디렉토리 소유자 확인**

```bash
chown -R appuser:appuser /opt/myapp/logs/
```

## 2. 프로세스

애플리케이션은 서버에서 하나의 **프로세스**로 동작한다. OS는 각 프로세스에 **_고유한 PID(Process ID)를 부여하고 CPU와 메모리를 할당해 관리_** 한다. <br/>
서버를 운영하다 보면 애플리케이션이 응답하지 않는 상황이 발생할 수 있다. 이때 프로세스 상태를 모르는 상황에서 단순히 애플리케이션만 재시작하는 것은 문제의 원인을 파악하지 못한 채 조치를 하는 것이다. <br/>
또한 프로세스를 종료할때 `kill -9` 명령어로 강제 종료하는 것은 프로세스가 정상적으로 종료할 기회를 주지 않기 때문에, 처리 중이던 요청이 중간에 잘리고, 파일이 손상될 수 있다. <br/>


### 프로세스 상태

| 상태 | 기호 | 설명 | 정상 여부 |
|---|---|---|---|
| Running | R | CPU를 점유하며 실행 중 | 정상 |
| Sleeping | S | 인터럽트 가능한 대기 (I/O, 이벤트 대기) | 정상 |
| Uninterruptible Sleep | D | 인터럽트 불가 상태, 중지 시그널을 받지 못함, 주로 하드웨어 자원(디스크, 네트워크 I/O)에 접근한 상태 | 지속될 경우 의심해야함 |
| Zombie | Z | 종료됐지만 부모 프로세스가 정리하지 않은 상태 | 쌓이면 PID 고갈 |
| Stopped | T | 중지 시그널을 받아 실행이 중단된 상태 | 의도적이면 정상 |


- Running(R), Sleeping(S), Stopped(T) 상태는 정상적인 상태이다. 
- Uninterruptible Sleep(D) 는 디스크 I/O나 네트워크 I/O를 기다리는 상태로, OS가 시그널을 처리하지 못하기 때문에 **_지속된다면 디스크 장애나 네트워크 문제를 의심_** 해야 한다. 또한 프로세스 자체를 강제종료 할 수 없기 때문에, 서버자체를 재시작해야 할 수도 있다.
- Zombie(Z)는 부모 프로세스가 자식 프로세스의 종료 신호를 처리하지 않아 PID를 계속 점유하고 있는 상태로, **_좀비 프로세스가 쌓이면 PID 고갈로 새 프로세스를 만들 수 없게 된다._** 따라서 좀비 프로세스가 발견되면 원인이 되는 부모 프로세스를 확인하거나 서비스를 재시작해야 한다.


### 프로세스 상태 확인











`ps aux`의 STAT 컬럼에는 상태 뒤에 추가 플래그가 붙기도 한다. Java 앱을 운영하다 보면 자주 보이는 패턴이다.

```bash
$ ps aux | grep java
appuser  1234  2.5 15.2 ...  Sl  12:00  java -jar app.jar
#                             ↑↑
#                             S = Sleeping (정상)
#                             l = 멀티스레드 (Java, Node 앱에서 흔히 보임)
```

| 추가 플래그 | 의미 |
|---|---|
| `l` | 멀티스레드 프로세스 |
| `s` | 세션 리더 (다른 프로세스를 자식으로 둠) |
| `+` | 포그라운드에서 실행 중 |

### 주요 명령어

| 명령어 | 설명 |
|---|---|
| `ps aux` | 현재 실행 중인 모든 프로세스 목록 |
| `ps aux \| grep java` | java 프로세스만 필터링 |
| `top` / `htop` | 실시간 프로세스 모니터링 |
| `kill <PID>` | SIGTERM 전송 (정상 종료 요청) |
| `kill -9 <PID>` | SIGKILL 전송 (강제 종료) |
| `systemctl status <서비스>` | systemd 서비스 상태 확인 |
| `systemctl restart <서비스>` | systemd 서비스 재시작 |

### SIGTERM vs SIGKILL — 왜 kill -9를 바로 쓰면 안 되는가

`kill <PID>`는 기본적으로 SIGTERM을 보낸다. 이것은 프로세스에게 "이제 종료해도 돼"라고 요청하는 것이다. 프로세스는 이 신호를 받으면 처리 중인 요청을 마무리하고, DB 커넥션을 닫고, 임시 파일을 정리한 뒤 종료한다.

`kill -9`는 SIGKILL이다. OS가 즉시 프로세스를 강제 종료한다. 프로세스는 정리 작업을 전혀 하지 못한다. 처리 중이던 요청은 중간에 잘리고, 파일이 손상될 수 있다.

**순서**: 항상 SIGTERM 먼저, 10~30초 기다린 후 응답 없으면 SIGKILL.

### 장애 시나리오: 애플리케이션이 갑자기 응답하지 않는다

모니터링에서 앱이 응답하지 않는다는 알람이 왔다.

**Step 1: 프로세스가 살아있는지 확인**

```bash
$ ps aux | grep java | grep -v grep
# 출력이 없으면 프로세스가 죽은 것 → OOM 또는 다른 원인
# 출력이 있으면 살아있지만 응답 못 하는 것 → 상태 확인 필요
```

**Step 2: 살아있다면 — 좀비 상태 확인**

```bash
$ ps aux | grep java
appuser  1234  0.0  0.0      0     0 ?        Z    12:00   0:00 [java] <defunct>
#                                              ↑ Z = Zombie
```

좀비 상태라면 부모 프로세스(배포 스크립트, supervisord 등)를 확인하거나 서비스를 재시작해야 한다.

**Step 3: 정상 상태인데 응답이 없다면 — 스레드 덤프 확인**

프로세스는 살아있지만 응답이 없는 경우 데드락이나 스레드 고갈을 의심한다.

```bash
# Java 앱 스레드 덤프 (SIGQUIT)
kill -3 <PID>
# 표준 출력 또는 /var/log/ 에 스레드 상태 출력됨
```

**Step 4: 재시작**

```bash
systemctl restart myapp
# 또는 직접 실행 중인 경우
kill -15 <PID>   # SIGTERM
# 10초 후에도 안 죽으면
kill -9 <PID>    # SIGKILL
```

---

## 3. 메모리 — 이유 없이 앱이 죽는다면 OOM Killer를 의심하라

> 가장 무서운 장애 유형이다. 앱 로그에 아무것도 남지 않고 그냥 죽는다.

서버 메모리가 부족해지면 OS는 스스로를 보호하기 위해 **OOM(Out Of Memory) Killer**를 작동시킨다. OOM Killer는 가장 많은 메모리를 쓰는 프로세스를 선택해 강제 종료한다. 대상은 보통 Java 애플리케이션이다.

OOM으로 죽은 프로세스는 앱 로그에 아무것도 남기지 않는다. 앱이 정리할 기회 자체가 없기 때문이다. "이유 없이 주기적으로 앱이 죽는다"는 증상이 있다면 OOM을 가장 먼저 의심해야 한다.

### 메모리 구조

| 영역 | 설명 |
|---|---|
| 물리 메모리 (RAM) | 실제 하드웨어 메모리. 빠르다. |
| 스왑 (Swap) | 물리 메모리 부족 시 디스크 일부를 메모리처럼 사용. 매우 느리다. |
| JVM Heap | Java 앱이 사용하는 메모리. `-Xmx`로 최대 크기 설정. |

스왑 사용량이 늘어난다는 것은 물리 메모리가 부족하다는 신호다. 스왑을 쓰면 성능이 급격히 저하된다. DB 쿼리 타임아웃, HTTP 응답 지연이 함께 발생하는 경우 스왑을 확인해볼 것.

### 주요 명령어

| 명령어 | 설명 |
|---|---|
| `free -h` | 물리 메모리와 스왑 사용량 확인 |
| `vmstat 1 5` | 1초 간격으로 5번 메모리/CPU/스왑 상태 출력 |
| `dmesg \| grep -i killed` | OOM Killer 작동 이력 확인 |
| `cat /proc/<PID>/status \| grep VmRSS` | 특정 프로세스의 실제 메모리 사용량 |

### 장애 시나리오: 앱이 주기적으로 죽는다

배포 후 잘 돌아가던 앱이 매일 새벽 2~3시 사이에 죽는다. 앱 로그에는 아무것도 없다.

**Step 1: OOM Killer가 죽였는지 확인**

```bash
$ dmesg | grep -i "killed process"
[1234567.890] Out of memory: Killed process 5678 (java) total-vm:4096000kB, anon-rss:3800000kB, file-rss:0kB, shmem-rss:0kB
```

이 메시지가 있으면 OOM이 원인이다. `total-vm`은 가상 메모리, `anon-rss`가 실제 사용한 물리 메모리다.

**Step 2: 현재 메모리 상태 확인**

```bash
$ free -h
              total        used        free      shared  buff/cache   available
Mem:           7.7G        6.9G        200M        50M        600M        500M
Swap:          2.0G        1.8G        200M
```

스왑도 거의 다 찼다면 심각한 메모리 부족 상태다.

**Step 3: 대응**

- **단기**: 메모리를 많이 쓰는 불필요한 프로세스 종료, 서버 재시작
- **중기**: JVM Heap 크기 재조정 (`-Xmx`), 메모리 누수 확인
  ```bash
  # Heap 크기 확인 (현재 설정)
  ps aux | grep java | grep -o '\-Xmx[^ ]*'
  ```
- **장기**: 서버 메모리 증설 또는 수평 확장

> 새벽에 주기적으로 죽는 패턴은 새벽 배치 작업 실행 시 메모리 사용이 급증하는 경우가 많다. 배치 실행 시간과 OOM 발생 시간을 비교해보자.

---

## 4. 디스크 — 로그 파일이 서버를 죽인다

> 디스크 풀은 예고 없이 온다. 그리고 모든 것이 멈춘다.

디스크가 100%가 되면 로그 파일 쓰기가 실패하고, DB 데이터 저장이 실패하고, 앱이 임시 파일을 만들지 못해 다운된다. 한 번 가득 차면 연쇄 장애가 발생한다.

디스크 문제는 두 종류다:

1. **용량 고갈**: 가장 흔하다. 로그 파일이 쌓이는 경우가 대부분.
2. **inode 고갈**: 용량은 남아있어도 파일 개수가 한도를 초과한 경우. 증상은 동일하게 "파일 생성 불가".

### 주요 명령어

| 명령어 | 설명 |
|---|---|
| `df -h` | 파일시스템별 디스크 사용량 확인 |
| `df -i` | inode 사용량 확인 |
| `du -sh /var/log/*` | 특정 디렉토리 내 항목별 용량 확인 |
| `du -sh /* 2>/dev/null \| sort -rh \| head -10` | 용량 많이 쓰는 디렉토리 상위 10개 |
| `lsblk` | 블록 디바이스 목록과 마운트 포인트 확인 |

### 장애 시나리오: 디스크가 가득 찼다

앱 배포 후 갑자기 로그가 안 남기 시작한다. 파일 저장도 실패한다.

**Step 1: 어느 파티션이 꽉 찼는지 확인**

```bash
$ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/xvda1       20G   20G     0 100% /        ← 100%!
tmpfs           3.9G     0  3.9G   0% /dev/shm
```

**Step 2: 어떤 디렉토리가 공간을 차지하는지 추적**

```bash
$ du -sh /* 2>/dev/null | sort -rh | head -10
15G	/var
2.1G	/opt
1.2G	/usr
...

$ du -sh /var/* 2>/dev/null | sort -rh | head -5
14G	/var/log
500M	/var/lib
...
```

로그가 주범인 경우가 대부분이다.

**Step 3: 로그 파일 정리**

```bash
# 30일 이상 된 로그 파일 삭제
find /var/log/app -name "*.log.*" -mtime +30 -delete

# 큰 로그 파일 압축 (삭제 전에 확인 필요)
gzip /var/log/app/app.log.2026-02-01
```

**Step 4: inode 고갈 확인 (용량은 남아있는데 파일 생성 불가인 경우)**

```bash
$ df -i
Filesystem      Inodes  IUsed  IFree IUse% Mounted on
/dev/xvda1     1310720 1310720     0  100% /    ← inode 100%!

# 파일이 많은 디렉토리 찾기
find / -xdev -printf '%h\n' 2>/dev/null | sort | uniq -c | sort -rn | head -10
```

> **근본 해결**: 로그 로테이션(logrotate)을 설정해 로그 파일이 일정 크기나 기간 이상 쌓이지 않도록 한다. 문제가 생긴 뒤 정리하는 것보다 미리 막는 게 훨씬 낫다.

---

## 5. 파일 디스크립터 — Too many open files의 정체

> 파일을 연 기억이 없는데 "파일이 너무 많다"는 에러가 난다?

리눅스에서는 파일뿐만 아니라 **소켓, 파이프, 디바이스** 등 모든 I/O 자원을 파일 디스크립터(fd)로 관리한다. 백엔드 앱은 생각보다 많은 fd를 사용한다:

- DB 커넥션 풀의 각 커넥션 → 소켓 fd
- HTTP 클라이언트의 각 연결 → 소켓 fd
- 열려있는 로그 파일 → 파일 fd
- Kafka, Redis 등 외부 시스템 연결 → 소켓 fd

프로세스당 열 수 있는 fd 수에 OS 한도가 있다. 기본값은 보통 1024로 매우 낮다. 이 한도를 초과하면 새 파일을 열거나 새 연결을 맺을 수 없어 `Too many open files` 에러가 발생한다.

### 주요 명령어

| 명령어 | 설명 |
|---|---|
| `ulimit -n` | 현재 셸의 fd 한도 확인 |
| `lsof -p <PID>` | 특정 프로세스가 열고 있는 fd 목록 |
| `lsof -p <PID> \| wc -l` | 특정 프로세스의 fd 총 개수 |
| `cat /proc/<PID>/limits` | 프로세스의 각종 리소스 한도 확인 |
| `cat /proc/sys/fs/file-max` | 시스템 전체 fd 한도 |

### 장애 시나리오: Too many open files 에러

트래픽이 늘면서 앱 로그에 이런 에러가 쌓이기 시작한다:

```
java.net.SocketException: Too many open files
java.io.IOException: Too many open files
```

**Step 1: 현재 fd 사용량 확인**

```bash
# java 프로세스 PID 확인
$ PID=$(ps aux | grep java | grep -v grep | awk '{print $2}')

# 현재 열린 fd 수
$ lsof -p $PID | wc -l
1031

# fd 한도 확인
$ cat /proc/$PID/limits | grep "open files"
Max open files            1024                 1024                 files
# 한도(1024)에 거의 다 찼다
```

**Step 2: 어떤 fd가 많이 열려있는지 확인**

```bash
$ lsof -p $PID | awk '{print $5}' | sort | uniq -c | sort -rn
    800 IPv4          ← 소켓이 800개!
    150 REG           ← 파일 150개
     50 FIFO
```

소켓이 비정상적으로 많다면 커넥션이 닫히지 않고 쌓이고 있는 것이다.

**Step 3: fd 한도 영구 적용**

```bash
# /etc/security/limits.conf에 추가
*    soft    nofile    65536
*    hard    nofile    65536

# systemd 서비스라면 서비스 파일에도 추가
[Service]
LimitNOFILE=65536
```

> **경고**: fd 한도를 높이는 것은 임시 방편이다. 근본 원인은 커넥션 누수(커넥션을 열고 닫지 않는 코드)나 커넥션 풀 설정 오류인 경우가 많다. 한도만 높이면 더 늦게 터질 뿐이다. `lsof -p <PID> | grep CLOSE_WAIT` 로 닫히지 않는 소켓이 있는지 확인하자.

---

## 6. 네트워크 — "연결이 안 돼요"는 수십 가지 원인이 있다

> "외부 API 연결이 안 된다"는 말 뒤에 DNS 문제일 수도, 포트 문제일 수도, 방화벽 문제일 수도 있다.

백엔드 서버는 항상 네트워크를 통해 세상과 연결된다. 클라이언트의 요청을 받고, DB에 연결하고, 외부 API를 호출한다. 이 연결 중 어느 하나라도 막히면 서비스가 정상 동작하지 않는다.

그런데 "연결이 안 된다"는 원인이 다양하다. 코드의 URL이 틀렸을 수도 있고, DNS가 안 될 수도 있고, 해당 포트로의 연결이 방화벽에 막혔을 수도 있다. 원인에 따라 해결 방법이 완전히 다르다.

### 소켓 상태 이해

| 상태 | 설명 |
|---|---|
| LISTEN | 포트를 열고 연결 대기 중 |
| ESTABLISHED | 연결된 상태, 데이터 교환 중 |
| TIME_WAIT | 연결 종료 후 약 60초 대기 (정상) |
| CLOSE_WAIT | 상대방은 끊었는데 내 쪽이 아직 안 닫은 상태. 비정상적으로 쌓이면 문제 |

`TIME_WAIT`은 정상이다. 연결 종료 후 혹시 지연된 패킷이 올 경우를 대비해 잠시 대기하는 것이다. 트래픽이 많은 서버에서는 `TIME_WAIT` 수천 개가 정상이다.

`CLOSE_WAIT`이 계속 쌓인다면 코드에서 커넥션을 제대로 닫지 않는 문제다.

### 주요 명령어

| 명령어 | 설명 |
|---|---|
| `ss -tlnp` | 열려있는 TCP 포트와 사용 프로세스 확인 |
| `ss -s` | 소켓 상태별 요약 통계 |
| `ping <호스트>` | 기본 네트워크 연결 확인 |
| `dig <도메인>` | DNS 조회 결과 확인 |
| `nslookup <도메인>` | DNS 조회 (dig 대안) |
| `curl -v https://example.com` | HTTP 요청 상세 과정 확인 |
| `traceroute <호스트>` | 패킷 경로 추적 |

### 장애 시나리오: 외부 API 또는 DB 연결 실패

앱 로그에 `Connection refused` 또는 `Connection timed out` 에러가 찍힌다.

**Step 1: 기본 네트워크 연결 확인**

```bash
$ ping api.external.com
Request timeout for icmp_seq 0
Request timeout for icmp_seq 1
# 연속 timeout → 네트워크 자체가 막혔거나 서버가 다운됨
```

**Step 2: DNS 조회 확인**

```bash
$ dig api.external.com
;; ANSWER SECTION:
api.external.com.   300   IN  A   203.0.113.1    ← DNS 정상
# ANSWER SECTION이 비어있으면 DNS 해석 실패 → /etc/resolv.conf 확인
```

**Step 3: 포트 연결 확인**

```bash
# curl로 포트 연결 테스트 (timeout 5초)
$ curl -v --connect-timeout 5 telnet://api.external.com:443
# 또는
$ curl -v https://api.external.com/health

# DB 포트 확인 (서버 자신의 포트)
$ ss -tlnp | grep 5432
LISTEN  0  128  0.0.0.0:5432  0.0.0.0:*  users:(("postgres",pid=1234))
```

**Step 4: CLOSE_WAIT 누적 확인**

```bash
$ ss -s
Total: 1452
TCP:   2100 (estab 800, closed 1200, orphaned 0, timewait 150)

# CLOSE_WAIT 개수 직접 확인
$ ss -tnp state close-wait | wc -l
350   ← 지속적으로 증가하면 커넥션 누수
```

---

## 7. 로그 — 장애의 흔적을 어디서 찾는가

> 앱 로그만 뒤지다가 원인을 못 찾는 경우, 시스템 로그를 안 봤기 때문이다.

장애가 발생했을 때 가장 먼저 하는 것이 로그 확인이다. 그런데 어떤 로그를 봐야 하는가?

로그는 크게 세 가지다:

| 종류 | 위치 | 어떤 정보 |
|---|---|---|
| 앱 로그 | 앱 설정에 따라 (예: `/var/log/app/`) | 직접 작성한 로그 (요청, 에러, 비즈니스 이벤트) |
| 시스템 로그 | `journalctl` 또는 `/var/log/syslog` | OS, systemd 서비스 시작/종료, 인증 이벤트 |
| 커널 로그 | `dmesg` | OOM Killer, 하드웨어 오류, 커널 이벤트 |

앱이 갑자기 죽었다면 앱 로그에 아무것도 없을 수 있다. 이때는 커널 로그(`dmesg`)를 봐야 OOM 여부를 알 수 있다. 서비스가 재시작됐다면 `journalctl`에서 재시작 원인을 찾을 수 있다.

### 주요 명령어

| 명령어 | 설명 |
|---|---|
| `journalctl -u <서비스> -f` | 특정 서비스 로그 실시간 확인 |
| `journalctl -u <서비스> --since "1 hour ago"` | 최근 1시간 로그 확인 |
| `journalctl -u <서비스> -n 100` | 최근 100줄 확인 |
| `tail -f /var/log/app/app.log` | 파일 로그 실시간 확인 |
| `grep -i "error\|exception" /var/log/app/app.log` | 에러 로그 필터링 |
| `dmesg -T \| tail -50` | 커널 로그 최근 50줄 (타임스탬프 포함) |
| `dmesg -T \| grep -i "error\|killed\|oom"` | 커널 에러 이벤트 확인 |

### 장애 시나리오: 간헐적으로 에러가 발생하는데 원인을 모르겠다

특정 시간대에만 에러가 발생한다. 재현도 안 된다.

**Step 1: 에러 패턴과 발생 시간 파악**

```bash
$ grep -i "error\|exception" /var/log/app/app.log | tail -100
2026-03-08 02:14:33 ERROR [http-nio-8080-exec-5] Connection pool timeout
2026-03-08 02:14:35 ERROR [http-nio-8080-exec-7] Connection pool timeout
# 새벽 2시 14분대에 집중됨
```

**Step 2: 해당 시간대 시스템 로그 확인**

```bash
$ journalctl -u myapp --since "2026-03-08 02:10:00" --until "2026-03-08 02:20:00"
Mar 08 02:13:00 server01 systemd[1]: myapp.service: Main process exited, code=killed, status=9/KILL
Mar 08 02:13:05 server01 systemd[1]: myapp.service: Start request repeated too quickly.
# 앱이 kill 됐다가 재시작됨 → OOM 의심
```

**Step 3: 커널 로그에서 OOM 확인**

```bash
$ dmesg -T | grep -i "killed\|oom" | grep "2026-03-08"
[Mar 8 02:12:58] Out of memory: Killed process 5678 (java)
# OOM이 원인이었다
```

> **팁**: 로그 로테이션(logrotate)은 필수다. 로그 파일이 무한정 커지면 디스크가 가득 차고, 로그 자체가 서비스 장애의 원인이 된다. `/etc/logrotate.d/`에 앱 로그 로테이션 설정을 반드시 추가하자.

---

## 8. 환경 변수 — 환경마다 다른 설정, 안전하게 다루기

> DB 비밀번호를 코드에 하드코딩하면 안 된다는 건 안다. 그럼 어떻게 관리해야 하는가?

개발 환경과 운영 환경은 DB 주소, API 키, 외부 서비스 URL 등이 모두 다르다. 이 값들을 코드에 직접 넣으면 두 가지 문제가 생긴다. 환경마다 다른 코드를 빌드해야 하고, 비밀 값이 git 저장소에 올라간다.

**환경 변수**는 이 문제를 해결한다. 프로세스가 실행될 때 OS로부터 키-값 쌍을 전달받아 사용한다. 코드는 동일하고, 실행 환경에 따라 다른 값이 주입된다.

### 주요 명령어

| 명령어 | 설명 |
|---|---|
| `env` | 현재 프로세스의 모든 환경 변수 출력 |
| `printenv <변수명>` | 특정 환경 변수 값 출력 |
| `export KEY=value` | 환경 변수 설정 (현재 셸 + 하위 프로세스에 전달) |
| `echo $KEY` | 환경 변수 값 확인 |
| `unset KEY` | 환경 변수 제거 |

### systemd 서비스에서 환경 변수 주입

systemd로 관리되는 서비스는 서비스 파일에서 환경 변수를 설정하거나, 별도 파일로 분리해서 주입한다.

```ini
# /etc/systemd/system/myapp.service
[Service]
User=appuser
ExecStart=/usr/bin/java -jar /opt/myapp/app.jar

# 직접 설정
Environment="SPRING_PROFILES_ACTIVE=prod"
Environment="SERVER_PORT=8080"

# 파일로 분리 (비밀 값은 별도 파일로)
EnvironmentFile=/opt/myapp/.env
```

```bash
# /opt/myapp/.env
DB_URL=jdbc:mysql://prod-db.internal:3306/mydb
DB_PASSWORD=s3cr3t_password
API_KEY=my-secret-api-key
```

```bash
# 서비스 파일 수정 후 반드시 reload
systemctl daemon-reload
systemctl restart myapp

# 환경 변수가 제대로 주입됐는지 확인
systemctl show myapp --property=Environment
```

### 실무에서 자주 놓치는 주의사항

**1. `.env` 파일은 절대 git에 커밋하지 않는다**

```bash
# .gitignore에 반드시 추가
.env
*.env
.env.prod
```

**2. `env` 명령어 출력을 조심한다**

```bash
$ env
DB_PASSWORD=s3cr3t_password  ← 비밀번호가 그대로 노출!
API_KEY=my-secret-api-key
```

디버깅 중 `env` 출력이 로그에 남거나 화면에 보이면 비밀 값이 노출된다. 특히 CI/CD 파이프라인 로그를 주의한다.

**3. 환경 변수는 자식 프로세스에 상속된다**

```bash
export DB_PASSWORD=s3cr3t
# 이 셸에서 실행하는 모든 명령어가 DB_PASSWORD를 볼 수 있다
# 사용 후 제거
unset DB_PASSWORD
```

---

## 마치며 — 장애가 났을 때 무엇부터 확인할까

서버 지식은 평소에 쓸 일이 거의 없다. 장애가 났을 때, 갑자기 필요하다. 그때 이 체크리스트를 순서대로 따라가면 대부분의 원인을 좁힐 수 있다.

### 장애 초기 대응 체크리스트

```bash
# 1. 프로세스가 살아있나?
ps aux | grep java

# 2. OOM으로 죽었나? (앱 로그에 아무것도 없을 때)
dmesg | grep -i "killed process"

# 3. 디스크가 꽉 찼나?
df -h

# 4. 메모리가 부족한가?
free -h

# 5. 네트워크 연결이 되나?
ping db.internal
ss -tlnp | grep 8080

# 6. 앱 로그와 시스템 로그에 뭐가 있나?
tail -100 /var/log/app/app.log | grep -i error
journalctl -u myapp --since "30 minutes ago"
```

이 여섯 가지를 순서대로 확인하면 "코드 문제인가, 서버 문제인가"를 빠르게 구분할 수 있다.

### 핵심 정리

| 증상 | 먼저 확인할 것 |
|---|---|
| 앱이 갑자기 죽었고 로그가 없다 | `dmesg \| grep killed` (OOM 의심) |
| 로컬에서는 됐는데 서버에서 안 된다 | `ls -la`, `id` (권한 문제 의심) |
| 앱은 떠있는데 응답이 없다 | `ps aux` (Zombie), 스레드 덤프 |
| 로그 쓰기 실패, 파일 저장 실패 | `df -h`, `df -i` (디스크/inode 고갈) |
| Too many open files | `lsof -p <PID> \| wc -l` (fd 한도) |
| 외부 API/DB 연결 실패 | `ping`, `dig`, `ss -tlnp` (네트워크) |
| 간헐적 에러, 원인 불명 | `journalctl`, `dmesg -T` (로그 추적) |

서버 지식은 코드 실력을 대체하지 않는다. 하지만 코드가 아무리 좋아도 서버를 모르면 장애 앞에서 속수무책이 된다. 이 글에서 다룬 내용이 실제 장애 현장에서 도움이 되길 바란다.
