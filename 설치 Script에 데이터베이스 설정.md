### 🔑 DB 관리자(Postgres) 비밀번호 설정 정책
* **방식**: 설치 시 `--pwfile` 옵션을 이용한 공통 비밀번호 주입.
* **비밀번호**: `k1plant1234!` (예시)
* **장점**: 
  - 현장 사용자의 비밀번호 분실 리스크 방지.
  - 개발자의 원격 유지보수 용이성 확보.
* **보안**: 설치 완료 직후 임시 비밀번호 파일(`db_pw.txt`)은 NSIS 스크립트에 의해 자동 삭제됨.


```Python
import psycopg2

# IP 대신 서버 컴퓨터 이름을 직접 사용
server_host = "THEPLUS-SERVER" 
try:
    conn = psycopg2.connect(
        host=server_host,
        database="piping_db",
        user="postgres",
        password="your_password",
        port=5432
    )
except Exception as e:
    print(f"서버 접속 실패: {e}")
```
# 🌐 클라이언트-서버 자동 연결 전략

## 1. 호스트네임 연결 (Primary)
- IP 대신 윈도우 컴퓨터 이름(Host Name)을 Connection String의 host 값으로 사용.
- 유동 IP 환경에서도 재설정 없이 연결 유지 가능.

## 2. UDP Beacon 탐색 (Advanced)
- 클라이언트 실행 시 LAN 내 모든 IP에 탐색 패킷 전송.
- 서버 응답 확인 시 해당 IP를 설정 파일에 자동 저장.
- 사용자 개입 없는 'Zero-Config' 구현 가능.

## 3. 예외 처리
- 연결 실패 시 '서버 IP 수동 입력' 창을 띄워 사용자 대응 경로 확보.
- 윈도우 방화벽(5432 포트) 허용 여부를 프로그램 시작 시 자체 체크.

# UDP Beacon 탐색 (Advanced)
# -----------------------------
1. 작동 원리 (매커니즘)
- 서버 (수신 대기): UDP 9999번 포트를 열고 "누구 나 찾는 사람 있나?" 하고 계속 듣고 있습니다.
- 클라이언트 (브로드캐스트): "the PLUS Piping 서버 어디 있나요?"라는 특정 신호(Beacon)를 네트워크 전체(255.255.255.255)에 뿌립니다.
- 서버 (응답): 신호를 받으면 "내가 서버다! 내 IP는 192.168.0.10이다!"라고 클라이언트에게만 답장을 보냅니다.
- 클라이언트 (연결): 받은 IP를 사용하여 PostgreSQL에 접속합니다.

2. 서버 측 코드 (Server Beacon)
서버 역할을 하는 메인 PC에서 프로그램 실행 시 백그라운드 쓰레드로 돌려야 합니다.
```Python
import socket

def start_udp_beacon_server():
    # UDP 소켓 생성
    sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    # 모든 네트워크 인터페이스의 9999 포트에서 대기
    sock.bind(('', 9999))
    
    print("UDP Beacon 서버가 시작되었습니다. 클라이언트를 기다립니다...")
    
    while True:
        # 데이터 수신 (최대 1024 바이트)
        data, addr = sock.recvfrom(1024)
        message = data.decode('utf-8')
        
        if message == "FIND_PLUS_PIPING_SERVER":
            print(f"클라이언트 발견: {addr}")
            # 응답 메시지 전송 (나 여기 있어!)
            response = "I_AM_PLUS_PIPING_SERVER"
            sock.sendto(response.encode('utf-8'), addr)
```
3. 클라이언트 측 코드 (Client Discovery)
```Python
import socket

def find_server_ip():
    sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    sock.setsockopt(socket.SOL_SOCKET, socket.SO_BROADCAST, 1) # 브로드캐스트 권한 부여
    sock.settimeout(3.0) # 3초 동안 응답 없으면 포기
    
    server_ip = None
    message = "FIND_PLUS_PIPING_SERVER"
    
    try:
        # 전체 네트워크(255.255.255.255)에 신호 발송
        sock.sendto(message.encode('utf-8'), ('255.255.255.255', 9999))
        
        # 서버로부터의 응답 대기
        data, addr = sock.recvfrom(1024)
        if data.decode('utf-8') == "I_AM_PLUS_PIPING_SERVER":
            server_ip = addr[0] # 응답을 보낸 기기의 IP 주소 추출
            print(f"서버를 찾았습니다: {server_ip}")
    except socket.timeout:
        print("서버를 찾을 수 없습니다. (타임아웃)")
    finally:
        sock.close()
    
    return server_ip
```
4. 실전 배포 시 주의사항 (방화벽)
UDP 탐색이 안 되는 가장 큰 이유는 윈도우 방화벽입니다. NSIS 설치 스크립트에 PostgreSQL 포트(5432)뿐만 아니라 UDP 9999번 포트도 열어주는 코드를 반드시 추가해야 합니다.
### [NSIS 추가 구문]
```NSIS
; UDP Beacon용 9999 포트 개방
nsExec::Exec 'netsh advfirewall firewall add rule name="thePLUS_Piping_UDP" dir=in action=allow protocol=UDP localport=9999'
```
### 💡 "the PLUS Piping" 최적화 팁
1. 캐싱(Caching): 매번 찾으면 느려질 수 있으니, 한 번 찾은 IP는 settings.db나 ini 파일에 저장해 두세요. 접속에 실패할 때만 다시 UDP 탐색을 돌리는 것이 효율적입니다.
2. 멀티 서버 방지: 혹시 한 사무실에 서버를 두 대 설치하는 실수를 범할 수 있습니다. 이 경우 클라이언트가 리스트를 보여주고 "어느 서버에 접속할까요?"라고 묻는 UI를 추가하면 완벽합니다.
