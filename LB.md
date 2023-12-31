## Target Group에서 draining을 사용하는 이유
**Target Group의 draining time는 ELB에서 인스턴스 또는 컨테이너를 Target Group에서 제거하기 전에 대기시키는 시간을 나타낸다.**
### **Draining time을 사용하는 주요 이유:**
**```기존 연결의 완료:```** Draining time 동안에는 새로운 요청이 타겟(예: EC2 인스턴스)으로 라우팅되지 않지만, 기존에 시작된 연결은 계속 처리될 수 있습니다. 이로 인해 서비스 중단이나 중요한 트랜잭션의 중단을 방지할 수 있습니다.

**```부드러운 확장 및 축소:```** Auto Scaling 이벤트나 배포 중에도 사용자에게 영향을 주지 않는 부드러운 확장 및 축소를 지원합니다.

**```유연한 유지 보수:```** 인스턴스의 유지 보수나 업데이트를 수행할 때 서비스 중단 없이 타겟을 안전하게 등록 해제하고 다시 등록할 수 있습니다.

## ALB를 거친 request는 EC2에서 확인 시 클라이언트 IP가 ALB의 주소로 확인 되지만, NLB를 거친 request는 실제 request를 쏜 클라이언트의 IP로 확인된다. 이러한 현상이 나타나는 이유
**ALB와 NLB는 각각 응용 계층 및 전송 계층에서 작동하는 AWS의 두 가지 주요 로드 밸런서 유형이다. 그들의 통신 방식의 차이로 인해 요청의 출처 IP가 다르게 나타나는 특징이 있다.**

### **```ALB:```**

**작동 계층:** ALB는 응용 계층에서 작동한다.
**통신 방식:** ALB가 클라이언트와 EC2 인스턴스 사이에서 프록시 역할을 한다. 클라이언트의 요청을 받아들이고 새로운 연결을 사용하여 해당 요청을 EC2 인스턴스로 전달한다.
**IP 정보:** ALB가 연결을 초기화하기 때문에, EC2 인스턴스는 요청의 출처 IP로 ALB의 IP 주소를 보게 된다.

### **```NLB:```**

**작동 계층:** NLB는 전송 계층 에서 작동한다.
**통신 방식:** NLB는 클라이언트의 TCP/UDP 트래픽을 타겟으로 전달하면서 IP 패킷의 내용을 변경하지 않는다.
**IP 정보:** NLB는 연결 및 패킷을 수정하지 않기 때문에, EC2 인스턴스는 요청의 출처 IP로 실제 클라이언트의 IP 주소를 보게 된다.

## ALB의 Stickiness가 어떤 문제 때문에 사용하는지와, 해당 문제를 Stickiness option을 사용하지 않고 다른 기능이나 아키텍쳐적으로 해결할 수 있는 방법

### **```ALB의 Stickness 사용 이유```**
ALB의 Stickiness 기능은 특정 클라이언트의 모든 요청을 동일한 타겟으로 라우팅하는 기능이다. 이 기능이 필요한 주된 이유는 세션 상태 유지이다. 즉, 애플리케이션이 서버측에 상태 정보를 저장할 때, 동일한 사용자의 연속적인 요청을 동일한 서버로 라우팅하여 해당 상태 정보에 계속 접근할 수 있게 하려는 것이다.
### **```tickiness 없이 세션 상태 유지 문제 해결 방법:```**
**서버리스 세션:**
    세션 정보를 서버의 메모리가 아닌 외부 스토리지에 저장한다. 이렇게 하면 모든 서버가 동일한 세션 정보에 접근할 수 있다.

**JWT (JSON Web Tokens):**
    사용자 인증과 관련된 정보를 서버측에서 관리하는 대신 클라이언트 측에 토큰 형태로 전달한다. JWT는 상태를 저장하지 않기 때문에 서버 간의 연속성이 필요하지 않는다.

**Database Centralization:**
    애플리케이션의 상태와 데이터를 중앙 데이터베이스에 저장하여 모든 인스턴스가 동일한 데이터베이스를 조회하게 한다.

 **Stateless Architecture:**
    애플리케이션을 상태가 없는 (stateless) 방식으로 설계한다. 이렇게 하면 각 요청은 독립적으로 처리될 수 있으며, 어떤 서버에서든 해당 요청을 처리할 수 있다.
