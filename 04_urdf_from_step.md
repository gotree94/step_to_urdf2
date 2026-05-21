# 방법 4: urdf-from-step (자동 변환)

**최종 결과물**: URDF (ROS 패키지 형태) + STL 메시 파일
**난이도**: 낮음 (설치만 성공하면 가장 간단)
**라이선스 제약**: ❌ 없음 (BSD 3-Clause)

---

## 개요

[ReconCycle/urdf-from-step](https://github.com/ReconCycle/urdf-from-step)은 OpenCASCADE(OCCT) 라이브러리로 STEP 파일을 직접 파싱하여 URDF로 변환하는 ROS 패키지입니다.

**작동 원리**:
1. STEP 파일 로드 → OCCT로 파트 분석
2. 파트 이름에서 **"joint"**, **"link"** 키워드 검색 → URDF 트리 구조 자동 구성
3. 나머지 파트는 메시(STL)로 변환
4. URDF XML + STL 파일 + ROS launch 파일을 포함한 ROS 패키지 생성

**장점**:
- **완전 자동화**: STEP 파일만 준비하면 한 번에 URDF 생성
- 동일한 OCCT 엔진 사용 → FreeCAD와 같은 수준의 형상 정밀도
- ROS 패키지까지 자동 생성 (launch 파일 포함)

**단점**:
- **STEP 파일에 naming convention 강제** (파트명에 "joint"/"link" 키워드 포함 필요)
- **STL만 출력** (DAE 미지원) → 삼각형 메시 품질 한계
- **ROS1 기반** (ROS2 직접 지원 안 함, XACRO 미지원)
- pythonOCC-core 설치 난이도가 높음 → Docker 사용 강력 권장
- 2024년 9월 이후 유지보수 소극적

---

## 1. 설치

### 1.1 사전 요구사항

- ROS (Noetic 권장, 또는 Melodic)
- Docker (권장 설치 경로)
- 또는 pythonOCC-core (난이도 높음)

### 1.2 방법 A: Docker로 설치 (권장)

**1. Docker Desktop 설치**

1. [Docker Desktop for Windows](https://www.docker.com/products/docker-desktop/) 다운로드 및 설치
2. 설치 후 Docker Desktop 실행
3. WSL 2 기반 엔진 사용 옵션 선택 (Windows 10/11)
4. 재부팅 후 Docker 실행 확인: `docker --version`

**2. urdf-from-step 이미지 빌드**

```bash
# 1. GitHub에서 클론
git clone https://github.com/ReconCycle/urdf-from-step.git
cd urdf-from-step

# 2. Docker 이미지 빌드
docker build -t urdf_from_step .
```

빌드 시간: 처음 빌드는 OCCT 및 pythonOCC 컴파일이 포함되어 **30분~1시간** 소요될 수 있습니다.

> **TIP**: 미리 빌드된 이미지가 있는지 확인:
> ```bash
> docker pull reconcycle/urdf-from-step
> ```

### 1.3 방법 B: 네이티브 설치 (고급, Linux 권장)

Windows에서 네이티브 설치는 매우 까다롭습니다. Linux (Ubuntu 20.04) 권장:

```bash
# 1. ROS Noetic 설치
sudo apt install ros-noetic-desktop-full

# 2. pythonOCC-core 설치 (가장 까다로운 단계)
#    Conda 환경 권장
conda create -n occ_env python=3.8
conda activate occ_env
conda install -c conda-forge pythonocc-core

# 3. URDF 변환기 종속성 설치
sudo apt install ros-noetic-urdfdom
pip install numpy

# 4. catkin 워크스페이스 설정
mkdir -p ~/catkin_ws/src
cd ~/catkin_ws/src
git clone https://github.com/ReconCycle/urdf-from-step.git
cd ~/catkin_ws
catkin_make
source devel/setup.bash
```

> **Windows 네이티브 설치는 권장하지 않습니다.** pythonOCC-core의 Windows 지원이 제한적이므로 Docker 사용을 강력히 권장합니다.

---

## 2. STEP 파일 준비 (가장 중요한 단계)

### 2.1 파트 네이밍 규칙

urdf-from-step은 STEP 파일 내 **파트(Part) 이름**과 **어셈블리(Assembly) 이름**에서 키워드를 검색하여 URDF 구조를 자동 판단합니다.

| 키워드 | 의미 | 필수 |
|---|---|---|
| `link` | URDF의 **링크**로 변환됨 | ✅ 최소 1개 이상 |
| `joint` | URDF의 **관절**로 변환됨. joint 이름이 다음 link 이름을 결정 | ✅ 최소 1개 이상 |
| `base_link` | 특별 케이스: URDF의 루트 링크 | 권장 |

### 2.2 STEP 파일 구조 예시

올바른 구조의 STEP 파일 예시 (CAD 설계 트리 기준):

```
robot_arm (Assembly)
├── base_link (Part)              ← "link" 키워드 → URDF link
├── base_to_shoulder_joint (Assembly)  ← "joint" → URDF joint
│   ├── base_link_connection (Part)    ← 연결 정보
│   └── shoulder_link (Part)           ← "link" 키워드 → URDF link
├── shoulder_to_upper_arm_joint (Assembly)
│   ├── shoulder_link_connection (Part)
│   └── upper_arm_link (Part)
├── upper_arm_to_forearm_joint (Assembly)
│   ├── upper_arm_link_connection (Part)
│   └── forearm_link (Part)
└── ... (계속)
```

### 2.3 CAD 소프트웨어별 설정

#### SolidWorks

1. **FeatureManager 트리**에서 파트/어셈블리 이름에 키워드 포함
   - 예: `base_link`, `shoulder_link`, `shoulder_joint`
2. **File → Save As** → STEP AP214 (`*.step`)로 저장
   - AP203도 동작하나 AP214가 색상 정보 보존에 더 좋음
3. STEP 내보내기 옵션에서 **"Split assembly into individual parts"** 체크

#### Fusion 360

1. **Browser 트리**에서 컴포넌트 이름 변경
2. **File → Export → STEP** (.step)
3. 옵션: **"Export entire assembly"**

#### FreeCAD

1. Combo View에서 각 Body/Part 이름에 키워드 포함
2. **File → Export → STEP** (.step)

### 2.4 준비된 STEP 파일 예시 다운로드

공식 저장소에서 예시 STEP 파일 확인:

```bash
# 예시 파일 목록
ls urdf-from-step/test/step_files/
```

직접 테스트용 간단한 예시:
- `urdf-from-step/test/step_files/simple_robot_arm.step`

---

## 3. 실행 (URDF 변환)

### 3.1 Docker로 실행

```bash
# 1. STEP 파일을 위한 입력 디렉토리 생성
mkdir D:/step_input
mkdir D:/urdf_output

# 2. STEP 파일을 입력 디렉토리에 복사
copy robot_arm.step D:/step_input/

# 3. Docker 컨테이너 실행
docker run -it --rm \
  -v D:/step_input:/input_step_files \
  -v D:/urdf_output:/output_urdf \
  urdf_from_step \
  roslaunch urdf_from_step build_urdf_from_step.launch \
  step_file_path:="/input_step_files/robot_arm.step" \
  urdf_package_name:="robot_arm"
```

**매개변수 설명**:
| 매개변수 | 설명 | 예시 |
|---|---|---|
| `step_file_path` | 입력 STEP 파일 경로 (컨테이너 내부 기준) | `/input_step_files/robot_arm.step` |
| `urdf_package_name` | 생성될 ROS 패키지 이름 | `robot_arm` |

### 3.2 네이티브 (Linux) 실행

```bash
# 1. 워크스페이스 소스
source ~/catkin_ws/devel/setup.bash  # 또는 setup.zsh

# 2. 변환 실행
roslaunch urdf_from_step build_urdf_from_step.launch \
  step_file_path:="/home/user/step_files/robot_arm.step" \
  urdf_package_name:="robot_arm"
```

### 3.3 실행 중 출력 예시

```
[INFO] Loading STEP file: /input_step_files/robot_arm.step
[INFO] Analyzing STEP structure...
[INFO] Found 5 parts with 'link' keyword: base_link, shoulder_link, ...
[INFO] Found 4 parts with 'joint' keyword: base_to_shoulder, ...
[INFO] Computing joint transformations...
[INFO] Generating STL meshes...
[INFO] Creating URDF XML...
[INFO] Creating ROS package structure...
[INFO] DONE. Output: /output_urdf/robot_arm/
```

---

## 4. 출력 파일 구조

```
/output_urdf/robot_arm/
├── package.xml
├── CMakeLists.txt
├── urdf/
│   └── robot_arm.urdf
├── meshes/
│   ├── base_link.stl
│   ├── shoulder_link.stl
│   ├── upper_arm_link.stl
│   ├── forearm_link.stl
│   └── end_effector_link.stl
├── launch/
│   └── load_urdf.launch
└── config/
    └── robot_arm.yaml
```

### 생성된 URDF 예시

```xml
<?xml version="1.0"?>
<robot name="robot_arm">
  <link name="base_link">
    <visual>
      <geometry>
        <mesh filename="package://robot_arm/meshes/base_link.stl"/>
      </geometry>
      <material name="">
        <color rgba="0.8 0.8 0.8 1.0"/>
      </material>
    </visual>
  </link>
  <joint name="base_to_shoulder" type="fixed">
    <parent link="base_link"/>
    <child link="shoulder_link"/>
    <origin xyz="0 0 0.05" rpy="0 0 0"/>
  </joint>
  ...
</robot>
```

> **참고**: 현재 버전은 모든 관절을 **fixed** 타입으로 생성합니다. 로봇의 실제 관절 타입(revolute 등)과 limit 값은 생성된 URDF를 **수동 편집**해야 합니다.

---

## 5. 생성된 ROS 패키지 사용

### 5.1 ROS 패키지 빌드

```bash
# 생성된 패키지를 catkin 워크스페이스로 복사
cp -r D:/urdf_output/robot_arm ~/catkin_ws/src/

# 빌드
cd ~/catkin_ws
catkin_make  # 또는 catkin build
source devel/setup.bash
```

### 5.2 URDF 확인

```bash
# URDF 파싱 테스트
check_urdf ~/catkin_ws/src/robot_arm/urdf/robot_arm.urdf

# 출력 예시
# robot name: robot_arm
#   link name: base_link (1 children)
#   link name: shoulder_link (1 children)
#   ...
```

### 5.3 RViz에서 시각화

```bash
# launch 파일 사용
roslaunch robot_arm load_urdf.launch

# 또는 직접
rosrun urdf_tutorial display.launch model:=$(find robot_arm)/urdf/robot_arm.urdf
```

---

## 6. 추가 수동 작업 (필수)

### 6.1 관절 타입 수정

생성된 URDF는 모든 관절이 `type="fixed"`로 설정됩니다. 실제 로봇에 맞게 수정:

```xml
<!-- 수정 전 -->
<joint name="base_to_shoulder" type="fixed">

<!-- 수정 후 (회전 관절) -->
<joint name="base_to_shoulder" type="revolute">
    <axis xyz="0 0 1"/>
    <limit lower="-3.14" upper="3.14" effort="10.0" velocity="1.0"/>
```

### 6.2 관성(Inertia) 정보 추가

URDF 파일에 inertia 섹션 추가:

```xml
<link name="base_link">
  <inertial>
    <origin xyz="0 0 0" rpy="0 0 0"/>
    <mass value="1.0"/>
    <inertia ixx="0.01" ixy="0" ixz="0" iyy="0.01" iyz="0" izz="0.01"/>
  </inertial>
  ...
</link>
```

### 6.3 메시 품질 개선 (STL → DAE 변환)

urdf-from-step은 STL만 출력하므로, DAE 변환이 필요하다면:

**방법 A: Blender 후처리**

```bash
# STL → Blender → DAE
1. Blender에서 STL 불러오기
2. Shade Smooth + Auto Smooth 적용
3. DAE로 내보내기
4. URDF의 mesh 참조를 .stl → .dae로 수정
```

**방법 B: MeshLab 변환**

```bash
# MeshLab으로 STL → DAE 변환
meshlabserver -i base_link.stl -o base_link.dae
```

---

## 7. 문제 해결

| 문제 | 해결 방법 |
|---|---|
| STEP 파일 로드 실패 | STEP AP203/AP214 형식 확인. CAD에서 다시 내보내기 |
| "No links found" 오류 | STEP 파트명에 "link" 키워드가 있는지 확인 (대소문자 구분?) |
| "No joints found" 오류 | STEP 파트명에 "joint" 키워드가 있는지 확인 |
| 변환은 됐는데 STL이 비어 있음 | OCCT Tessellation 파라미터 문제. Deviation 값을 줄여보기 |
| ROS launch 실행 안 됨 | `source devel/setup.bash` 실행했는지 확인. catkin_make 재실행 |
| URDF에 관절이 모두 fixed로 생성됨 | **정상 동작입니다.** 관절 타입/limit은 수동 편집 필요 |
| Docker 빌드 너무 오래 걸림 | `docker pull reconcycle/urdf-from-step`로 Pre-built 이미지 사용 |
| pythonOCC 임포트 오류 | Conda 환경에서 설치했는지 확인. Windows는 Docker 사용 권장 |

---

## 8. 한계 및 극복 방법

| 한계 | 극복 방법 |
|---|---|
| **STL만 출력** | Blender/MeshLab에서 DAE로 변환 후 URDF 메시 경로 수정 |
| **모든 관절이 fixed** | URDF 파일 직접 편집하여 타입/축/limit 수정 |
| **ROS1 전용** | 생성된 URDF를 ROS2에서도 사용 가능 (`robot_state_publisher` 호환) |
| **naming convention 강제** | STEP을 내보내기 전 CAD에서 파트명 일괄 변경 |
| **유지보수 부족** | 필요시 포크(fork)해서 커스터마이징. 또는 다른 방법(FreeCAD) 병용 |
| **메시 품질 조절 불가** | OCCT 소스코드에서 deviation 파라미터 직접 수정 가능 |
