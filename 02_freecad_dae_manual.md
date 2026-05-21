# 방법 2: FreeCAD → DAE + 수동 URDF

**최종 결과물**: URDF (직접 XML 작성) + DAE 메시 파일
**난이도**: 높음
**라이선스 제약**: ❌ 없음

---

## 개요

FreeCAD로 STEP 파일에서 각 링크를 DAE 메시로 내보낸 후, URDF XML을 **직접 수동 작성**하는 방법입니다.

**장점**:
- URDF 구조를 100% 완전히 제어 가능
- 불필요한 자동화 툴 의존성 없음
- DAE 포맷 사용 → STL보다 우수한 시각적 품질
- 작은 로봇(2~5개 링크)에 적합
- 라이선스 제약 전혀 없음

**단점**:
- 링크 수가 많아질수록 작업량이 급증
- 관절 좌표계 계산을 수동으로 해야 함 (가장 실수하기 쉬운 부분)
- URDF XML 문법 숙지 필수
- 관성(inertia) 정보를 별도로 계산/입력해야 함

---

## 1. 설치

### 1.1 FreeCAD 설치

**Windows**:
1. [FreeCAD 공식 다운로드 페이지](https://www.freecad.org/downloads.php) 접속
2. 최신 안정화 버전 Windows 64-bit installer 다운로드
3. 설치 프로그램 실행

### 1.2 Python 종속 패키지 (선택사항)

URDF 검증용:

```bash
pip install yourdfpy
```

---

## 2. STEP 파일 준비 및 FreeCAD에서 불러오기

1. **FreeCAD 실행**
2. **File → Open** (`Ctrl + O`) → STEP 파일 선택
3. STEP이 올바르게 임포트되었는지 확인:
   - Combo View에서 각 링크가 별도의 **Body** 또는 **Part**로 표시되어야 함
   - 같은 트리 레벨에 있는지 확인 (중첩 불필요)

### 올바른 STEP 구조 예시

```
Assembly/
├── base_link (Body)
├── shoulder_link (Body)
├── upper_arm_link (Body)
├── forearm_link (Body)
└── end_effector_link (Body)
```

### 만약 하나의 Body로 합쳐져 있다면?

STEP 파일이 단일 Body로 되어 있으면 수동 분할이 필요합니다:

1. **Part Workbench**로 전환
2. **Part → Split** 또는 **Part → Slice** 사용
3. 또는 **Draft Workbench** → **Draft to Sketch** → **Part Design → Pad/Pocket** 으로 분할
4. 또는 **Mesh Workbench** → Split by Components

---

## 3. 각 링크 DAE로 내보내기

### 3.1 메시 품질 설정 (중요)

DAE 내보내기 전에 tessellation 품질을 설정합니다:

1. **Edit → Preferences**
2. **Mesh Design** 카테고리
3. **Tessellation** 탭:
   - **Maximum deviation**: `0.01 mm` (작을수록 정밀)
   - **Angular deflection**: `5 deg` (작을수록 부드러움)
4. **OK**

### 3.2 각 링크를 DAE로 내보내기

링크 개수만큼 반복:

1. Combo View에서 내보낼 Body 선택
2. **File → Export** (`Ctrl + E`)
3. 파일 형식에서 **"Collada (*.dae)"** 선택
4. 파일명: `base_link.dae`, `shoulder_link.dae`, ...
5. **저장**
6. 모든 링크에 대해 반복

> **TIP**: 일괄 내보내기 Python 스크립트:
> ```python
> import FreeCAD, Mesh
> 
> # 내보낼 Body 목록 (Combo View 이름 기준)
> bodies = ["base_link", "shoulder_link", "upper_arm_link", "forearm_link", "end_effector_link"]
> 
> for name in bodies:
>     obj = FreeCAD.ActiveDocument.getObject(name)
>     if obj and obj.isDerivedFrom("Part::Feature"):
>         Mesh.export([obj], f"D:/export/{name}.dae")
>         print(f"Exported: {name}.dae")
> ```
> FreeCAD **Python Console** (View → Panels → Python Console)에 붙여넣고 실행하세요.

### 3.3 참고: STL로 내보내기 (DAE가 안 될 경우)

```python
import Mesh
obj = FreeCAD.ActiveDocument.getObject("base_link")
Mesh.export([obj], "D:/export/base_link.stl")
```

---

## 4. URDF XML 수동 작성

### 4.1 URDF 기본 구조

```xml
<?xml version="1.0"?>
<robot name="my_robot">

  <!-- ====== 링크 1: base_link ====== -->
  <link name="base_link">
    <visual>
      <origin xyz="0 0 0" rpy="0 0 0"/>
      <geometry>
        <mesh filename="meshes/base_link.dae" scale="1 1 1"/>
      </geometry>
      <material name="base_color">
        <color rgba="0.5 0.5 0.5 1.0"/>
      </material>
    </visual>
    <collision>
      <origin xyz="0 0 0" rpy="0 0 0"/>
      <geometry>
        <mesh filename="meshes/base_link.stl" scale="1 1 1"/>
      </geometry>
    </collision>
    <inertial>
      <origin xyz="0 0 0" rpy="0 0 0"/>
      <mass value="1.0"/>
      <inertia ixx="0.01" ixy="0" ixz="0" iyy="0.01" iyz="0" izz="0.01"/>
    </inertial>
  </link>

  <!-- ====== 링크 2: shoulder_link ====== -->
  <link name="shoulder_link">
    <visual>
      <origin xyz="0 0 0" rpy="0 0 0"/>
      <geometry>
        <mesh filename="meshes/shoulder_link.dae" scale="1 1 1"/>
      </geometry>
    </visual>
    ... (collision, inertial 생략)
  </link>

  <!-- ====== 관절: base_to_shoulder ====== -->
  <joint name="base_to_shoulder" type="revolute">
    <parent link="base_link"/>
    <child link="shoulder_link"/>
    <origin xyz="0 0 0.1" rpy="0 0 0"/>
    <axis xyz="0 0 1"/>
    <limit lower="-3.14" upper="3.14" effort="10" velocity="1"/>
  </joint>

  ... (추가 링크와 관절)

</robot>
```

### 4.2 각 섹션 상세 설명

#### `<link>` 섹션

| 요소 | 설명 | 필수 |
|---|---|---|
| `<visual>` | 시각적 표현. DAE 메시 참조 | ✅ |
| `<collision>` | 충돌 검출용. 일반적으로 STL 또는 단순화된 메시 | ✅ |
| `<inertial>` | 관성 정보. `<mass>` + `<inertia>` 6개 값 | 권장 |
| `<material>` | 색상/재질 (DAE 자체에 색상이 있으면 생략 가능) | 선택 |

#### `<joint>` 섹션

| 속성 | 설명 |
|---|---|
| `name` | 관절 이름 (고유해야 함) |
| `type` | `revolute`, `prismatic`, `continuous`, `fixed`, `floating`, `planar` |
| `<parent>` | 부모 링크 이름 |
| `<child>` | 자식 링크 이름 |
| `<origin>` | 자식 링크 좌표계의 부모 기준 위치 (xyz: m, rpy: rad) |
| `<axis>` | 관절 회전/이동 축 방향 (부모 좌표계 기준) |
| `<limit>` | `lower`, `upper` (rad 또는 m), `effort` (N), `velocity` (m/s) |

### 4.3 관절 origin 계산 (가장 중요한 부분)

관절의 `<origin xyz="..." rpy="..."/>`는 **부모 링크 좌표계에서 자식 링크 좌표계까지의 변환**입니다.

**계산 방법 1: FreeCAD에서 직접 측정**

1. **Measure** Workbench로 전환
2. `base_link`의 원점 → `shoulder_link`의 원점까지의 거리 측정
3. XYZ 거리를 `<origin>`에 입력
4. 회전(RPY)은 두 좌표계 간의 오일러 각 차이

**계산 방법 2: Placement 속성 확인**

1. Combo View에서 자식 Body 선택
2. 속성(Property) 탭 → **Placement** 항목 확인
3. Placement의 Position (X, Y, Z)와 Rotation을 origin으로 사용
4. Rotation은 Quaternion → Euler 변환 필요:

```python
# FreeCAD Python Console
obj = FreeCAD.ActiveDocument.getObject("shoulder_link")
pl = obj.Placement
pos = pl.Base
rot = pl.Rotation
rpy = rot.toEuler()  # (Yaw, Pitch, Roll) in rad
print(f"xyz={pos.x} {pos.y} {pos.z}  rpy={rpy[2]} {rpy[1]} {rpy[0]}")
```

> **⚠ 주의**: FreeCAD의 Rotation.toEuler()는 ZYX 순서 (Yaw, Pitch, Roll). URDF의 rpy는 동일한 ZYX 순서이므로 그대로 사용 가능합니다.

### 4.4 inertia 계산

URDF 시뮬레이션(특히 physics)을 위해서는 각 링크의 관성 텐서가 필요합니다.

**FreeCAD에서 자동 계산**:

```python
# FreeCAD Python Console
obj = FreeCAD.ActiveDocument.getObject("base_link")
mass = obj.Mass  # FreeCAD 재질 설정 필요
inertia = obj.MatrixOfInertia
print(f"Inertia matrix:\n{inertia}")
```

또는 [MeshLab](https://www.meshlab.net/)이나 온라인 계산기 활용.

간략한 근사치로 충분하다면 박스/원기둥 근사 사용:

```python
def box_inertia(mass, width, depth, height):
    ixx = (1/12) * mass * (depth**2 + height**2)
    iyy = (1/12) * mass * (width**2 + height**2)
    izz = (1/12) * mass * (width**2 + depth**2)
    return ixx, iyy, izz
```

---

## 5. 출력 디렉토리 구조 예시

```
my_robot/
├── robot.urdf
└── meshes/
    ├── base_link.dae
    ├── shoulder_link.dae
    ├── upper_arm_link.dae
    ├── forearm_link.dae
    └── end_effector_link.dae
```

---

## 6. URDF 검증

### 방법 A: 터미널에서 검증

```bash
# ROS1 check_urdf (ROS 환경 필요)
check_urdf robot.urdf

# yourdfpy (Python, 권장)
pip install yourdfpy
yourdfpy robot.urdf --visualize
```

### 방법 B: 라이브러리로 검증

```python
from yourdfpy import URDF

robot = URDF.load("robot.urdf")
print(f"Links: {[l.name for l in robot.links]}")
print(f"Joints: {[j.name for j in robot.joints]}")
robot.show()  # 3D 시각화
```

---

## 7. 메시 품질 개선 팁

1. **DAE의 법선 정보 활용**: DAE는 vertex normal을 포함하므로 STL보다 같은 폴리곤 수로도 부드럽게 보입니다
2. **FreeCAD Tessellation 조정**: `Preferences → Mesh → Tessellation`의 deviation 값을 낮출수록 정밀
3. **너무 정밀하면** 파일 크기가 커지고 RViz/Webots 등에서 느려질 수 있습니다. `0.01~0.05mm` 사이가 적당
4. **Blender로 후처리**: DAE를 Blender에서 열고 **Shade Smooth + Auto Smooth** 적용 후 재내보내기

---

## 8. 문제 해결

| 문제 | 원인/해결 |
|---|---|
| URDF 파싱 에러 | XML 문법 오류 확인. 태그가 올바르게 닫혔는지, 경로가 정확한지 |
| RViz에 메시 안 보임 | `<mesh filename>` 경로가 URDF 파일 기준 **상대 경로**인지 확인 |
| 관절이 이상한 방향으로 회전 | `<axis>` 값 오류. 회전축 벡터 확인 (정규화된 단위 벡터) |
| 관절 위치가 틀림 | `<origin>` 값 오류. FreeCAD Placement와 실제 관절 위치 차이 확인 |
| inertia 계산 실패 | FreeCAD에 재질(Material)이 설정되어 있어야 `Mass` 속성 사용 가능 |
| DAE 색상 손실 | STEP 자체에 색상 정보가 없는 경우 FreeCAD에서 각 Body에 색상 지정 후 재내보내기 |
