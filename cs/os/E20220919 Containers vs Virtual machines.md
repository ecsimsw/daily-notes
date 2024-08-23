## Containers vs virtual machines

### 기본 개념

Containers and virtual machines are very similar resource virtualization technologies. 
Virtualization is the process in which a system singular resource like RAM, CPU, Disk, or Networking can be ‘virtualized’ and represented as multiple resources.    

The key differentiator between containers and virtual machines is that <b> virtual machines virtualize an entire machine down to the hardware layers </b> and <b> containers only virtualize software layers above the operating system level. </b>

<br>
<br>

![SWTM-2060_Diagram_Containers_VirtualMachines_v03](https://user-images.githubusercontent.com/46060746/191042848-d2560cb6-efb5-4c66-b901-5c2977e7e1e7.png)

<br>

### 컨테이너는 커널을 공유하여 더 가볍게, 빠르게.

Virtual machine은 Host OS 위에 하이퍼바이저를 띄우고 그 위에 VM 마다 GUEST_OS를 올려 환경을 구성하는 반면, Container(도커를 예시)는 Host OS 위에 하이퍼바이저 없이 도커 엔진이 돌고, 그 위에 애플리케이션이 띄워진다. 즉 물리적으로 격리한 VM와 달리, 컨테이너는 논리적으로 격리하여 환경을 만드는 개념이다. 컨테이너는 커널을 공유하고, 그렇기에 더 빠르고 가볍다.

<br>

### Host OS, 각 Guest OS 간 덜 의존적인 VM

반대로 Host OS와 만들고자 하는 환경의 의존은 VM이 훨씬 낮다. 커널을 공유하는 컨테이너의 경우에는 Host OS에 해당하는 컨테이너 환경을 구성해야 하는 반면, Host OS 위에 각 환경마다 Guest OS를 설치하여 커널이 분리되는 VM의 경우 Host OS와 무관하게 환경을 구성할 수 있는 것이다. 그저 Guest OS를 원하는 상황으로 만들어주면 된다. 같은 이유로 보안의 경우에도 커널을 공유하는 컨테이너가 더 취약하다. 
