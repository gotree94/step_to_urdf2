# 방법 3: STEP → STL → Blender + PHOBOS

**최종 결과물**: URDF (또는 SDF/SMURF) + DAE/OBJ 메시 파일
**난이도**: 중간-높음
**라이선스 제약**: ❌ 없음

---

## 개요

STEP 파일을 일단 STL로 변환한 후, Blender 3D에서 PHOBOS 애드온을 사용하여 URDF를 생성하는 방법입니다.

**장점**:
- **메시 품질 최상급**: Blender의 리토폴로지, Subdivision Surface, 법선 편집 기능으로 STL의 삼각형 질감을 획기적으로 개선 가능
- **WYSIWYG 편집**: 관절 위치/축을 3D 뷰에서 직접 보고 조작 가능
- **다양한 출력 포맷**: URDF, SDF, SMURF 모두 지원
- **센서, 관성 등 전체 URDF 스펙 지원**

**단점**:
- 워크플로우가 2단계 (STEP → STL, STL → Blender → URDF)
- Blender의 PHOBOS 워크플로우 학습 필요
- Blender 3.3 LTS 사용 권장 (최신 Blender와 PHOBOS 호환성 확인 필요)

---

## 1. 설치

### 1.1 FreeCAD 설치 (STEP → STL 변환용)

STEP을 STL로 변환하기 위해 FreeCAD가 필요합니다.

1. [FreeCAD 공식 다운로드 페이지](https://www.freecad.org/downloads.php) 접속
2. Windows 64-bit installer 다운로드 및 설치

### 1.2 Blender 설치

1. [Blender 공식 사이트](https://www.blender.org/download/) 접속
2. PHOBOS와의 호환성을 위해 **Blender 3.3 LTS** 권장
   - ⚠ 최신 Blender (4.x)는 PHOBOS와 완전히 호환되지 않을 수 있습니다
   - [Blender 3.3 LTS 다운로드 (과거 릴리즈)](https://www.blender.org/download/lts/3-3/)
3. 설치 후 실행하여 정상 동작 확인

### 1.3 PHOBOS 애드온 설치

**방법 A — ZIP 설치 (권장)**:

1. [PHOBOS GitHub 릴리즈](https://github.com/dfki-ric/phobos/releases)에서 최신 `phobos.zip` 다운로드
   - 또는 직접 클론 후 ZIP으로 압축:
   ```bash
   git clone https://github.com/dfki-ric/phobos.git
   cd phobos
   zip -r phobos.zip phobos/
   ```

2. Blender 실행
3. **Edit → Preferences → Add-ons**
4. 우측 상단 **Install** 버튼 클릭
5. 다운로드한 `phobos.zip` 선택
6. 설치 완료 후 목록에서 "Phobos"를 검색하여 **체크박스 활성화**
7. **Preferences** 창 닫기

**방법 B — Git Clone (개발자용)**:

```bash
# Blender addons 디렉토리로 이동
cd %APPDATA%\Blender Foundation\Blender\3.3\scripts\addons\
git clone https://github.com/dfki-ric/phobos.git
```

### 1.4 PHOBOS Python 종속 패키지 설치

PHOBOS에는 여러 Python 패키지가 필요합니다.

**방법 A — install_requirements.py 사용**:

```bash
cd phobos
블렌더\blender.exe --python install_requirements.py
```

**방법 B — 수동 설치** (Blender 번들 Python 사용):

```bash
# Blender 번들 Python 경로 확인
블렌더설치폴더\3.3\python\bin\python.exe -m pip install numpy scipy pyyaml lxml
```

> **⚠ 중요**: Blender 번들 Python이 아닌 시스템 Python에 설치하면 PHOBOS가 인식하지 못합니다.

### 1.5 설치 확인

1. Blender 재시작
2. **Edit → Preferences → Add-ons** 에서 "Phobos"가 체크되어 있는지 확인
3. 3D 뷰 우측 **N 키**를 눌러 사이드바 열기
4. **Phobos** 탭이 나타나야 함

---

## 2. STEP → STL 변환 (FreeCAD)

### 2.1 FreeCAD에서 STEP 불러오기

1. **FreeCAD 실행**
2. **File → Open** → STEP 파일 선택
3. 각 링크가 별도의 Body로 임포트되었는지 확인

### 2.2 각 링크를 STL로 내보내기

**메시 품질 설정 (중요)**:

1. **Edit → Preferences → Mesh Design → Tessellation**
2. **Maximum deviation**: `0.05 mm` (Blender에서 후처리할 것이므로 너무 촘촘할 필요 없음)

**내보내기**:

각 링크를 개별 STL로 내보내기:

```python
# FreeCAD Python Console에서 실행
import Mesh

links = ["base_link", "shoulder_link", "upper_arm_link", "forearm_link", "end_effector_link"]

for name in links:
    obj = FreeCAD.ActiveDocument.getObject(name)
    if obj:
        Mesh.export([obj], f"D:/stl_files/{name}.stl")
        print(f"Exported: {name}.stl")
```

또는 **File → Export**에서 STL 포맷 선택 후 수동으로 하나씩 내보내기.

---

## 3. Blender에서 STL 불러오기 및 메시 가공

### 3.1 STL 불러오기

1. **Blender 실행**
2. **File → Import → STL (.stl)**
3. 첫 번째 STL 파일 (예: `base_link.stl`) 선택
4. **File → Import → STL** 로 나머지 파일 모두 추가 불러오기
   - 또는 한 번에 여러 STL 선택 가능

> 모든 링크가 같은 Blender 씬(scene)에 각각 별도의 **Object**로 임포트되어야 합니다.

### 3.2 Object 이름 정리

각 Object의 이름을 URDF 링크 이름과 일치시킵니다:

1. **Outliner** (우측 상단 패널)에서 Object 확인
2. 각 Object 선택 → **F2 키** → 이름 변경 (예: `base_link`, `shoulder_link`, ...)

### 3.3 메시 품질 개선 (핵심 단계)

STL의 삼각형 면이 보기 싫다면 다음 기법들을 적용합니다:

#### A. Shade Smooth + Auto Smooth (빠른 개선)

1. 모든 Object 선택 (A 키)
2. Object 우클릭 → **Shade Smooth**
3. Object 우클릭 → 상단 메뉴 **Face → Shade Smooth** (또는 우측 메뉴)
4. **Object Data Properties** (녹색 삼각형 아이콘):
   - **Normals → Auto Smooth** 체크
   - 각도: `30°` (기본값)
5. 3D 뷰에서 결과 확인

> **효과**: 면과 면 사이의 법선을 평균내어 삼각형 경계가 덜 보이게 만듭니다. 가장 간단하면서도 효과적입니다.

#### B. Subdivision Surface Modifier (고급)

1. Object 선택
2. **Modifiers** (파란색 렌치 아이콘) → **Add Modifier → Subdivision Surface**
3. 설정:
   - **Levels Viewport**: `1` 또는 `2`
   - **Levels Render**: `2` 또는 `3`
4. **Apply** 버튼 클릭 (Modifier 우측 ▼ → Apply)

> ⚠ 폴리곤 수가 급증합니다. 필요 이상으로 높은 레벨은 삼가세요.

#### C. Decimate Modifier (폴리곤 최적화)

너무 많은 폴리곤을 줄여야 한다면:

1. Object 선택
2. **Add Modifier → Decimate**
3. **Ratio**: `0.1` ~ `0.5` (작을수록 폴리곤 감소)
4. 결과 확인 후 **Apply**

> **권장 워크플로우**: Subdivision Surface (레벨 1~2) → Decimate (0.3~0.5) → Shade Smooth + Auto Smooth

---

## 4. PHOBOS로 URDF 작업

### 4.1 PHOBOS 활성화

1. 3D 뷰에서 **N 키**로 사이드바 열기
2. **Phobos** 탭 선택
3. 또는 상단 메뉴: **Phobos** 항목 확인

### 4.2 뼈대(Bone) 구조 생성 (관절 트리)

PHOBOS는 Blender의 **Bone(뼈대)** 시스템을 사용하여 URDF의 관절-링크 구조를 표현합니다.

**수동 생성법**:

1. **Armature** 추가: **Add → Armature → Single Bone**
2. Armature 이름 변경 (예: `robot_armature`)
3. **Edit Mode** (Tab 키)로 전환
4. 각 관절 위치에 Bone 생성:
   - Bone의 **Root**(시작점) = 관절 원점
   - Bone의 **Tip**(끝점) = 자식 링크 방향
5. 부모-자식 관계 설정:
   - Bone 선택 → **Parent** 설정 (Bone 속성에서)
   - Armature의 Bone 계층 구조 = URDF의 관절 트리

**PHOBOS 도구 사용법**:

1. 사이드바 Phobos 탭 → **Joints** 섹션
2. **Add Joint** 버튼:
   - **Name**: 관절 이름
   - **Type**: revolute / prismatic / fixed 등
   - **Parent/Child**: 링크 선택
3. 3D 뷰에서 Bone 위치/회전 조정 후 **Apply Transform**

### 4.3 링크(Link)와 메시 연결

각 Object를 URDF 링크로 지정:

1. 메시 Object 선택 (예: `base_link`)
2. 사이드바 Phobos 탭 → **Links** 섹션
3. **Add Link** 버튼
4. 설정:
   - **Link Name**: URDF 링크 이름 (일반적으로 Object 이름과 동일)
   - **Visual Mesh**: ✅ (시각용 메시로 사용)
   - **Collision Mesh**: ✅ (충돌용으로도 사용)
   - **Inertial**: 관성 정보 입력 (Mass, Inertia Tensor)

### 4.4 관절 파라미터 설정

각 Bone(=관절)의 상세 파라미터:

1. **Edit Mode**에서 Bone 선택
2. 사이드바 Phobos 탭 → **Joint Properties**
3. 설정:
   - **Type**: revolute / prismatic / continuous / fixed
   - **Axis**: X, Y, Z (회전축, Bone의 local 좌표계 기준)
   - **Limit Lower/Upper**: 회전 각도 제한 (radians)
   - **Limit Effort**: 최대 토크 (Nm)
   - **Limit Velocity**: 최대 속도 (rad/s)
4. **Apply** 클릭

### 4.5 관성(Inertia) 설정

1. 링크 선택 → Phobos **Inertial** 섹션
2. **Mass**: kg 단위 질량
3. **Inertia Tensor**: ixx, ixy, ixz, iyy, iyz, izz 값 입력
   - PHOBOS가 메시 형상으로부터 자동 계산: **Auto Calculate** 버튼
   - 또는 수동 입력
4. **Center of Mass**: 질량 중심 위치 (링크 좌표계 기준)

---

## 5. URDF 내보내기

### 5.1 내보내기 설정

1. 사이드바 Phobos 탭 → **Export** 섹션
2. **Export Format**: `URDF` 선택
3. 옵션 설정:
   - **Export Path**: 출력 폴더 지정
   - **Mesh Format**: `Collada (.dae)` 권장 (STL보다 품질 우수)
   - ✅ **Export Visual Meshes**
   - ✅ **Export Collision Meshes** (별도로 간소화된 메시 사용 가능)
   - ✅ **Export Inertial Data**
4. **Export URDF** 버튼 클릭

### 5.2 CLI 도구로 내보내기 (선택사항)

```bash
# Blender를 헤드리스로 실행하여 내보내기
blender --background robot.blend --python export_phobos.py
```

`export_phobos.py` 예시:
```python
import bpy
import phobos

# PHOBOS를 통해 내보내기
phobos.exports.urdf.exportUrdf(
    bpy.context,
    filepath="D:/output/robot.urdf",
    mesh_format="dae"
)
```

---

## 6. 출력 파일 구조

```
output/
├── robot.urdf
└── meshes/
    ├── base_link.dae
    ├── shoulder_link.dae
    ├── upper_arm_link.dae
    ├── forearm_link.dae
    └── end_effector_link.dae
```

---

## 7. URDF 검증

```bash
# yourdfpy로 시각화
pip install yourdfpy
yourdfpy output/robot.urdf --visualize

# ROS 환경 (ROS1)
check_urdf output/robot.urdf
roslaunch urdf_tutorial display.launch model:=output/robot.urdf

# ROS2
ros2 run urdf_tutorial display.launch.py model:=output/robot.urdf
```

---

## 8. 문제 해결

| 문제 | 해결 방법 |
|---|---|
| PHOBOS 탭이 안 보임 | Addon 설치 확인. Blender 3.3 LTS 사용 권장. N키로 사이드바 열기 |
| Bone과 Mesh가 분리됨 | Bone 선택 → Mesh 선택 → **Ctrl + P** → **Armature Deform** |
| URDF 링크 순서가 틀림 | Bone 계층 구조를 부모-자식 순서로 올바르게 구성 |
| DAE 색상 손실 | Blender 상단 메뉴 **Render Properties → Color Management → View Transform: Standard**로 설정 |
| 내보내기 오류 | Python Console에서 오류 메시지 확인. 필수 패키지(lxml, numpy) 설치 확인 |
| Auto Smooth 적용 안 됨 | 메시에 **Edge Split Modifier** 추가 후 Auto Smooth와 함께 사용 |
| 폴리곤이 너무 많음 | Decimate Modifier로 적절히 감소 (Ratio 0.3~0.5) |

---

## 9. 고급: 메시 품질 최적화 요령

1. **STL 변환 시** FreeCAD tessellation은 `0.05mm` 정도로 적당히 설정 (Blender에서 후처리할 것이므로 너무 미세할 필요 없음)
2. **Blender 워크플로우**:
   - Shade Smooth + Auto Smooth (필수, 가장 큰 효과)
   - 필요시 Subdivision Surface (레벨 1만으로도 충분한 경우多)
   - Decimate로 폴리곤 수를 적절히 제한 (RViz 성능 고려)
3. **DAE 내보내기**: STL보다 DAE가 법선 정보를 유지하므로 훨씬 부드럽게 보입니다
4. **Collision Mesh는 별도로**: Visual은 DAE(고품질), Collision은 Decimate로 단순화한 STL 사용 가능
