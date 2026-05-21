# 방법 1: FreeCAD + CROSS Workbench ⭐ (최고 추천)

**최종 결과물**: URDF (또는 XACRO) + DAE/STL 메시 파일
**난이도**: 중간
**라이선스 제약**: ❌ 없음

---

## 개요

FreeCAD(무료 오픈소스 CAD)에 CROSS Workbench 플러그인을 설치하여 STEP 파일을 불러온 후 URDF로 내보내는 방법입니다.

**장점**:
- 단일 툴체인 (FreeCAD 하나로 CAD 읽기 → 메시 변환 → URDF 내보내기까지 완료)
- DAE(Collada) 포맷 출력 → STL보다 부드러운 메시 표현 가능
- Tessellation 파라미터를 직접 제어하여 메시 품질 조절 가능
- XACRO 지원 → ROS2 호환성 우수
- 완전 무료, 라이선스 제약 없음

**단점**:
- 각 링크/관절을 수동으로 지정해야 함
- FreeCAD와 ROS 환경에 대한 기본 이해 필요

---

## 1. 설치

### 1.1 FreeCAD 설치

**Windows (권장)**:
1. [FreeCAD 공식 다운로드 페이지](https://www.freecad.org/downloads.php) 접속
2. 최신 안정화 버전 (≥ v0.21.2 권장) Windows 64-bit installer 다운로드
3. 설치 프로그램 실행 → 기본 설정 그대로 설치
4. FreeCAD 실행하여 정상 동작 확인

> **⚠ 참고**: CROSS Workbench는 FreeCAD v0.21.2 이상에서 공식 Addon Manager를 통한 설치를 권장합니다. 가능하면 최신 릴리즈를 받으세요.

### 1.2 CROSS Workbench 설치

**방법 A — Addon Manager 사용 (권장)**:

1. FreeCAD 실행
2. 메뉴: **Edit → Preferences** (또는 **Ctrl + ,**)
3. 좌측 카테고리에서 **Addon Manager** 선택
4. 오른쪽 **Custom repositories** 섹션에서 **+** 버튼 클릭
5. 다음 정보 입력:
   - **Repository URL**: `https://github.com/galou/freecad.cross.git`
   - **Branch**: `main`
6. **OK** 클릭하여 Preferences 닫기
7. 메뉴: **Tools → Addon manager**
8. Addon Manager 창에서 "CROSS" 검색
9. CROSS Workbench 찾아서 **Install** 버튼 클릭
10. 설치 완료 후 FreeCAD 재시작

**방법 B — 수동 설치**:

```bash
# FreeCAD Mod 디렉토리로 이동 (Windows 예시)
cd %APPDATA%\FreeCAD\Mod\
# 또는: C:\Users\<사용자명>\AppData\Roaming\FreeCAD\Mod\

# GitHub에서 클론
git clone https://github.com/galou/freecad.cross.git Cross
```

### 1.3 Python 종속 패키지 확인

CROSS Workbench는 URDF 내보내기에 다음 Python 패키지를 사용합니다:

```bash
pip install beautifulsoup4 lxml
```

FreeCAD 번들 Python이 아닌 시스템 Python에 설치해야 할 수 있습니다. CROSS가 내보내기 시 자동으로 체크하고 부재 시 안내합니다.

---

## 2. 준비: STEP 파일 요구사항

변환할 STEP 파일은 다음과 같은 구조여야 합니다:

- 로봇의 **각 링크(link)** 가 **별도의 Part** 또는 **Body**로 분리되어 있어야 함
- 각 Part의 **배치(Placement)** 가 올바른 상대 좌표계를 가져야 함
- 조립품(Assembly) 구조에서 각 파트가 올바르게 정렬되어 있어야 함

> **TIP**: STEP 파일이 하나의 단일 바디로만 되어 있다면, FreeCAD에서 슬라이스/컷 등으로 분할하거나 STEP 자체를 다시 내보내야 할 수 있습니다.

---

## 3. FreeCAD에서 STEP 불러오기

1. FreeCAD 실행
2. **File → Open** (또는 **Ctrl + O**)
3. STEP 파일 선택 (`.step` 또는 `.stp`)
4. Import 설정:
   - 기본 Import 설정으로 충분
   - 만약 옵션이 나오면 **"Import as single document"** 선택 (여러 파트가 각각 FreeCAD Body로 임포트됨)
5. STEP 파일이 로드되면 좌측 **Combo View**에 모델 트리가 표시됨
   - 각 파트(링크)가 개별 항목으로 보여야 함
   - 예: `base_link`, `shoulder_link`, `upper_arm_link` 등

---

## 4. CROSS Workbench로 URDF 작업

### 4.1 CROSS Workbench 활성화

1. FreeCAD 상단의 Workbench 드롭다운 메뉴 클릭
2. 목록에서 **"CROSS"** 선택
   - 목록에 없으면 FreeCAD 재시작 후 다시 확인

### 4.2 각 링크(Link) 설정

각 FreeCAD Body를 URDF의 **링크(Link)** 로 지정합니다:

1. Combo View (좌측 트리)에서 첫 번째 Body 선택 (예: `base_link`)
2. CROSS 도구모음에서 **"Create URDF Link"** 버튼 클릭
   - (또는 메뉴: CROSS → Create URDF Link)
3. 링크 속성 설정 다이얼로그:
   - **Link Name**: URDF에서 사용할 이름 (예: `base_link`)
   - **Visual Mesh Export**: 메시 포맷 선택 (**Collada (.dae)** 권장)
   - **Collision Mesh Export**: 충돌용 메시 (일반적으로 Visual과 동일)
   - **Mass / Inertia**: 필요시 입력 (나중에도 수정 가능)
4. **OK** 클릭
5. 모든 Body(링크)에 대해 반복

### 4.3 각 관절(Joint) 설정

링크 간 **관절(Joint)** 을 정의합니다:

1. CROSS 도구모음에서 **"Create URDF Joint"** 버튼 클릭
2. 관절 속성 설정:
   - **Joint Name**: 예: `base_to_shoulder`
   - **Type**: 관절 종류 선택
     - `revolute` (회전) — 일반적인 로봇 팔 관절
     - `prismatic` (직동)
     - `continuous` (무한 회전)
     - `fixed` (고정)
   - **Parent Link**: 부모 링크 선택 (예: `base_link`)
   - **Child Link**: 자식 링크 선택 (예: `shoulder_link`)
   - **Origin**: 관절 좌표계 위치 (X, Y, Z, Roll, Pitch, Yaw)
     - FreeCAD 뷰에서 가져오거나 수동 입력
   - **Axis**: 회전/이동 축 (예: `0 0 1` = Z축 회전)
   - **Limits**: (revolute/prismatic) 하한/상한 각도 또는 거리
3. **OK** 클릭
4. 모든 관절에 대해 반복

### 4.4 URDF 내보내기

1. 모든 링크와 관절 설정 완료 후
2. CROSS 도구모음에서 **"Export URDF"** 버튼 클릭
   - (또는 메뉴: CROSS → Export URDF)
3. 저장 설정:
   - **Export directory**: 출력 폴더 지정
   - **Export format**: `URDF` 또는 `XACRO` 선택
     - ROS1 → **URDF**
     - ROS2 → **XACRO** 권장
4. **Export** 버튼 클릭

> **문제 발생 시**: Python 콘솔에서 직접 내보내기:
> ```python
> import ExportURDF
> ExportURDF.export_urdf()
> ```
> (FreeCAD Python Console에서 실행)

---

## 5. 출력 파일 구조

```
출력폴더/
├── robot.urdf              (또는 robot.xacro)
├── meshes/
│   ├── base_link.dae
│   ├── shoulder_link.dae
│   ├── upper_arm_link.dae
│   └── ...
└── config/
    └── robot.yaml          (선택사항)
```

---

## 6. URDF 검증 및 활용

### URDF 파일 확인

```bash
# ROS1 환경
check_urdf robot.urdf

# yourdfpy (Python, ROS 없이)
pip install yourdfpy
yourdfpy robot.urdf --visualize
```

### RViz에서 시각화 (ROS 환경)

```bash
# ROS1
roslaunch urdf_tutorial display.launch model:=robot.urdf

# ROS2
urdf_to_graphiz robot.urdf
ros2 run urdf_tutorial display.launch.py model:=robot.urdf
```

---

## 7. 메시 품질 개선 (중요)

DAE로 내보내도 기본 tessellation이 너무 거칠 수 있습니다. FreeCAD에서 tessellation 품질을 높이려면:

### 방법 A: 편집 메쉬 파라미터 변경

1. FreeCAD 메뉴: **Edit → Preferences**
2. 카테고리: **Mesh Design** (또는 **Mesh**)
3. **Tessellation** 탭:
   - **Maximum deviation**: `0.01 mm` (기본값보다 작게. 0.05 → 0.01로)
   - **Angular deflection**: `10 deg` → `5 deg` 또는 더 작게
4. **OK** 클릭 후 STEP 다시 불러오기

### 방법 B: 각 링크별 내보내기 시 설정

CROSS Workbench에서 URDF Link 생성 시 Mesh Export 옵션에서 **세부 설정** 버튼 확인:
- Deviation 값을 낮출수록 폴리곤 수 ↑, 파일 크기 ↑, 품질 ↑
- `0.1` → `0.01` 로 낮추면 약 10배 정밀해짐

### 방법 C: 내보낸 DAE를 Blender에서 리파인

1. Blender에서 DAE 불러오기
2. **Modifier → Subdivision Surface** 적용
3. **Shade Smooth** + **Auto Smooth** 활성화
4. 다시 DAE로 내보내기

---

## 8. 문제 해결

| 문제 | 해결 방법 |
|---|---|
| CROSS Workbench가 목록에 안 보임 | FreeCAD 재시작. Addon Manager에서 설치 확인. FreeCAD 버전 ≥ 0.21.2 확인 |
| STEP 불러오기 실패 | 다른 STEP 파일로 테스트. FreeCAD 재시작. STEP 내보내기 옵션(AP203/AP214) 변경 |
| URDF 내보내기 버튼 안 눌림 | 모든 링크에 Link 속성이 설정되었는지 확인. 최소 2개 이상의 링크가 연결된 관절 필요 |
| DAE 색상 안 나옴 | STEP의 색상 정보 손실 가능. FreeCAD에서 각 Body에 재질/색상 수동 지정 후 재내보내기 |
| meshes 폴더가 비어 있음 | Export 시 메시 옵션을 "Export meshes"로 설정했는지 확인 |
| 관절 위치가 틀림 | 각 Body의 Placement (좌표계)가 올바른지 확인. STEP 파일의 원점이 정확한지 검토 |

---

## 9. 고급: XACRO 내보내기

ROS2를 사용한다면 XACRO가 더 좋은 선택입니다:

- **CROSS**는 XACRO 내보내기를 기본 지원합니다
- 변수화, 매크로, 수학 연산 사용 가능
- 여러 로봇을 조합하는 워크셀(workcell) 개념 지원

내보내기 시 **Export format**을 `XACRO`로 선택하면 `.xacro` 파일이 생성됩니다.
