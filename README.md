# [4DoF Kinematics]
 

### 기구학이란
- 기계나 **기구**의 **움직임**을 분석하고, 이를 **수학적**으로 **해석**하는 학문입니다.

<br>
 
이 페이지는 로봇팔의 **각 관절 각도**를 기반으로 **말단(End-effector)** 위치를 계산하는 방식인 **Forward Kinematics (순기구학)**,
<br>
로봇팔의 **말단(End-effector)** 위치를 기반으로 **각 관절의 각도**를 구하는 방식인 **Inverse Kinematics (역기구학)** 에 대해 기술되어 있습니다.

<br>

본 기술 문서에서는 **말단 장치**를 **수평으로 고정**을 기준으로 **3자유도(3DoF, Degrees of Freedom)** 모델을 우선적으로 분석하였고,
<br>
이후 **말단 장치**의 **링크 길이**를 고려하는 방식으로 해석을 진행하였습니다.

<br><br>

---

### 로봇팔 물성치

- 기구학에서 사용되는 파라미터에 대한 **Raccoon** 로봇팔의 수치는 다음과 같다.

| Raccoon | 기구학 |
| :---: | :---: |
| <img src="" width="600" alt="7_3-1" /> | <img src="https://github.com/user-attachments/assets/17d3bd79-796f-4db8-8b7d-57dabf09db2e" height="300" alt="7_3-2" /> |

* **링크 길이** *(단위: mm)*:
  * $L_1 = 0$  *(xy평면 즉, 원점을 Joint 2로 설정하기 위해 0으로 설정)*
  * $L_2 = 100$
  * $L_3 = 100$
  * $L_4 = 6$  *(말단 (End effector)가 Gripper이기에 Gripper가 물체를 잡는 위치를 고려하여 길이 조정.)*

<br>

* **관절 각도**:
  * $\theta_1 = Joint 1$
  * $\theta_2 = Joint 2$
  * $\theta_3 = Joint 3$
 
 로봇의 말단은 항상 **지면과 수평**으로 설정한다.

<br><br>

 ---
 
 ## 1. Forward Kinematics (순기구학)
 
 - **각 관절의 각도**를 기반으로 **말단(End-effector)** 의 좌표를 구하는 과정이다.
 
| 이미지 | 수식 |
| :---: | --- |
| <img src="https://github.com/user-attachments/assets/17d3bd79-796f-4db8-8b7d-57dabf09db2e" height="400" alt="7_3-2" /> | **Z 좌표**: $$Z = L_1 + L_2 \sin(\theta_2) + L_3 \sin(\theta_2 + \theta_3)$$ <br> **P (XY평면으로 정사영)**: $$P = L_2 \cos(\theta_2) + L_3 \cos(\theta_2+\theta_3) + L_4$$ <br> **X좌표**: $$X = P \cos(\theta_1)$$ <br> **Y좌표**: $$Y = P \sin(\theta_1)$$ |

<br>

- **P**는 **XY** 평면으로 로봇팔을 **정사영** 한 거리이다.
- **​X,Y 좌표**: $𝜃_1$은 **Joint 1**가 회전한 각도이기에, $P$의 거리와 삼각비 관계를 통해 좌표를 얻어낼 수 있다.
- **Z 좌표**: **Joint 2**와 **Joint3**의 각도인 $𝜃_2, 𝜃_3$를 이용하여 계산 가능하다.

<br>

* **Forward Kinematics Solution:**

  $X = (L_2 \cos(\theta_2) + L_3 \cos(\theta_2+\theta_3) + L_4)\cos(\theta_1)$
  <br>
  $Y = (L_2 \cos(\theta_2) + L_3 \cos(\theta_2+\theta_3) + L_4)\sin(\theta_1)$
  <br>
  $Z = L_1 + L_2 \sin(\theta_2) + L_3 \sin(\theta_2+\theta_3)$

<br><br>

 ---
 
## 2.Inverse Kinematics (역기구학)
 
- **말단(End-effector)** 의 **위치 (X,Y,Z)** 를 기반으로 각 관절의 각도를 구하는 과정이다.

| 과정 | 이미지 | 수식 |
| --- | :---: | --- |
| **$\theta_1$ 계산** <br><br> Joint 1이 회전한 각도 <br> 말단의 X와 Y좌표를 알기에<br> $atan2$ 함수를 통해 계산한다. | <img src="https://github.com/user-attachments/assets/17d3bd79-796f-4db8-8b7d-57dabf09db2e" height="300" alt="7_3-2" /> |  $$\theta_1 = \text{atan2}(Y, X)$$ |
| **2차원 좌표 변환** <br><br>Z와 P평면으로 절단한 새로운 좌표계 생성<br> $W$ = 목표점까지의 수평거리  <br> $H$ = 목표점까지의 높이 <br> $H'$ = 3자유도일때의 목표점일때의 높이 <br> $W'$ = 3자유도일때의 목표점일때의 수평거리| <img src="https://github.com/user-attachments/assets/9444fb13-d6f4-4907-894a-c51e8d229e8b" height="200" alt="7_3-3" /> |  $$H = Z, \quad W = \sqrt{X^2 + Y^2}$$ <br> $$H' = H - L_4, \quad W' = W$$ |
| **거리 D 계산 및 $\theta_3$ 계산 (코사인 법칙)**<br><br> $D$는 Link1 말단에서 Link3 말단까지의 거리<br> $D$를 계산 한 후, $\theta_3$를 기준으로 <br>코사인 법칙을 적용한다. | <img src="https://github.com/user-attachments/assets/5522f899-9c55-426e-b2bc-21c3b8858b9d" width="250" alt="7_3-4" /> | $$D = \sqrt{W'^2 + (H' - L_1)^2}$$ <br> $$\cos(\theta_3) = \frac{L_2^2 + L_3^2 - D^2}{2 L_2 L_3}$$ <br> $$\sin(\theta_3) = \pm \sqrt{1 - (\cos(\theta_3))^2}$$ <br> $$\theta_3 = \text{atan2}(\sin(\theta_3), \cos(\theta_3))$$ |
| **$\theta_2$ 계산** <br><br> $\beta$ = $D$와 $W$축이 이루는 각도 <br> $\alpha$ = $L_2$와 $L_3$이 이루는 각도 | <img src="https://github.com/user-attachments/assets/21f32287-28f6-4697-b248-16e32f756627" width="250" alt="7_3-5" /> | $$\beta = \text{atan2}(H'-L_1, W')$$ <br> $$\cos(\alpha) = \frac{D^2 + L_2^2 - L_3^2}{2 L_2 D}$$ <br> $$\sin(\alpha) = \pm \sqrt{1 - (\cos(\alpha))^2}$$ <br> $$\alpha = \text{atan2}(\sin(\alpha), \cos(\alpha))$$ <br> $$\theta_2 = \beta - \alpha$$ |

<br>

* **최종 정리:**

  $\theta_1 = \text{atan2}(Y, X)$
  <br>
  $\theta_2 = \beta - \alpha$
  <br>
  $\theta_3 = \text{atan2}(\sin(\theta_3), \cos(\theta_3))$

<br><br>
