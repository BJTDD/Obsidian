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

### BP_MSS_Bomb
- 심지타들어가는 소리나다가 오버렙되면 터지게 MSS만듬
- Inputs에 변수만들어 타입을 트리거로 만듬 이 트리거는 블루프린트든 다른곳에서 실행시키면 사용가능
- BP에서 Spawn Sound Attached 하여 만든 MSS사용
- 오버랩되면 액터지우고 트리거로 실행되게
- Create Sound 2D 함수 사용
- 
