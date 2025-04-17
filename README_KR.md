# 🌱 언리얼 엔진 5를 활용한 멀티바디 역학 잔디 모션 구현

> *Ghost of Tsushima*에서 영감을 받은 물리 기반 풀 애니메이션 시스템을 언리얼 엔진 5, Niagara, Procedural Content Generation, 다중 강체 동역학을 활용해 구현한 프로젝트입니다.

## 📽️ 데모 영상
[🔗 YouTube에서 보기](https://youtu.be/5h7HZT5iuCI?si=WpGUy6z84sb_mj0Y)

![Grass Motion Sample 01](./resources/sample_01.gif)  
![Grass Motion Sample 02](./resources/sample_02.gif)  
![Grass Motion Sample 03](./resources/sample_03.gif)

## 🔗 소스 코드
[GitHub 저장소](https://github.com/donguklim/Ghost-of-Tsushima-Grass-plus-Rotational-Dynamics) – 알고리즘 상세 설명이 포함된 README 파일 포함

---

# 🎯 프로젝트 목표

- 언리얼 엔진 5로 *Ghost of Tsushima* 스타일의 풀 구현
- 물리 역학 기반의 모션 구현

Sucker Punch Studio가 GDC에서 발표한 [영상](https://youtu.be/Ibe1JBF5i5Y?si=W0vj9MOF8JJ87BsB)을 보게 되었고, 이것을 언리얼 엔진으로 구현하고자 마음먹게 되었습니다.

GDC 발표 영상에서 Sucker Punch Studio는 잔디의 모션은 어떻게 생성되는지 말해주지 않았고, 이에 물리역학 기반의 모션을 넣는 것을 목표에 추가했습니다.

---

# 🌾 고스트 오브 쓰시마의 잔디와 구현 과정

## 고스트 오브 쓰시마의 Bezier 커브 기반 잔디 모델링
![Bezier Curve Grass Example](./resources/bezier_curve_example.jpg)

잔디는 일정 갯수의 포인트로 컨트롤 할 수 있는 베지어 커브를 기반으로 렌더링된 모델을 사용합니다.

잔디의 움직임은 베지어 포인트를 연결하는 라인 시그먼트의 각도를 조절 하므로서 이루어집니다.

위의 그림과 달리 고스트 오브 쓰시마는 3개의 라인 시그먼트로 이루어진 degree 4의 베지어 커브를 사용합니다.

## 마테리얼 셰이더를 사용한 잔디 모델링
- UE5의 마테리얼 셰이더에서 WPO를 조절하여 정사각형의 2D 메쉬로 렌더링 모델을 구현.
- 고스트 오브 쓰시마와 달리 위의 그림과 같은 degree 3의 베지어 커브 기반 잔디를 모델링 함.
- 땅과 맞다은 P0의 조인트의 각도는 인스턴스의 트랜스포메이션을 사용해 조절.
- P1 조인트의 각도는 WPO를 사용해 조절.

고스트 오브 쓰시마와 달리 degree 3의 베지어 커브 기반 잔디를 모델링 하였는데,
이는 관절이 줄어들음으로서 더 적은 리소스를 요구하는 것 외에도 중요한 추가적 이점이 하나 있습니다. 베지어 커브의 길이를 계산할 수 있다는 것입니다.

## 잔디의 길이 조절

베지어 커브는 다음과 같은 특성을 지닙니다.

- 베지어 커브의 길이는 관절의 각도에 따라 달라짐.
- Degree 4 이상의 베지어 커브는 분석 가능 길이 계산식이 존재 하지 않음. 
  - 수치해석적으로만 길이를 구할 수 있음.

고스트 오브 쓰시마에서는 degree 4 베지어 커브를 사용하고 있고, 잔디 길이는 모션에 따라 변합니다. 

GDC 발표에서 개발자는 게임내에서는 잔디 길이 변화가 그리 눈에 띄지 않기에 이 문제는 그냥 넘어갔다고 했습니다.

그러나 바람의 세기가 상대적으로 강하고 변화 폭이 큰 시뮬레이션을 할 경우, 잔디는 더 커다란 폭으로 흔들리는 모션을 일으키게 되며 이렇게 되면 길이 변화는 눈에 띄게 됩니다.

저의 경우 degree 3의 베지어 커브를 사용하였고, 계산된 베지어 커브 길이에 따라 메쉬 크기를 조절하여 잔디 길이가 일정하게 하였습니다.

---

베지어 커브는 유명하니까 누군가 degree 3 베지어 커브 길이 계산식 만들어 둔게 있겠지 라고 생각했는데 구글링 해도 나오지 않더군요.

몇시간 정도 투자해서 스스로 계산식을 만들었습니다.

[계산식 풀이 과정](https://github.com/donguklim/Ghost-of-Tsushima-Grass-plus-Rotational-Dynamics?tab=readme-ov-file#quadratic-bezier-curve-length-control)

---

## 계층구조로 런타임에 잔디 생성

고스트 오브 쓰시마의 잔디는 계층구조를 가지고 있습니다. 

큰 사이즈의 그리드에, 하위 레벨의 더 작은 그리드가 포함된 식으로 카메라와의 거리에 따라 각 그리드의 잔디가 런타임에 생성/소멸 됩니다.

큰 사이즈의 그리드 일 수록 잔디의 밀도가 낮아지고, 생성/소멸이 되는 카메라와의 거리가 더 길어집니다.
반대로 작은 사이즈의 그리드 일 수록 잔디의 밀도가 높아지고, 생성/소멸 거리가 더 짧습니다.

이는 Unreal Engine의 PCG의 hierarchical grid 시스템과 런타임 생성을 사용하여 손쉽게 구현 할 수 있었습니다.

## 보로노이 다이어그램 기반 지역 생성

![Voronoi](./resources/voronoi_example.jpg)  
**보로노이 다이어그램 예시:** 동일한 가장 가까운 포인트를 공유하는 위치들은 같은 영역에 속합니다.

고스트 오브 쓰시마에서는 보로노이 다이어그램을 이용해 지역을 구분하고, 각 지역별로 잔디가 공통된 특징(길이, 모양, 방향 등)을 공유하고 개체별로 노이즈가 있도록 하였다고 합니다.

구체적으로 어떤식으로 생성하였는지는 알 수 없었지만, 저도 또한 보로노이 다이어 그램을 사용해 같은 지역의 잔디가 다음과 같은 특성을 공유하게 하였습니다.
공유되는 특성:
- 풀 길이
- 너비
- 강성
- 방향의 흐름(각 지역마다 다른 랜덤 시드를 써서 Pelin 노이즈로 방향을 생성)
- 색상 노이즈
- 개체 밀도

일부 특성은 Linear interpolation으로 부드럽게 전환 하도록 하였습니다:
- 개체 밀도
- 풀 길이

아래 이미지에서 흰 박스는 보로노이 다이어그램 지역의 중심점에 위치해 있습니다.

![Voronoi Region](./resources/grass_voronoi_regions.jpg)  
**풀 보로노이 다이어그램 구역 예시:** 박스 메시는 보로노이 다이어그램 포인트 위치. 세 개의 영역이 색상, 형상, 밀도, 방향의 흐름이 다름

![Linear Interpolation](./resources/voronoi_region_linear_intp.jpg)  
**Linear Interpolation 예시:** 중심(하얀 박스)에서 경계로 갈수록 풀 길이가 점차 변화

보로노이 중심점은 PCG를 이용해 생성됩니다.
각 잔디 인스턴스가 될 포인트는 중심점 이될 포인트 셋과의 거리를 구하여 자신이 속한 지역의 중심점 포인트의 attribute를 가져 옵니다.
기본 PCG로는 두 포인트 세트 간의 거리를 구하는 기능만이 존재하고, 가장 가까운 포인트의 attribute를 가져오는 기능은 없습니다.

다른 포인트 세트에서 가장 가까운 포인트의 attribute를 가져오는 기능은 PCG Extended Toolkit이라는 써드 파티 플러그인을 사용하여 쓸 수 있습니다.

Linear interpolation을 위해 각 잔디 인스턴스가 되는 포인트는 자신이 속한 지역의 중심점과의 거리 외에도 2번째로 가까운 보로노이 지역의 거리와 attribute 또한 가져옵니다.
이 두 값을 사용해 사용해 linear interpolation을 수행합니다.

## 바람 시뮬레이션

- 바람은 leveled gradient noise 함수를 사용해 생성. 
- 기본적인 바람의 포스를 조절 할 수 있는 UI가 존재하며, 기본 포스에 노이즈로 인한 포스가 추가 됩니다.
- 노이즈가 움직이는 방향은  기본 바람 포스와 같은 방향을 가지게 됩니다.

# ⚙️ 물리 기반 모션 구현

잔디의 모션에 대한 논문을 구글링 하면서, 고스트 오브 쓰시마와 똑같이 베지어 커브 잔디를 사용해 모션을 만드는 ID3 논문을([Grass Swaying with Dynamic Wind Force](https://dl.acm.org/doi/10.1145/2856400.2876008)) 찾게 되었습니다.

처음에는 논문에 나온 내용을 그대로 구현하려고 했으나 논문은 몇 몇 문제를 가지고 있었고, 결국 제 구현 방식은 논문에서 나온것과 전혀 다른 계산식과 알고리즘을 사용하게 되었습니다.

## 기본 다이나믹

레퍼런스 논문에서는 각 조인트는 탄성이 있어서 원래 형태로 돌아가려는 restoration 토크를 일으키고, 바람 포스가 일으키는 토크, 공기 마찰 포스로 인한 토크로 인해 발생하는 각 가속도(angular acceleration)를 매 프레임 마다 구합니다.
그리고 각가속도로 인한 각 속도(angular velocity)와 각 변위(angular displacement)를 업데이트하는 다이나믹스 방식으로 모션을 만듭니다.

### 바람과 공기 마찰로 인한 포스
특정 표면이 공기와 접하고 있을때 포스는 다음과 같이 계산 됩니다.
1. 공기 마찰로 인한 포스 = $-cs\overrightarrow{v}$
   -  c는 공기 마찰 계수입니다. 계수가 높다는 것은 현재 공기의 밀도가 높다는 것입니다.
   - $\overrightarrow{v}$는 p의 속도 입니다.
   - s는 공기와 접하는 면적 입니다.
2. 바람으로 인한 포스 = $s\overrightarrow{W}$
    - 여기서도 s는 마찬가지로 면적입니다.
    - $\overrightarrow{W}$는 바람의 면적당 포스를 가리킵니다.

### Restoration 포스에 대한 기본 전제
![Bezier Curve Grass Example](./resources/bezier_curve_example.jpg)

1. P0은 어느 축으로든 회전 가능한 볼 조인트
2. P1은 잔디의 정해진 로컬 축에서만 회전 가능한 힌지 조인트
3. 각 조인트는 탄성을 가지고 있고, 각 변위에 비래하여 restoration 토크를 일이킨다.
    - Resotration Torque = $-k\overrightarrow{\Delta\theta}$
    - k = 조인트의 강도
    - $\overrightarrow{\Delta\theta}$ = 잔디의 각 변위. 벡터 방향은 회전 축을, 벡터 길이는 회전 각도를 가리킨다.  Left hand rule 사용.


## 레퍼런스 논문의 문제점
### 부정확한 Restoration Torque 계산 방식
논문저자는 Restoration 토크를 종례와는 다른 이상한 방식으로 계산하여, 잔디의 각변위와 restoration torque가 
[선형적이지도 않고 모노토닉하지도 않은 관계가 생기게 하였습니다.](https://github.com/donguklim/Ghost-of-Tsushima-Grass-plus-Rotational-Dynamics/blob/main/README.md#inconsistent-restoration-force-direction)

잔디를 구부러 트리면 특정 각도 까지는 restoration 토크가 증가하다가 더 구부러 트리면 토크가 줄어들어 0이 되고 거기서 더 구브러 트리면 다시 증가하게 되어있었습니다.

논문 저자는 이에 대한 어떤 정당성을 설명하지 않습니다. 이는 논문 저자의 단순 실수로 보입니다.

### 포스를 점에 작용하는 포스로 취급하는 부정확함

바람과 공기 마찰로 인한 포스는 베지어 라인 시그먼트 막대 전체에 일어나게 됩니다. 
그러나 논문의 저자는 각 포스가 막대 전체가 아니라 막대의 끝부분 한점에만 일어나는 것 같은 취급을 합니다.

그 과정에서 공기 마찰로 인해 막대기 끝에 전해지는 포스의 계산식에도 실수가 있었습니다.

올바른 토크를 구하고자 한다면 막대의 라인 시그먼트에 [미분하여 토크를 구하여야 합니다.](https://github.com/donguklim/Ghost-of-Tsushima-Grass-plus-Rotational-Dynamics/blob/main/README.md#wind-and-air-friction-are-not-point-forces)

### 물리적 요소를 무시한 토크 및 가속도 계산

논문의 저자가 사용하는 각 가속도 계산 방식은 관절이 존재하므로서 생기는 반동이나 드래깅을 전혀 고려하지 않고있습니다. 


## 🧩 ABA 알고리즘 도입 과정

도입전에 겪은 일. 
1. Claude AI에게 이런 문제를 뭐라고 하며 어떤 키워드로 검색해서 리서치를 해야할지 물어봄.
2. Claude AI는 **Multi-Bar Linkage System** 이란 키워드를 줌.
3. 해당 키워드로 조사하여 토크, free body diagram 그리는법등, 비롯한 물리학 지식을 좀더 키울 수 있었지만 명확한 해결책을 찾을 수 없었음.
4. 2주동안 free body diagram을 그리고 수학 공식을 적어가며 독자적인 알고리즘을 나름대로 만듬.
5. Claude AI가 업데이트 한듯하여 한번 더 어느 키워드로 조사해야할지 물어봄.
6. Claude AI가 **Multi-body Dynamics**라는 키워드를 줌.

Multi-body Dynamics는 체인이나 링크로 rigid body가 연결되어 있을때의 다이나믹스에 대한 분야였습니다.

![Linear Interpolation](./resources/pepe_flabbergasted.jpg)

제가 필요로 했던 분야였습니다.

Claude에게 왜 저번에는 안가르쳐줬냐고 따지니까 최근 업데이트 했고 Multi-body dynamics를 비롯한 로봇 공학 지식을 추가했다는 답변을 듣게되었습니다.

4,5 일동안 Multi-body dynamics의 기초 개념에 대해 공부하였고, 제가 구현하고자 하는 모션에 가정 효율적인 알고리즘은 Articulated Body Algorithm이란 것을 알게 되었습니다.


## 📌 정밀하지 않은 독자적인 알고리즘 - Payback Method
---
이 섹션은 제가 어떤 독자적인 알고리즘을 생각해 냈고, 그것이 어떻게 작동하는지 정리한 섹션입니다.

어차피 정밀하지도 않은 알고리즘이고 나중에 다른 알고리즘으로 교체했으니 이부분은 굳이 읽지 않으셔도 됩니다.

2주걸려 생각해낸 알고리즘이었지만 헛고생이었습니다.

---

Claude가 준 키워드로 검색하여 Freebody Diagram을 그리는 방식을 배우거나, Torque에 대해 더 깊이 배우기는 했지만, 결국 이것 또한 명확한 방법을 알 수 없었습니다.
**Multi-Bar Linkage**로 검색하면 나오는 것은 모든 끝부분에 있는 조인트가 모두 고정된 경우에 대해서만 Torque를 구하는 경우만을 다뤘으며 베지어 커브 잔디처럼 한쪽 끝이 고정되어 있지않고 열려있는 경우에 대해서는 다루지 않았습니다.

그 결과 독자적으로 [Payback](https://github.com/donguklim/Ghost-of-Tsushima-Grass-plus-Rotational-Dynamics/blob/main/README.md#payback-moment-algorithm) 이라는 알고리즘을 사용하였습니다. 알고리즘은 다음과 같은 아이디어로 만들어 졌습니다.

1. P1 관절이 있는 경우의 토크, 각 가속도를 어떻게 구하는지 잘 모르겠다.
2. 하지만 만약 지금 restoration torque랑 공기마찰, 바람으로 인한 torque가 서로 상쇄해서 net 토크가 0이되어 P1 관절이 움직이고 있지 않다면, 각 가속도를 구할 수 있다.
3. P1에 토크를 빌려줘서 net 토크가 0이 되게 만들어서 가속도를 구하고 나중에 다시 돌려 받아서 bar2를 회전시키자. 단 P1이 움직이면서 생긴 inertia force로 인한 토크 또한 고려해서 다시 돌려 받아야한다.
4. p1이 고정되어 있지 않았을때 bar2의 각 가속도를 구하는 방식을 모르겠다.
5. 3번에서 했듯이 이번에는 p0에 토크를 빌려줘서 p1이 고정되어 있도록 하게 하고 p1의 각 가속도를 구하자. 단 bar2가 움직임으로서 생기는 반동 또한 고려해서 P0에서 토크를 돌려받아야한다.
6. P0에 빌려줬던 토크를 또 돌려받아야 한다. 3번으로 돌아가자.

토크를 빌려줘서 각 관절을 고정시키고 나중에 빌려준 토크를 받는 방식으로 대략적인 각 가속도를 구하자는 아이디어였습니다.
빌려주고 다시 받는 과정이 무한히 반복될 수도 있기에 이것을 일정 횟수만 반복하고 멈추도록 하였습니다.

나중에는 현재 관절의 net 토크가 0이 되게 만드는게 아니라 bar가 움직임으로서 생기는 반동까지 고려해서 토크를 빌려주는 것으로 linear system을 푸는 방식으로 계산하여 무한대로 반복하는 것과 비슷한 효과를 낼 수 있지 않을까 생각하였습니다.

Linear system을 푸는 방식으로 접근하는 것, 토크릴 빌려주고 돌려주는 횟수를 제한하는 것 둘다 어느 정도 그럴듯한 움직임을 보여주기는 했습니다.

## 🦾 Articulated Body Algorithm

Multi-body Dynamics 분야에서 알려진 가장 효율적인 시뮬레이션 알고리즘으로 각 관절에 전달되는 포스를 고려하여 각 가속도를 정밀하게 시뮬레이션 할 수 있는 알고리즘 입니다.

알고리즘의 자세한 내용은 Roy Featherstone 교수님의 책 [Rigid Body Dynamics Algorithms](https://link.springer.com/book/10.1007/978-1-4899-7560-7)에 나와 있습니다.

현재 이 프로젝트가 디폴트로 사용하는 알고리즘이며, 제가 독자적으로 만든 알고리즘은 일단 옵션으로 선택할 수 있게 해두긴 했는데 그냥 버려도 될것 같습니다.

## 🛑 최대 각 변위 제한

현실의 잔디는 무한하게 회전하거나 꼬을 수 없습니다. 어느 축으로 회전하든 간에 최대 각도가 특정 threshold 이하가 되도록 하였고, 최대 각변위에 도달하면 각 속도는 0이 되게 하였습니다.

[각변위 제한 알고리즘 링크](https://github.com/donguklim/Ghost-of-Tsushima-Grass-plus-Rotational-Dynamics/blob/main/README.md#angular-displacement-magnitude-limitation-on-p0)

## 🚧 지면 충돌 처리

지면 충돌은 지면의 노멀 벡터와 Bar1 방향의 dot product가 일정 threshold 값 이하가 되도록 하였고 threshold에 도달하면 속도가 0이 되게 하였습니다.

[지면 충돌 알고리즘 링크](https://github.com/donguklim/Ghost-of-Tsushima-Grass-plus-Rotational-Dynamics/blob/main/README.md#ground-collision)

---

# 🛠️ 시스템 통합

## 다이나믹 모션과 고스트 오브 쓰시마의 잔디의 조합

다이나믹스 모션을 사용했기에 각 잔디는 각 속도와 각 변위를 저장하여 가지고 있었고, 이를 손쉽게 구현하기 위해서 나이아가라를 사용할 필요가 있습니다.

나이아가라 에미터는 나이아가라 데이터 채널(NDC)을 통해 생성되며 PCG가 잔디의 포인트 데이터를 NDC에 write하게 됩니다.

## PCG + NDC 데이터 채널 활용
PCG에서 NDC에 데이터를 쓰게 해주는 것은 최근 에픽게임즈에서 추가한 기능이며 아직 실험 스테이지인 플러그인을 추가하므로서 쓸 수 있습니다.
이것은 새로 추가된 기능이라서 이에 대한 튜토리얼이나 가이드를 만든 사람이 아무도 없었고, 제가 스스로 트라이얼&에러를 겪으며 사용방식을 나름 정리하여 다른 사람들에게 도움이 될 수 있도록 튜토리얼 비디오를 만들었습니다.

[🔗 PCG + Niagara 데이터 채널 튜토리얼](https://youtu.be/C1LmzQKNnzI)

### PCG와 Niagara 파티클 클린업

PCG에서 Static Mesh Spawner와 사용할대와 달리, PCG는 직접적으로 나이아가라의 파티클을 지울 수 없습니다.
PCG는 그저 NDC에 데이터를 쓰는것만이 가능합니다.

PCG는 카메라 위치에 따라 파티클 데이터를 다시 NDC에 보내게 되며 나이아가라 에미터에서 필요없는 파티클을 클린업 해주지 않으면 중복된 잔디가 생성됩니다.
그렇기에 에미터가 필요없는 파티클이나 에미터 인스턴스 자신을 없애도록 해야합니다.

현재 구현에서는 에미터가 다음과 같은 방식으로 파티클을 없애고 있습니다.
1. 파티클이 속한 PCG 그리드와 카메라 간 거리 기준으로 제거
2. 파티클 생성 시간과 마지막으로 PCG가 데이터를 보낸 시점 비교해서 오래된 파티클 제거.

자세한 방식은 [튜토리얼 비디오](https://youtu.be/C1LmzQKNnzI)에서 확인 하실 수 있습니다.

---
# 📈 구현 요약 및 퍼포먼스

## 프로젝트 요약

- *Ghost of Tsushima* GDC 발표자료 기반
- 계층적 PCG + Niagara + 물리를 활용한 실시간 풀 생성 및 모션 시스템
- **Articulated Body Algorithm (ABA)** 를 적용하여 물리적 사실감 강화
- 독자적인 개선 사항 구현:
    - **2차 베지어 곡선**을 이용한 고정 길이 풀
    - ABA 기반 물리 동역학 모션
    - 관절 최대 각도 변위에 대한 물리적 제약
    - 각변위 기반 풀 잎 회전 추가

---

## ⚔️ *Ghost of Tsushima*와 비교

| 기능                            | Ghost of Tsushima        | 본 프로젝트 구현                      |
|---------------------------------|---------------------------|--------------------------------|
| 베지어 곡선 타입                | 3차 (4 포인트)           | 2차 (3 포인트)                     |
| 풀 길이                         | 제어 안됨 (가변적)        | 고정 길이                          |
| 움직임                          | 불명                      | ABA 기반 다중물체 역학 포워드 시뮬레이션       |
| 런타임 생성 방식                | 커스텀 엔진               | UE5 PCG + Niagara 데이터 채널 인터페이스 |

---

## 주요 기능

### 풀 모델링 및 최적화
- 런타임 스폰/클린업을 위한 **계층적 PCG 그리드**
- **2차 베지어 곡선**을 사용한 풀 블레이드 모델링
    - *Ghost of Tsushima*의 3차 베지어와 달리 길이 고정 유지

![Bezier Curve Grass Example](./resources/bezier_curve_example.jpg)  
**베지어 곡선 예시:** P0, P1 지점이 회전 관절로 작동합니다.

### 바람 및 움직임 시뮬레이션
- 노이즈 함수를 이용한 바람 시뮬레이션
- 스켈레톤 기반 관절 구조:
    - P₀: 볼 조인트 (자유도 3)
    - P₁: 힌지 조인트 (자유도 1)
- 초기 자세로 복원되는 탄성 복원력 (강도는 무작위화)
- 작용하는 힘:
    - 바람 힘 (UI로 조정 가능)
    - 공기 저항 (UI로 조정 가능, 감쇠 역할)
    - 관절에서의 복원력

### 물리 시스템
- [I3D 논문](https://dl.acm.org/doi/10.1145/2856400.2876008)을 참고하되 개선 적용
- 계산 오류 제거
- 원 논문의 물리적으로 부정확한 가속도 계산방식, 모순적인 알고리즘을 다음과 같이 대체:
    - **ABA**를 사용한 가속도 계산
    - 각도 변위 최대 값 제한 제약
    - 지면 충돌 처리
- 고정된 잔디 길이

---

## 움직임 비교

### 강한 바람에서의 움직임

**최대 각도 변위 제한 및 충돌 처리 전**

![No Limit](./resources/no_limit.gif)  
[🔗 YouTube에서 보기](https://youtu.be/sHjHLRHukEs)  
[🔗 바람 강도 40부터 보기](https://youtu.be/sHjHLRHukEs?si=raVWfqdE0HeZyLcM&t=68)

**최대 각도 변위 제한 및 충돌 처리 후**

![After Limit](./resources/after_limit.gif)  
[🔗 데모 영상 (바람 강도 40부터)](https://youtu.be/5h7HZT5iuCI?si=dYmNk5WoUefEqJj9&t=36)

제한이 없을 경우:
1. 제한 받지 않고 끊임없이 회전하고 꼬여질 수 있는 풀은 무한한 복원력을 저장할 수 있음
    - (실제 였다면 풀은 그렇게 되기 전에 이미 부러졌을 것)
2. 그로인해 바람 움직임과 일치하지 않는 부자연스러운 모션 발생

### 참고 논문 결과 비교

참고 논문에서 다음과 같은 사항을 수정하였을때의 결과
- 계산 오류 수정
- 모순적인 알고리즘 제거 및 다음으로 대체:
    - 각도 제한
    - 지면 충돌
- 풀 길이 고정 적용

[🔗 YouTube에서 보기](https://youtu.be/qu_WTiCiIrc)  
[![영상 썸네일](https://img.youtube.com/vi/qu_WTiCiIrc/hqdefault.jpg)]

강한 바람 속에서는 풀이 직선 형태로 정렬되는 경향을 보입니다.

---

## 퍼포먼스 평가

[GitHub 샘플 프로젝트](https://github.com/donguklim/Ghost-of-Tsushima-Grass-plus-Rotational-Dynamics)를 사용해,
고정된 PCG **클린업 멀티플라이어 값 (1.1)**과 다른 각 PCG 그리드 사이즈별 **샘플링 레이트**에 따른 FPS를 측정하였습니다.

각 그리드 사이즈는 아래의 고정된 PCG 생성 radius를 사용합니다.:

| PCG 그리드 사이즈 | 생성 Radius |
|-------------|-----------|
| 400         | 1600      |
| 800         | 3200      |
| 1600        | 6000      |
| 3200        | 8000      |

> *(“n SR” = n 사이즈 그리드의 샘플링 레이트)*  
> FPS는 플레이 시작 몇 초 후에 측정되었습니다.

---

### 📊 RTX 2080 TI를 사용한 FPS 결과 (각 그리드 사이즈별 1제곱 미터당 샘플링 레이트에 따라)

| **FPS 범위** | **400 SR** | **800 SR** | **1600 SR** | **3200 SR** |
|------------|------------|------------|-------------|-------------|
| 60 – 70    | 32         | 16         | 8           | 4           |
| 65 – 75    | 16         | 8          | 4           | 2           |
| 68 – 78    | 8          | 4          | 2           | 1           |
| 75 – 80    | 2          | 1          | 1           | 1           |
---


## 🔮 향후 발전해야할 부분
- Render Target 또는 추가 NDC 사용하여 플레이어 및 오브젝트와의 상호작용 구현
- 풀 파편 바람 효과 추가
- 더 다양한 형태의 풀 모델 추가
- Nanite 활용하여 최적화 개선
- 그림자 품질 개선
- NDC 내부에서의 포인트 생성 또는 PCG로부터의 압축된 포인트 데이터 전송
    - 현재 모든 풀 생성 포인트는 PCG에서 NDC로 전송됨.
    - 통신 오버헤드를 줄이기 위해, 포인트 데이터를 압축해서 보내거나 Niagara 이미터가 직접 포인트 생성을 결정하도록 하는 방식을 고려할 수 있음.
    - 다만, 이러한 방식은 PCG 내에서 풀의 위치나 attribute에 대한 제어가 제한될 수 있음.

---

# 🛠️ 사용 플러그인
- Niagara
- PCG
- PCGNiagaraInterop (experimental)
    - PCG에서 NDC로 데이터 쓰기 가능
- PCGExtendedToolkit
    - 보로노이 다이어그램 구역 생성에 사용

---

# 📚 참고 자료

- [Rigid Body Dynamics Algorithms - Roy Featherstone's book](https://link.springer.com/book/10.1007/978-1-4899-7560-7)
- [GDC Presentation – Procedural Grass in *Ghost of Tsushima*](https://youtu.be/Ibe1JBF5i5Y?si=EbGqmGS29uNdBPUn)
- [I3D Paper – Grass Swaying with Dynamic Wind Force](https://link.springer.com/article/10.1007/s00371-016-1263-7)
- [Unreal Engine Documentation – Niagara Data Channels Intro](https://dev.epicgames.com/community/learning/tutorials/RJbm/unreal-engine-niagara-data-channels-intro)

---

# 📁 다른 포트폴리오

- [Implementation Photon Mapping & Disney's Photon Beam PBR with DirectX 12](https://github.com/donguklim/DirectX12PhotonBeam)
- [Implementation Photon Mapping & Disney's Photon Beam PBR with Vulkan](https://github.com/donguklim/vk_raytracing_tutorial_KHR/tree/master/photon_beam)
