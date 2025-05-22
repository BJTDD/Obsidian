## Meta Sound

### Wave Player
- Audio - MetaSound Source
- 위의 MetaSound눌러 아웃풋 포멧 스테레오 등 변경가능 사용할 Wave Player에 맞춰 사용
- 두가지 플레이어 이상 동시에 사용위해 Mixer 사용
- 변수만들어 피치 값을 조절

### Interfaces
- Listener:케릭터
- Spatialization:공간
- OneShot: 주로사용 할거임

### Random
- Wave Asset을 변수로만들어 배열로만들고 랜덤으로 재생되게 Random Get사용

### BP EventGraph
- Spawn Sound Attached, Create Sound 2D 두 함수 사용 

### BP_MSS_Bomb
- 심지타들어가는 소리나다가 오버렙되면 터지게 MSS만듬
- Inputs에 Input_Explosion 변수만들어 타입은 Trigger. 이 트리거는 블루프린트든 다른곳에서 실행시키면 사용가능
- BP에서 Spawn Sound Attached 하여 만든 MSS사용
- 오버랩되면 내 액터를 지움. 그럼 Destroyed 이벤트가 실행되어 Execute Trigger Parameter 함수 실행 

### Trigger1
- Trigger Repeat사용 Period는 BPM To Seconds를 연결해 사용
- Trigger Sequence를 사용해 순차적으로 실행되게 사용

### Trigger2
- 주로 사용되는 함수들 정리

### BP_MSS_Wind
- 스테레오 바람사운드 만들기. 케릭터의 속도에따라 바람 사운드 조절
- Input값을 MSS에서 만들어 BP에서 사용
- 노이즈를 만들어 하이패스필터, LFO, Mixer를 조합 

### MSS_CarEngine
- 시동거는 소리와 엔진 돌아가는 소리가 Wav에 같이들어있는데 이를 분리
- 차 속도에따라 엔진소리 피치값 조절


## Physics

### Physics Constraint
- Physics Constraint 컴포넌트를 만들어 사용할 컴포넌트 두가지 이름을 넣어줌. 두 컴포넌트를 연결할수있음
- 첫번째 컴포넌트를 Root로 하고 움직인는 컴포넌트를 달아주면됨
- Angular Limits가 Free 라서 회전이됨,  원하는 축을 Locked해서 해당 방향을 잠굴수있음

### Physics Constraint - Linear Limits
- 가동 범위 지정

### Physics Constraint - Linear Motor
- 원래 위치로 돌아아게 
- Position Target: 해당 위치로 이동하려는 힘을 줌(탄성)
- Velocity Target: 해당 위치로 해당 속도로 이동 되게

### Physics Constraint - Angular Limit
- Swing1: Z, Swing2: Y, Twist: X 축   회전값 범위 지정

### Physics Constraint - Angular Motor
- Angular Drive Mode: Twist and Swing

### Cable
- 케이블 연결 Attach End To에 연결할 컴포넌트 이름 지정 

### Breakable
- 특정 물리 값이 들어가면 끊어지게
- Linear Limits - Advanced - Linear Breakable 체크, Simulate Physics - Mass 조절

### BP_SpringPad


### BP_TwoDoor
- 서부영화 양쪽문 느낌
- 실린더(기둥)가 루트 큐브(문)가 자식컴포넌트
- PhysicsConstraint Disable Collision체크
- Linear Limits는 모두 Locked 
- Angular Limits는 Z축만 Limited(Swing1) 80
- Angular Motor은 Twist and Swing 체크하고  Orientation, Velocity의 Strength값 조절 여기선 300, 10

### Slerp
- Angular Limits가 하나라도 락 걸려있으면 활성화가 안됨 Free로하면 사용가능
- Target Velocity x값만 200 그냥움직임

### Twist And Swing
- 힘을 줘야 움직임

### BP_RevolvingDoor
- 회전문 만들어서 선택후 Modeling Mode의 Mesh-Union으로 합쳐주기 이런식으로 합치면 기즈모가 중앙에 오게됨
- 만들어진 Static Mesh를 BP에 배치하고 물리적용. 무게와 Angular Damping 값 조절
- PhysicsConstraint 의 Root를 실린더로해서 Z축만 사용

### BP_Seasaw

### BP_Thuster
- PhysicsThruster 컴포넌트 이용 스태틱매시의 자식노드로 사용
- 기즈모 x축이 땅방향으로하면 그쪽으로 힘을줌, Thrust Strength 150만, Auto Activate 체크

### BP_SpringBoard
- 다이빙

### BP_Propeller
- 블루프린트 이벤트 그래프로 옵션 설정
- 자동으로 돌아가게 하고 싶은데 Twist and Swing이 한번 건드려줘야 움직여서 뭔가 잘 안됨

### RadialForceActor
- 해당 타입의 주변에 물리적 힘을줌
- 레벨에 RadialForceActor 배치하고 주변에 큐브를 쌓아둠. Force Strength은 100만, Auto Activate 끄기
- 레벨 블루프린트로 트리거박스에 내가 오버랩되면 Activate 체크 되게 만듬

### Balloon
- PhysicsThruster  힘은 중력가속도 만큼, 풍선 무게는 0.01
- 블루프린트 사용하면 Auto Activate 를 체크 해제해서 사용
- 여기선 벽 밀고 가야해서 Thrust Strength 100줌

### Physics Asset 
- 물리적효과 어떻게줄지

### Physics Material
- Collision의 Phys Material Override에 넣어줌
- Restitution(복원력)값 수정 - 탄성계수 
- Friction 마찰력

##  Fracture
- 건물 파편 적용 (좀 무거움) 
- 적용할 메시누르고 Fracture Mode에서 New로 만들어 원하는 경로에 Geometry Collection을 만듬
- Damage 임계값을 수정하여 힘 받는 양을 조절

### Uniform
- 랜덤 갯수로 쪼개짐

### Brick
- 벽돌 모양대로 쪼개짐

### Radial
- 중심점에서 쪼개짐
- 노이즈 주면 경계부분에서 자연스럽게

### ChaosCacheCollection
- 좀 가볍게 사용하는 방법이 있는듯

## PCG
- 런타임때 맵 수정 가능
- 플러그인 Procedural Content Generation Framework PCG 다운

### PCG_Floor 
- 블루프린트만들어 PCG컴포넌트 추가하고  Instance - Graph 에 PCG Graph 넣어줌
- 스플라인만들어 우클릭 Spline Generate Panel 로 사각형 만듬 Closed Loop 체크
- Spline Sample에서 On Interior로 변경(포인트를 만들어줌), Unbounded체크, 디버그 모드 체크
- Transform Points로 크기 지정가능
- Static Mesh Spawner - Mesh Entries 추가해서 Descripor에서 메시 지정 가능
- 콜리전설정해서 통과 안되게
- 누르고 메뉴창 Actor에서 Static Mesh로 바꿀수있음.

### PCG_Wall
- 블루프린트에 PCG, Spline 컴포넌트 추가, PCG에 PCG_Wall 넣어줌
- 벽을 3층 쌓고 그 위에 나무, 버섯, 통나무, 덤불 올려줌
- Simple Sampler의 모드를 Distance로 하여 Distance Increment값을 조절하여 거리를 지정
- Transform Points로 크기와 위치 오프셋 지정가능  
- Offset Min,Max로 위치(X축:거리, Z축:높이)지정 가능 다르게하면 랜덤으로 사이값
- Rotation Min,Max   Scale Min,Max 도 마찬가지

### BP_PCG_Gate
- 곂친곳 다른 메시로 바꿔줌
- 콜리전 박스 추가해서 루트로 만들고 Tags에 Gate이름 추가
- PCG_Wall의 1층벽과 곂친곳을 다른 벽으로 교체함
- 기존벽을 스폰하고 해당 Tag와 곂친곳을 뺀다 만약 곂친부분(Intersection)이 있으면 덤불을 생성한다.
- 스플라인과 해당 Tag 콜리전 박스가 곂처져있어야함(충돌)

### PCG_Biome
- 블루프린트 Spline 만들어 Closed Loop 체크.
- 숲 만들기

### BP_PCG_Road
- 숲에 길 만들기
- 스플라인 데이터 가져와서 Must Overlap Self 체크. 액터는 만들어둔 블루프린트에 스플라인만 있음
- 