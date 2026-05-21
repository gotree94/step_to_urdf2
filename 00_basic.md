# URDF 생성을 위한 기준 좌표계 정의도

> 이 문서는 단순한 "기구도"가 아니라, **URDF(Universal Robot Description Format)를 만들기 위한 핵심 기준 좌표계 정의도**에 대한 설명입니다.
>
> 로봇의 각 링크(Link)와 관절(Joint)의 **위치 / 방향 / 회전축**을 정의하기 위한 설계 문서입니다.
>
> 이 그림 하나로 다음을 만들 수 있습니다:
> - URDF
> - DH Parameter
> - Forward Kinematics
> - Inverse Kinematics
> - Jacobian
> - ROS2 TF Tree
> - MoveIt Motion Planning
> - Isaac Sim / Gazebo 모델

---

## 1. 전체 구조

그림의 핵심 구조는 다음과 같습니다:

```
[Base]
  ↓
  J0
  ↓
Link1
  ↓
  J1
  ↓
Link2
  ↓
  J2
  ↓
  ...
```

즉, **Link -- Joint -- Link -- Joint** 구조입니다.
URDF도 정확히 같은 구조로 이루어집니다.

---

## 2. 그림에서 가장 중요한 요소

이 그림에는 4가지 핵심 정보가 포함되어 있습니다:

| 요소 | 의미 |
|------|------|
| 빨강 X | X축 |
| 초록 Y | Y축 |
| 파랑 Z | Z축 |
| 보라색 원 | 회전축 (Joint Axis) |

---

## 3. URDF에서 제일 중요한 것 = Joint Origin

URDF의 핵심은 `<origin>` 태그입니다:

```xml
<joint>
    <origin xyz="" rpy=""/>
</joint>
```

이 그림은 바로 그 **origin**을 정의하기 위한 그림입니다.

---

## 4. 좌표계(Frame)의 의미

그림의 숫자 `0, 1, 2, 3, 4, 5, 6`은 각각의 좌표계(Frame)를 의미합니다:

- Frame {0}
- Frame {1}
- Frame {2}
- ...

---

## 5. Base Frame

맨 아래 `[BASE]`는 월드 기준 좌표계입니다.
즉, `world` 또는 `base_link`가 됩니다.

ROS TF Tree에서는 다음과 같습니다:

```
world
 └── base_link
```

---

## 6. J0 = 첫 번째 관절

그림에서 `[Link1] J0 : Base joint`의 의미는 **첫 번째 회전 관절**입니다.

---

## 7. Z축이 중요한 이유

URDF와 DH Parameter에서는 일반적으로 **Z축 = 회전축**입니다.

그림에서 파란 화살표(Z)는 각 관절의 **회전 방향**을 나타냅니다.

즉, URDF에서 다음과 같이 정의됩니다:

```xml
<axis xyz="0 0 1"/>
```

---

## 8. 왜 좌표계가 링크마다 필요한가?

로봇은 링크마다:
- 위치가 다르고
- 방향이 다르고
- 회전축도 다릅니다

따라서 **각 링크마다 독립 좌표계(Frame)**를 가져야 합니다.

---

## 9. Joint Origin의 실제 의미

예를 들어 Frame {1}에서 Frame {2}로 이동하려면:
- 얼마나 이동?
- 어느 방향?
- 얼마나 회전?

이 필요합니다. 이것이 URDF의 `<origin xyz="" rpy=""/>`입니다.

---

## 10. 그림의 치수(mm)의 의미

예시 값: `612.7`, `570.15`, `177.15`, `115.3`

이 값들은 **링크 길이**입니다.
URDF에서는 미터(m) 단위로 변환되어 사용됩니다:

```xml
<origin xyz="0 0 0.6127"/>
```

---

## 11. Upper Arm / Lower Arm

그림의 "Upper arm", "Lower arm"은 각각 URDF에서 다음과 같습니다:

```xml
<link name="upper_arm"/>
<link name="lower_arm"/>
```

---

## 12. Joint Axis 의미

보라색 원 부분은 **회전하는 축**입니다.
URDF에서는 다음과 같이 정의됩니다:

```xml
<axis xyz="0 0 1"/>
```

---

## 13. 실제 URDF 대응 예시

그림의 `J1 : Shoulder joint`는 URDF에서 다음과 같이 표현됩니다:

```xml
<joint name="joint1" type="revolute">

    <parent link="base_link"/>
    <child link="upper_arm"/>

    <origin xyz="0 0 0.197"
            rpy="0 0 0"/>

    <axis xyz="0 0 1"/>

</joint>
```

---

## 14. Parent / Child 개념

URDF는 트리 구조입니다:

```
Base
 └── Shoulder
      └── Elbow
           └── Wrist
```

즉, **부모 링크 기준으로 자식 링크가 정의**됩니다.

---

## 15. 왜 좌표축 방향이 중요한가?

축 방향이 틀리면 모든 것이 틀립니다:

- ❌ FK 틀림
- ❌ IK 틀림
- ❌ Jacobian 틀림
- ❌ MoveIt 오동작
- ❌ Gazebo 폭발
- ❌ Isaac Sim 회전 이상

> **좌표계 방향이 가장 중요합니다.**

---

## 16. DH Parameter와 연결

이 그림은 사실상 **DH Parameter 생성을 위한 그림**입니다.

DH에서 필요한 값:

| 파라미터 | 의미 |
|---------|------|
| θ | 회전 |
| d | Z축 이동 |
| a | X축 이동 |
| α | 축 꼬임 |

모두 이 그림에서 얻을 수 있습니다.

---

## 17. TF Tree와 연결

ROS2에서는 다음과 같은 형태의 TF Tree를 생성합니다:

```
base_link
 └── link1
      └── link2
           └── link3
```

RViz에서 보이는 좌표축이 바로 이 그림을 기반으로 합니다.

---

## 18. IK Solver에서 중요한 이유

IK(역기구학)는 **End Effector 위치 → 각 관절 각도**를 계산합니다.
이때 각 관절의 **위치와 축 방향**을 정확히 알아야 합니다.

> 이 그림이 IK의 기반입니다.

---

## 19. 실제 개발 흐름

실제 로봇 개발은 보통 다음과 같은 순서로 진행됩니다:

```
CAD
 ↓
좌표계 정의       ← ❰ 이 그림 ❱
 ↓
URDF 생성
 ↓
ROS2 TF 생성
 ↓
FK / IK
 ↓
MoveIt
 ↓
Isaac Sim
 ↓
실제 로봇 제어
```

---

## 20. 핵심 요약

> 이 그림은 **"각 관절의 좌표계와 회전축을 정의한 로봇의 수학적 구조도"**입니다.

그리고 이것이 다음 모든 것의 **출발점**입니다:

- ✅ URDF
- ✅ Forward Kinematics
- ✅ Inverse Kinematics
- ✅ Jacobian
- ✅ Dynamics
- ✅ ROS2
- ✅ MoveIt
- ✅ Isaac Sim
