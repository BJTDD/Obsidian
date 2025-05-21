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

## Trigger2
- 주로 사용되는 함수들 정리

### BP_MSS_Wind
- 스테레오 바람사운드 만들기. 케릭터의 속도에따라 바람 사운드 조절
- Input값을 MSS에서 만들어 BP에서 사용
- 노이즈를 만들어 하이패스필터, LFO, Mixer를 조합 

### MSS_CarEngine
- 시동거는 소리와 엔진 돌아가는 소리가 Wav에 같이들어있는데 이를 분리
- 차 속도에따라 엔진소리 피치값 조절