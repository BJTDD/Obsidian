## CPP AI
- 플레이어 발견, 쳐다보기
- task만들기, service만들기
- Games아래 Source펼치면 프로젝트 이름 아래  프로젝트.Build.cs 에서 플러그인 추가 가능
- GameplayTasks 추가 다 닫고 Generate 비주얼스튜디오
### task
- 클래스 추가 BTTask_BlackboardBase 를 부모로 BTTask_Attack 생성
- Attack task 에서 무엇을 할지 결정
- 에디터 Behavior Tree 우클릭하면 내가 추가한 task 추가 가능.
- 노티파이 만들어 몽타주에서 사용
- BTTask attack에서 사운드, 몽타주 실행. 몽타주에 노티파이 달아서 trace함수 실행
### 상태
- 전투, 비전투 - 블랙보드 키 값으로 제어 

### service
- 클래스 생성 부모는 BTService_BlackboardBase 
- 플래이어의 거리를 체크하여 분기 
- 시퀀스나 셀렉터에 데코레이터, 서비스를 달아서 조건 걸어 사용
- Move to 경우 Acceptable Radius값을 조절해 범위조절

### 정리
- 서비스는 블랙보드 키값을 변경시킬수있다. 키값으로 분기하여 여러 시퀀스를 만들어낸다.
- 시퀀스의 말단노드로 태스크를 사용하는데 왼쪽에서 오른쪽 순으로 실행된다.
- 태스크에서 ai의 행동을 실행하고 enum값으로 성공 실패 등을 리턴한다.

### Selector
- 왼쪽부터 맞는거 하나 실행
### Sequence
- 왼쪽부터 맞는거 다 실행
### Simple Parallel
- 왼쪽 태스크 동안만 오른쪽 서브트리 실행

### 데미지
- 데미지 줌 ApplyDamage()
	- 좀비 -> 플레이어
		- 좀비가 플레이어 가까이 와서 공격 몽타지 실행하면 노티파이의 함수실행
	- 플레이어 -> 좀비
		- fire 공격시 trace에서 ApplyDamage호출
- 데미지 받음 TakeDamage()
	- ApplyDamage()의 결과로 엔진에 의해 자동으로 호출
	- 좀비
	- 플레이어
### 이펙트
- Build.cs에 Niagara추가 -> 제너레이트 하면 헤더생김 NiagaraSystem추가 (혹은 파티클시스템)
- PlayHitFX() 만들어 사용

### 디버깅
- nav범위 밖에서 데미지는 잘 박히는데 안에서는 15데미지가 추가로 박힘
- ***테스트한다고 플레이어 블루프린트에 fire 리플렉션 함수에 트레이스를 연결해뒀음...*** 