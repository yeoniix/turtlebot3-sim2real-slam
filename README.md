# 🐢turtlebot3-sim2real-slam
ROS2 기반 TurtleBot3의 시뮬레이션 및 실환경 지도 생성·자율주행 검증 프로젝트

## 1.	시뮬레이션 환경 구축 및 테스트
1)	시뮬레이션 환경 구성 (Gazebo -> ROS2)
Gazebo Harmonic 환경에서 TurtleBot3 Burger 모델을 활용해 자율 주행 실험을 수행하였다. 실제 실험 환경과 유사한 조건을 구현하기 위해 사용자가 직접 설계한 미로 형태의 SDF(Simulation Description Format) 맵을 제작하였으며, Gazebo 내부에 배치하였다. 

2)	센서 데이터 처리 및 브릿지 통신 (Gazebo -> ROS2)
시뮬레이션이 시작되면 TurtleBot3에 장착된 LiDAR센서 플러그인이 주변 환경을 스캔해 레이저 거리 정보를 생성한다. 생성된 데이터는 Gazebo 내부 메시지 형식(gz.msgs.LaserScan)으로 발행되며, ros_gz_bridge의 parameter_bridge가 이를 감지하여 ROS2 표준 메시지 형식(sensor_msgs/msg/LaserScan)으로 변환 후 /scan 토픽으로 전달한다. 
동시에 로봇의 위치와 자세 정보인 /odom 및 좌표계 변환 정보 /tf 역시 Gazebo에서 생성되어 브릿지를 통해 ROS2 네트워크로 전달된다. 이러한 과정은 시뮬레이션 환경에서 생성된 데이터를 ROS2 생태계에서 그대로 활용할 수 있도록 하는 역할을 수행한다. 

3)	로봇제어 흐름(ROS2 -> Gazebo)
사용자는 teleop_keyboard 노드를 이용해 키보드 입력 기반 원격 제어를 수행한다. 사용자의 입력은 ROS2의 /cmd_vel 토픽을 통해 선속도와 각속도 명령으로 발행된다.
이후 ros_gz_bridge는 해당 명령을 Gazebo 메시지 형식(gz.msgs.Twist)로 변환해 시뮬레이터 내부의 TurtleBot3 모터 컨트롤러로 전달한다. 결과적으로 사용자의 제어 명령이 Gazebo 내 로봇의 실제 움직임으로 반영되며 이를 통해 주행 및 센서 데이터 수집이 이루어진다.

4)	SLAM 기반 지도 생성 
ROS2 환경에서는 cartographer_node가 실행되어 실시간 지도 작업을 수행했다. Cartographer는 LiDAR스캔 데이터(/scan), 로봇 위치 정보(/odom), 좌표 변환 정보(/tf)를 통합적으로 활용해 로봇의 위치를 추정하고 주변 환경을 2차원 점유 격자 지도 형태로 생성한다. 초기에는 teleop_keyboard를 이용한 수동 조작 방식으로 지도 생성을 수행했으나 넓은 공간을 탐색하는 과정에서 조작의 어려움과 비효율성이 발생하였다. 이를 개선하기 위해 Naviagation2(Nav2)의 Goal Pose기능을 활용하였다. 사용자는 Rviz2상에서 탐색이 필요한 위치를 목표 지점으로 지정했으며 로봇은 해당 위치까지 자율적으로 이동하며 주변 환경을 스캔하였다. 이러한 방식으로 사용자의 지속적인 조작 없이도 효율적으로 미로 내부를 탐색할 수 있었으며, 결과적으로 보다 빠르고 안정적으로 전체 지도를 완성할 수 있었다.

5)	시각화 및 검증
Rviz2는 /scan, /odom, /tf, /map 등의 주요 토픽을 구독해 센서 데이터와 생성된 지도를 실시간으로 시각화한다. 이를 통해 사용자는 로봇의 현재 위치, LiDAR스캔 결과, 지도 생성 진행 상황을 한 화면에서 확인할 수 있으며 SLAM알고리즘이 정상적으로 동작하는지 검증할 수 있다. 







## 2.	실물 TurtleBot3 연동 및 실환경 검증
1)	네트워크 환경 구축
실제 TurtleBot3 실험을 위해 Wi Fi-Router를 이용해 제어용 PC와 TurtleBot3를 동일 네트워크에 연결하였다. 이후 SSH를 활용해 원격으로 TurtleBot3 SBC(Raspberry Pi)에 접속했으며, ROS2 노드 실행 및 상태 모니터링을 수행하였다. 

2)	실환경 미로 제작 및 주행
시뮬레이션에서 사용한 환경과 유사한 구조의 실제 미로를 3d 프린터기를 이용한 구조물, 스티로폼 보드와 박스를 이용하여 제작하였다. TurtleBot3는 해당 환경을 직접 주행하며 LiDAR 데이터를 수집하였고, ROS2 네트워크를 통해 Cartographer가 실시간 지도 생성을 수행하였다.



4)	Sim – to – Real 검증
시뮬레이션 환경에서 검증한 SLAM 파이프라인을 실제 TurtleBot3에 동일하게 적용해 지도 생성 성능을 확인하였다. 이를 통해 Gazebo 기반 가상환경과 실제 환경 간의 차이를 비교, 분석할 수 있었으며 시뮬레이션에서 설계한 시스템이 실제 로봇에서도 정상적으로 동작함을 확인하였다.


<완성 지도>


<img width="337" height="304" alt="image" src="https://github.com/user-attachments/assets/080b3da7-e790-4b92-afa6-3db4558c9c9a" />

<실물 터틀봇 미로 환경>


<img width="251" height="251" alt="image" src="https://github.com/user-attachments/assets/9c074716-0a20-422b-bb22-5aceefa7c716" />

<완성된 SLAM 지도>


<img width="317" height="276" alt="image" src="https://github.com/user-attachments/assets/f7ce513e-3fb3-4360-971f-26dd879337b7" />


