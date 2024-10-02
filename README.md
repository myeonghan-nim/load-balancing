# load balancing

## 로드 밸런싱

서버에 들어오는 네트워크 트래픽을 여러 서버에 분산시키는 것을 의미하며 **로드 밸런서**는 이를 수행하는 장치 혹은 소프트웨어를 의미.
로드 밸런싱을 통해 트래픽 과부하로 인한 서버 다운을 방지하고 서버 자원을 효율적으로 사용.
로드 밸런싱은 수평적 확장과 밀접한 관련이 있으며 크게 두 가지 방식이 존재

1. L4 로드 밸런싱: 전송 계층(Transport Layer, TCP/UDP)에서 로드 밸런싱
2. L7 로드 밸런싱: 애플리케이션 계층(Application Layer, HTTP)에서 로드 밸런싱

### 로드 밸런싱 알고리즘

1. Round Robin: 서버들에 차례대로 요청을 분산
2. Least Connections: 현재 연결 수가 가장 적은 서버에 요청을 분산
3. IP Hash: 클라이언트의 IP 주소를 해싱하여 특정 서버에 연결
4. Weighted Round Robin: 서버의 성능에 따라 가중치를 부여하여 트래픽 분산

## TIPs

### nginx

1. nginx를 특정 위치에 존재하는 `nginx.conf` 파일을 사용해 실행하려면 다음과 같은 명령어 사용

```bash
$ sudo nginx -c /your/custom/path/nginx.conf

# 종료
$ sudo nginx -s stop
```

### AWS의 ELB

0.  전제 조건
    - AWS에 두 대의 EC2 인스턴스를 실행 중
    - 각 EC2 인스턴스에는 nginx로 실행 중인 웹 서버가 존재
      - `/var/www/html/index.html`에 `"This is Server ${server_name}`이라고 기록된 상태
    - 각 EC2 인스턴스는 다음과 같이 설정된 상태
      - Security Group의 Inbound Rules에 HTTP 트래픽(0.0.0.0/0) 허용
1.  ELB 생성
    1.  EC2 서비스의 좌측 메뉴에서 Load Balancers를 선택하고 Create Load Balancer 선택
    2.  HTTP/HTTPS 기반 트래픽 처리를 위해 Application Load Balancer 선택
        1. Name 입력
        2. Scheme에 Internet-facing 선택
        3. IP Address Type에 IPv4 선택
    3.  네트워크 설정
        1. EC2 인스턴스와 동일한 VPC 선택
        2. Availability Zones에서 EC2 인스턴스가 있는 모든 영역을 선택
        3. Security Groups에서 기존 보안 그룹을 선택하거나 새로 생성하여 HTTP(80번 포트) 접근을 허용
    4.  리스너 설정
        1. 기본적으로 HTTP(80번 포트) 리스너가 설정되어 있으나 다른 포트에서 수신을 원하면 추가 리스너를 설정
    5.  대상 그룹 생성
        1. Name에 이름 입력
        2. Target Type을 Instance로 선택
        3. Protocol을 HTTP로, Port를 80으로 선택
        4. VPC를 EC2 인스턴스와 동일한 VPC로 설정
        5. Health Check 설정
           1. Protocol을 HTTP로, Path를 / (기본 경로)로 설정
           2. Health check가 실패할 경우 대상을 자동으로 제외하도록 설정
    6.  대상 인스턴스 등록
        1. Register targets에서 두 개의 EC2 인스턴스를 선택
        2. Add to registered 클릭하여 대상 그룹에 EC2 인스턴스 등록
        3. 설정 완료 후 Create 버튼 클릭
2.  로드 밸런서 확인
    1. ELB 상태 확인: 약간 시간이 걸리지만 상태가 활성화인지 확인
    2. ELB DNS 확인: 활성화된 후 Load Balancer 목록에서 해당 ALB의 DNS 이름을 확인
3.  테스트: 브라우저에서 ALB의 DNS 이름을 통해 접속
