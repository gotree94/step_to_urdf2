# STEP → URDF 변환 가이드

CAD 파일(STEP)을 로봇 기술용 URDF 포맷으로 변환하는 **4가지 무료 방법**을 상세히 정리합니다.

---

## 목차

1. [FreeCAD + CROSS Workbench](./01_freecad_cross.md) — ⭐ **최고 추천**
2. [FreeCAD → DAE + 수동 URDF](./02_freecad_dae_manual.md)
3. [STEP → STL → Blender + PHOBOS](./03_blender_phobos.md)
4. [urdf-from-step (자동 변환)](./04_urdf_from_step.md)

---

## 방법별 특징 비교

| 항목 | FreeCAD + CROSS | FreeCAD → DAE + 수동 | Blender + PHOBOS | urdf-from-step |
|---|---|---|---|---|
| **라이선스 제약** | ❌ 없음 | ❌ 없음 | ❌ 없음 | ❌ 없음 |
| **메시 포맷** | DAE (권장), STL | DAE | DAE, OBJ, STL | STL |
| **메시 품질 제어** | Tessellation 파라미터 직접 설정 | Tessellation 파라미터 직접 설정 | Blender 리토폴로지/법선 편집 가능 | OCCT 기본 tessellation |
| **관절/링크 설정** | GUI (CROSS Workbench) | 수동 (XML 직접 편집) | GUI (PHOBOS, WYSIWYG) | 키워드 기반 자동 |
| **자동화 수준** | 중간 | 낮음 | 중간 | 높음 |
| **ROS 버전** | ROS1 + ROS2 (XACRO) | 무관 | ROS1 + ROS2 | ROS1 |
| **난이도** | 중간 | 높음 | 중간-높음 | 낮음 (설치만 되면) |
| **STEP 네이티브** | ✅ FreeCAD가 직접 읽음 | ✅ FreeCAD가 직접 읽음 | ❌ STL 가공 필요 | ✅ OCCT 직접 파싱 |
| **결과물 품질** | ⭐ 우수 | ★★★★★ (완전 제어) | ⭐⭐⭐ (메시 최상) | ★★★ (STL 한계) |

---

## 용어 설명

| 용어 | 설명 |
|---|---|
| **STEP (.step/.stp)** | ISO 10303 표준의 CAD 교환 포맷. B-rep (경계표현) 방식으로 형상을 정의. |
| **URDF (.urdf)** | ROS에서 로봇을 기술하는 XML 포맷. 링크(link)와 관절(joint)의 트리 구조 + 메시 파일 참조. |
| **STL (.stl)** | 삼각형 메시 포맷. 색상/질감 정보 없음. 면이 평평하게 보이는 단점. |
| **DAE (.dae)** | Collada 포맷. 정점 법선(vertex normal)을 포함하여 같은 폴리곤 수로도 더 부드럽게 표현 가능. 색상 정보 유지. |
| **OBJ (.obj)** | Wavefront 포맷. DAE와 유사하게 법선/텍스처 정보 포함 가능. |
| **XACRO (.xacro)** | URDF의 매크로 확장판. 파라미터화, 수학 연산, 파일 include 지원. ROS2에서 선호됨. |
| **OpenCASCADE (OCCT)** | STEP 파싱에 사용되는 핵심 CAD 커널. FreeCAD와 urdf-from-step이 사용. |
| **Tessellation** | B-rep(곡면)을 삼각형 메시로 변환하는 과정. deviation 값이 작을수록 정밀. |

---

## 빠른 선택 가이드

| 상황 | 추천 방법 |
|---|---|
| **단일 툴체인, 적당한 품질, ROS2 지원까지** | [01 FreeCAD + CROSS](./01_freecad_cross.md) |
| **메시 품질이 가장 중요 (시각적 완성도)** | [03 Blender + PHOBOS](./03_blender_phobos.md) |
| **완전 수동 제어, 특수한 요구사항** | [02 FreeCAD → DAE + 수동](./02_freecad_dae_manual.md) |
| **STEP 파일의 파트명에 joint/link가 이미 명명됨, 빠른 자동 변환** | [04 urdf-from-step](./04_urdf_from_step.md) |

---

> 각 방법의 상세 설치 과정과 실행 절차는 개별 문서를 참조하세요.
