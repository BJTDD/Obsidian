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

## Fracture
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
- 엔진 - 콘텐츠 - EditorResources - FieldNodes 의  FS_AnchorField_Generic, FS_MasterField 사용
- FS_MasterField: 충격을 줌. Directional Magnitude 높이면 힘이 강해짐 Use Directional Vector 체크하면 해당 방향으로 힘을 줌
- FS_AnchorField_Generic: 이것과 곂친 부분은 충격 안 받음
- 큐브로 벽을만들어 Fracture적용 
- 사용하는벽 GC_Brick_2 의 Initialization Fields에  FS_AnchorField_Generic 이것들 다 추가해주기 
- 그대로 사용하면 무거우니 브릭을 카오스캐시매니저 생성(CCC_Brick_2)후 카오스캐시매니저위치를 브릭으로 이동
- 카오스캐시매니저 캐시모드를 RECORD모드로해서 시뮬레이션 실행 파편들이 다멈추면 종료
- 캐시모드를 Play로 하고 Start Time을 움직여서 잘되는지 확인
- CCC_Brick_2을 레벨에 배치해 단독으로 사용가능

### Level Sequence
- CCC를 레벨 시퀀스에 넣어 Start Time 을 조절해 시퀀스를 만들 수 있음

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

### BP_PCG_Building
- 블루프린트에 콜리전 박스 추가 사용할 메시보다 크기 크게 Box Extent 조절
- 이번엔 박스콜리전을 사용했으니 PCG에서 Volume Sampler사용 Voxel Size는 박스콜리전 사이즈로
- 피봇이 중앙정렬이 아니라면 Z로테이션을 180도 돌려 사용
- 스태틱매시 피봇 바꾸려면 해당 스태틱매시 눌러서 모델링모드로 가서 XForm눌러 센터, 바텀 눌러 지정

## Landscape
- 대규모 맵은 Landscape에서 PCG로 작업

### Lv_PCG_Landscape
- 랜드스케이프모드 - 페인트 - 타겟레이어에서 각 레이어마다 우클릭 Fill레이어
- 해당 레이어를 눌러 페인트 칠 할 수 있음

### PCG_Volume
- PCGVolume액터 레벨에 배치하여 PCGComponent - Settings 안쪽에 PCG Graph 넣을수있음
- 볼륨에 곂친공간에 원하는 메시를 넣을거임
- Surface Sampler사용 - 간격과 양 조절
- Attribute Filter Use Constant Threshold 체크. 타겟레이어에 사용할 레이어이름 넣고 툴강도의 오퍼레이터와 비교해서 생성

### WaterBodyLake
- Water 플러그인 다운
- WaterBodyLake액터 레벨에 배치
- 물에 곂친 메시들 지우기
- 옵션에서  Terrain, Wave 설정 가능

### WaterBodyRiver
- 강 만들어 강 주위와 안쪽에 돌 배치

### BP_PCG_Spawner
- 플러그인 Procedural Content Generation Frameworkd PCG Geometry Script Interop 다운
- 메시위에 메시생성 할거임. 블루프린트의 변수를 가져와 PCG에서 사용 가능
- Attribute Noise: Density 값을 해당 범위 랜덤으로 모드 Set
- Density Filter: Density 해당값 사이만 패스
- Normal To Density: Z축이 1. 해당 메시가 위로 보고있는것만 사용할거임 다시 필터사용
- 블루프린트 Construction Script에서 나의 Static Mesh 컴포넌트를 My Mesh변수로 Set 시킬꺼임 
- My Mesh는 PCG Graph에서 변경시키고있음. 마찬가지로 다른 변수들도 가져와서 사용

### PCG_Grid
- Get Actor Data 모드를 Get Single Point로 해서 하나만 가져오게 
- Bounds Modifier는 보기편하게하려고
- Copy Points 가지고있는 그리드를 복사해서 타겟에 넘겨줌 
- 그리드를 활용해 PCG Graph 만듬

## Lv_Dungeon

### PCG_Path
- 런타임에 계속 바뀜
- 0번 인덱스에서 마지막 인덱스로 


### 모션 디자인 
- 게임 인트로, ui