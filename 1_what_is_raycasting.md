[목록으로](https://github.com/365kim/raycasting_tutorial)

## :crystal_ball: 레이캐스팅이 뭐야?

- __레이캐스팅__ 은 2차원 맵에서 3차원의 원근감을 만드는 __렌더링 기술__ 입니다. 

    - __레이캐스팅__ 은 스크린의 모든 수직선에 대해 계산(calculation)만 하면 되어서 속도가 빠릅니다.
    - 컴퓨터가 지금보다 느려서 3D 엔진을 실시간으로 실행할 수 없던 과거에는 __레이캐스팅__ 이 최초의 해결책이었습니다. 
<br>

- __레이캐스팅__  기술을 사용한 게임 중 가장 유명한 게임은 'Wolfenstein 3D'입니다.

    - Wolfenstein 3D의 레이캐스팅 엔진은 극히 제한적이어서 286컴퓨터에서도 작동했답니다. 
    - 하단 우측 스크린샷은 Wolf3D의 맵에디터 스크린샷인데, 모든 벽이 2D 직사각형으로 되어있고 높이마저도 다 똑같은 것을 볼 수 있습니다. 이 엔진으로는 계단, 점프 또는 높이차이와 같은 것은 구현할 수 없었습니다.
    - 나중에 출시된 Doom, Duke Nukem 3D 같은 게임에서는 고급 레이캐스팅 엔진으로 기울어진 벽, 다른 높이, 질감있는 바닥/천장, 투명벽 등을 가능하게 했습니다.
    - __스프라이트(sprite)__ 는 enemies, objects, goodies와 같은 2D 이미지를 말합니다. 이번 튜토리얼에서는 스프라이트에 대해 설명하지 않습니다.    <br>
    ![1a](https://user-images.githubusercontent.com/60066472/83317174-e96f3f00-a265-11ea-8433-cc7afeffb42b.jpg)
    ![2](https://user-images.githubusercontent.com/60066472/83316370-6bf50000-a260-11ea-9dc7-1e537d792a81.jpg)
<br>

- __레이캐스팅__ 은 __레이트레이싱__ 과 다릅니다!

    - 레이캐스팅(Raycasting)은 4MHz 그래픽 계산기에서도 실시간으로 작동하는 빠른 semi-3D 기술인 반면에, 레이트레이싱 (RayTracing)은 실제 3D장면의 반사, 그림자를 지원해서 현실감있는 렌더링 기술입니다. 
    - 나중에서야 컴퓨터가 빨라져서 꽤 높은 해상도와 복잡한 장면들을 실시간으로 처리할 수 있게되었습니다.
<br>

- 아래의 cpp파일은 레이캐스터(untextured & textured)의 코드 전체입니다. (다운로드해서 보실 수 있습니다)

    - [raycaster_flat.cpp](https://lodev.org/cgtutor/files/raycaster_flat.cpp)
    - [raycaster_textured.cpp](https://lodev.org/cgtutor/files/raycaster_textured.cpp)
    - 2016년 2월 업데이트: 코드단순화를 지적해준 Thomas van der Berg에게 감사를 표합니다<br>(perpWallDist 단순화, wallX에 재사용 가능)
<br>
<br>

[원문링크](https://lodev.org/cgtutor/raycasting.html#Introduction)
<br>

[__다음편으로 : 아주 기본적인 원리__](https://github.com/365kim/raycasting_tutorial/blob/master/2_basics.md)
