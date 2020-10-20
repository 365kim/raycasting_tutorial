[목록으로](https://github.com/365kim/raycasting_tutorial)

## :crystal_ball: 예제코드로 이해하는 레이캐스터 구현 (untextured)
- 예제코드 전체보기 : [raycaster_flat.cpp](https://lodev.org/cgtutor/files/raycaster_flat.cpp)

- 레이캐스터의 기본이 되는 __텍스쳐 없이 색상만 표현한 레이캐스터__ (Untextured Raycaster) 부터 시작하겠습니다.

    - __FPS 카운터__ (fps : frames per second, 초당 프레임)도 다룹니다.
    - __이동/회전을 위한 충돌감지__ 기능이 있는 입력키 (input key)도 다룹니다.
<br>


```cpp
#define mapWidth 24
#define mapHeight 24
#define screenWidth 640
#define screenHeight 480

int worldMap[mapWidth][mapHeight]=
{
  {1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1},
  {1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1},
  {1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1},
  {1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1},
  {1,0,0,0,0,0,2,2,2,2,2,0,0,0,0,3,0,3,0,3,0,0,0,1},
  {1,0,0,0,0,0,2,0,0,0,2,0,0,0,0,0,0,0,0,0,0,0,0,1},
  {1,0,0,0,0,0,2,0,0,0,2,0,0,0,0,3,0,0,0,3,0,0,0,1},
  {1,0,0,0,0,0,2,0,0,0,2,0,0,0,0,0,0,0,0,0,0,0,0,1},
  {1,0,0,0,0,0,2,2,0,2,2,0,0,0,0,3,0,3,0,3,0,0,0,1},
  {1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1},
  {1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1},
  {1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1},
  {1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1},
  {1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1},
  {1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1},
  {1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1},
  {1,4,4,4,4,4,4,4,4,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1},
  {1,4,0,4,0,0,0,0,4,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1},
  {1,4,0,0,0,0,5,0,4,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1},
  {1,4,0,4,0,0,0,0,4,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1},
  {1,4,0,4,4,4,4,4,4,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1},
  {1,4,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1},
  {1,4,4,4,4,4,4,4,4,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1},
  {1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1}
};
```
- 지도는 __2차원 배열__ 로 나타낼 수 있고, 배열의 각 요소는 지도의 한 칸을 나타냅니다.

    - 위에 선언된 지도는 24칸 x 24칸 크기로 굉장히 작은 편이고, 코드 안에서 바로 정의되고 있습니다.
        - Wolfenstein 3D와 같은 실제 게임에서는 더 큰지도를 사용하고 코드 안에서 바로 정의하지 않고 파일에서 지도를 불러옵니다.
    - 만약 요소의 값이 0이라면, 그 칸은 비어있어서 플레이어가 지나갈 수 있는 칸이라는 뜻입니다. 만약 요소의 값이 0보다 크다면, 그 칸은 특정 색상이나 텍스쳐가 있는 벽이라는 뜻입니다.
        - __배열 요소값 '0'__ : 비어있는 큰 공간
        - __배열 요소값 '1'__ : 벽
        - __배열 요소값 '2'__ : 내부의 작은 방
        - __배열 요소값 '3'__ : 몇 개의 기둥
        - __배열 요소값 '4'__ : 복도
    - 이 코드는 아직 어떤 함수에도 포함되어 있지 않습니다. 메인함수 시작되기 전에 넣어주세요.
<br>

```cpp
int main(int argc, char *argv[])
{
  double posX = 22, posY = 12;  //x and y start position
  double dirX = -1, dirY = 0; //initial direction vector
  double planeX = 0, planeY = 0.66; //the 2d raycaster version of camera plane

  double time = 0; //time of current frame
  double oldTime = 0; //time of previous frame
```
- 우선 메인함수의 __변수를 선언__ 합니다.

    - __posX, posY__ 는 플레이어의 초기 위치벡터, __dirX, dirY__ 는 플레이어의 초기 방향벡터 입니다.
    - __planeX, planeY__ 는 플레이어의 카메라평면 입니다.
    - 방향벡터와 수직이기만 하면 변수선언 시 카메라평면의 길이는 다양할 수 있습니다.
    - 이 때의 "카메라평면의 길이 : 방향벡터 길이" 의 비율로  __FOV__ 가 결정됩니다. _(※2. 기본원리의 FOV 설명참조)_
    - 예제코드에서는 카메라평면보다 방향벡터가 약간 더 길어서 __FOV__ 는 90 °보다 작습니다.
        - 정확히 계산하자면 FOV = 2 * atan(0.66/1.0)=66.8°로 1인칭 슈팅게임(fps)에 적합함
    - 초기선언 이후에는, 입력키로 회전해서 방향벡터 __dir__ 과 카메라평면 __plane__ 의 값이 변경되더라도 항상 서로 수직이어야하고 동일한 길이가 유지되어야 합니다.
    - __time, oldTime__ 는 현재 및 이전 프레임의 시간을 저장합니다.
    - __time__ 과 __oldTime__ 의 시간차는 특정키를 눌렀을 때 (프레임 계산하는데 얼마나 걸리건 일정한 속도로 움직이기 위해) 이동거리를 결정하고, FPS를 측정하는데 사용할 것입니다.
- 여기까지 main함수의 선언부를 마치고 아래에서 본문을 이어서 설명합니다.
<br>

```cpp
  screen(screenWidth, screenHeight, 0, "Raycaster");

```
- 본문에서는 우선, screen()함수로 해상도를 지정해서 __화면을 생성__ 합니다.

    - 이 때 1280 * 1024 처럼 해상도를 높게 지정하면 광선추적 알고리즘이 빨라도 CPU에서 비디오카드로 전체화면을 불러오는게 너무 오래걸려서 렌더링이 느려지게 됩니다.
<br>

```cpp
 while(!done())
  {
```
- 화면을 생성한 후 바로 __게임루프가 시작__ 됩니다. 

    - 이 반복문은 계속 반복해서 전체 프레임을 그려내고 입력을 읽는 일을 합니다.
<br>

```cpp
 for(int x = 0; x < w; x++)
    {
      //calculate ray position and direction
      double cameraX = 2 * x / double(w) - 1; //x-coordinate in camera space
      double rayDirX = dirX + planeX * cameraX;
      double rayDirY = dirY + planeY * cameraX;
```
- 이제 진짜 __레이캐스팅을 시작__ 합니다. (for문)

    - 광선의 시작점은 플레이어 위치로 합니다. __(posX, posY)__
    - 레이캐스팅 반복문에 필요한 변수를 선언하고 값을 계산합니다.
    - __cameraX__ 는 for문의 x값(화면의 수직선)이 위치가 카메라평면에서 차지하는 x좌표 입니다.
        - for문의 x값이 0이면 (스크린의 왼쪽 끝이면) cameraX = -1
        - for문의 x값이 w/2이면 (스크린의 중앙이면) cameraX = 0
        - for문의 x값이 w이면 (스크린의 오른쪽 끝이면) cameraX = 1
    - 이를 활용해서 광선의 방향을 계산할 것입니다.
    - __rayDirX, rayDirY__ 는 광선의 방향벡터 입니다.
    - [앞서](https://github.com/365kim/raycasting_tutorial/blob/master/2_basics.md) 설명한 것과 같이 광선의 방향은 __( 방향벡터 ) + ( 카메라평면 x 배수 )__ 로 구할 수 있습니다. 벡터 x, y에 대해 각각 이 계산을 해줍니다.
    - 이 반복문은 스크린의 모든 x값(수직선)에 대해서 계산할 뿐, 모든 픽셀에 대해서 계산하는게 아니라 계산량이 얼마 안됩니다!
<br>

```cpp
      //which box of the map we're in
      int mapX = int(posX);
      int mapY = int(posY);

      //length of ray from current position to next x or y-side
      double sideDistX;
      double sideDistY;

       //length of ray from one x or y-side to next x or y-side
      double deltaDistX = std::abs(1 / rayDirX);
      double deltaDistY = std::abs(1 / rayDirY);
      double perpWallDist;

      //what direction to step in x or y-direction (either +1 or -1)
      int stepX;
      int stepY;

      int hit = 0; //was there a wall hit?
      int side; //was a NS or a EW wall hit?
```

- 이제 __DDA 알고리즘__ 과 관련된 변수를 선언하고 계산할 것입니다.

    - __mapX, mapY__ 는 현재 광선의 위치, 광선이 있는 한 칸(square) 입니다.
    - 광선의 위치 자체는 부동소수점수로 표현되서 광선이 맵상 어느 칸(square)에 있는지 그리고 그 한 칸안에서 어디쯤 있는지까지 알 수 있지만, __mapX, mapY__ 는 간단히 그 한 칸(square)의 좌표만 나타냅니다.
<br>
<br>

<p align="center"><img src="https://user-images.githubusercontent.com/60066472/83316382-6eeff080-a260-11ea-8e72-cbefdf2208c1.gif"></p>
    
- 위의 이미지는 초기 __sideDistX, sideDistY__ 및 __deltaDistX, deltaDistY__ 를 보여줍니다.

    - __sideDistX__ 는 '시작점 ~ 첫번째 x면을 만나는 점'까지의 광선의 이동거리 이고, 
    - __sideDistY__ 는 '시작점 ~ 첫번째 y면을 만나는 점'까지의 광선의 이동거리 입니다.
        - __sideDistX, sideDistY__ 의미는 나중에 지금과 다른 의미로 약간 변경될 예정입니다.
    - __deltaDistX__ 는 '첫번째 x면 ~ 바로 다음 x면'까지의 광선의 이동거리 입니다. (이때 x는 1만큼 이동)
    - __deltaDistY__  '첫번째 y면 ~ 바로 다음 y면'까지의 광선의 이동거리 입니다 (이때 y는 1만큼 이동)
    - 피타고라스 공식을 이용해서 __deltaDistX, deltaDistY__ 산출식을 위의 예제코드와 같이 쓸 수 있습니다.
        - 아래의 'deltaDist 공식유도' 참고
    - __perpWallDist__ 는 나중에 광선의 이동거리를 계산하는 데 사용할 것입니다. 
    
- DDA 알고리즘은 반복문을 실행할 때마다 x방향 또는 y방향으로 딱 한 칸(square)씩 점프합니다.

    - 광선의 방향에 따라 어느 방향으로 건너뛰는지 달라지는데 그 정보는 __stepX, stepY__ 에 +1 또는 -1 로 저장됩니다. _(주. stepX 또는 stepY 중 하나만 선택적으로 적용되는데 아래서 다시 설명나와요)_
   
- 마지막으로 벽의 x면 또는 y면과 부딪쳤는지 여부에 따라 루프를 종료할지 결정합니다.

    - __hit__ 은 벽과 부딪쳤는지 여부 (루프 종료조건) 입니다. 만약에 벽과 부딪혔고 그게 x면에 부딪힌 거라면 __side__ 의 값은 0으로, y면에 부딪히면 1이 됩니다. __x면, y면__ 은 두개의 칸(square)의 경계가 되는 부분의 선을 의미합니다.
<br>

<details>
<summary> deltaDist 공식유도 (눌러서 내용보기) </summary>
<div markdown="1">

```
deltaDistX = sqrt(1 + (rayDirY * rayDirY) / (rayDirX * rayDirX))
deltaDistY = sqrt(1 + (rayDirX * rayDirX) / (rayDirY * rayDirY))

But this can be simplified to:

deltaDistX = abs(|v| / rayDirX)
deltaDistY = abs(|v| / rayDirY)

Where |v| is the length of the vector rayDirX, rayDirY (that is sqrt(rayDirX * rayDirX + rayDirY * rayDirY)).
However, we can use 1 instead of |v|, 
because only the *ratio* between deltaDistX and deltaDistY matters for the DDA code that follows later below, 
so we get:

deltaDistX = abs(1 / rayDirX)
deltaDistY = abs(1 / rayDirY)

[thanks to Artem for spotting this simplification]
```

- 참고 : 만약 __rayDirX 또는 rayDirY__ 의 값이 0이면, 0으로 나누는 꼴이 되서 __deltaDistX 또는 deltaDistY__ 의 값이 무한대가 됩니다. 

    - 이 문제는 당신의 시스템이 _IEEE 754 부동소수점 표준_ 을 사용한다면, 이러한 경우에 예외를 발생시키지 않아 무한대도 DDA의 비교문에서도 제대로 작동하므로 괜찮습니다.
    - 예를 들어 C++, Java, JS에서는 괜찮지만 Python에서는 제대로 작동하지 않습니다. 
    - 만약 당신이 그렇지 않은 프로그래밍 언어를 사용한다면, 아래와 같은 코드로 DDA 반복문이 올바르게 작동하게 할 수 있습니다.

```cpp
      // Alternative code for deltaDist in case division through zero is not supported
      double deltaDistX = (rayDirY == 0) ? 0 : ((rayDirX == 0) ? 1 : abs(1 / rayDirX));
      double deltaDistY = (rayDirX == 0) ? 0 : ((rayDirY == 0) ? 1 : abs(1 / rayDirY));
```

</div>
</details>
<br>

<br>
<br>

```cpp
      //calculate step and initial sideDist
      if (rayDirX < 0)
      {
        stepX = -1;
        sideDistX = (posX - mapX) * deltaDistX;
      }
      else
      {
        stepX = 1;
        sideDistX = (mapX + 1.0 - posX) * deltaDistX;
      }
      if (rayDirY < 0)
      {
        stepY = -1;
        sideDistY = (posY - mapY) * deltaDistY;
      }
      else
      {
        stepY = 1;
        sideDistY = (mapY + 1.0 - posY) * deltaDistY;
      }
```
- DDA 알고리즘을 시작하기 전 __stepX, stepY 의 초기값__ 을 구해줍니다.

    - 만약 광선의 x방향 __rayDirX__ 의 값이 양수라면 __stepX__ 의 값은 __+1__ 로, 음수라면 __-1__ 로 설정합니다.
    - 만약  __rayDirX__ 의 값이 __0__ 라면, __stepX__ 는 사용되지 않으므로 어떤 값을 갖든 상관없습니다.
    - __stepY__ 의 값도 똑같이 구해줍니다.
    
- 그리고 __sideDistX, sideDistY 의 초기값__ 을 구해줍니다.
    - __sideDistX__ 의 값은, __rayDirX__ 의 값이 __양수__ 일 경우, '광선의 시작점부터 __오른쪽__ 으로 이동하다 처음 만나는 x면까지의 거리'입니다.
    - 반대로 __rayDirX__ 의 값이 __음수__ 일 경우, '광선의 시작점부터 __왼쪽__ 으로 이동하다 처음 만나는 x면까지의 거리'입니다.
     - __sideDistY__ 의 값도 마찬가지입니다. (시작점 기준 __위쪽__ 또는 __아래쪽__ )
    - __sideDistX, sideDistY__ 를 산출하기 위해 정수값인 __mapX__ 와 실제위치인 __posX__ 가 사용하고, 광선의 방향에 따라 산출식을 알맞게 설정합니다. 
    - __rayDirX__ 의 경우 __양수__ 일 경우, __sideDistX__ 는 __mapX + 1__ 에서 실제위치 __posX__ 를 빼주고 __deltaDistX__ 값을 곱해서 구할 수 있습니다.
    - __rayDirX__ 가 __음수__ 일 경우, __sideDistX__ 는 __posX__ 에서 __mapX__ 를 빼주고 __deltaDistX__ 값을 곱해서 구할 수 있습니다.
    - _(주. 아래의 이미지는 튜토리얼 원본과는 무관하게 이해를 돕기위해 rayDirX가 양수인 경우에 대해 추가한 이미지 입니다)_
　　　![spe](https://user-images.githubusercontent.com/60066472/83608224-852ae300-a5b7-11ea-874d-53a93bcdb093.jpg)
<br>
<br>

```cpp
      //perform DDA
      while (hit == 0)
      {
        //jump to next map square, OR in x-direction, OR in y-direction
        if (sideDistX < sideDistY)
        {
          sideDistX += deltaDistX;
          mapX += stepX;
          side = 0;
        }
        else
        {
          sideDistY += deltaDistY;
          mapY += stepY;
          side = 1;
        }
        //Check if ray has hit a wall
        if (worldMap[mapX][mapY] > 0) hit = 1;
      } 
```
- 이제 __DDA 알고리즘을 시작__ 합니다.

    - 이 while 반복문은 벽에 부딪힐 때까지 매번 한 칸(square)씩 광선을 이동시키는 루프입니다.
    - 반복할 때마다 __stepX__ 를 사용하면 x방향으로 한 칸 또는 __stepY__ 를 사용하면 y방향으로 한 칸 점프합니다. 항상 딱 한 칸씩만 점프합니다.
    - 만약 광선의 방향이 x축 방향과 완전히 일치한다면, (y방향이 바뀌지는 않을테니) 반복문을 돌 때 x방향으로만 한 칸씩 점프하면 됩니다.
    - 만약 광선이 y축 방향으로 아주 조금 기울어져 있으면 x방향으로 엄청 많이 점프하고나서야 y방향으로 1칸 점프할 것입니다.
    - 만약 광선의 방향이 y축 방향과 완전히 일치한다면, x방향으는 점프할 필요가 없는 식으로 반복문이 진행됩니다.
    - 광선이 점프할 때마다 __sideDistX, sideDistY__ 는 __deltaDistX, deltaDistY__ 가 더해지면서 값이 업데이트됩니다.
    - 광선이 점프할 때마다 __mapX, mapY__ 는 __stepX, stepY__ 가 더해지면서 값이 업데이트됩니다.

- 광선이 벽에 부딪히면 루프가 종료됩니다.
    - 이 때, 변수 __side__ 의 값이 0이면 벽의 x면에, 1이면 벽의 y면에 부딪혔다는 것을 알 수 있고, 또 __mapX, mapY__ 로 어떤 벽이랑 부딪힌 건지도 알 수 있습니다.
    - 우리는 그 벽에서 정확히 어느 지점에서 부딪힌 건지는 알 수 없는데, 지금은 텍스쳐 없이 색상만 표현하기 때문에 몰라도 괜찮습니다.
<br>
<br>

- 벽을 만나 DDA가 완료되었으니 이제 __광선의 시작점에서 벽까지의 이동거리__ 를 계산하겠습니다.

    - 이 거리는 나중에 벽을 얼마나 높게 그릴지 알아내는데 사용됩니다.
    - __어안렌즈 효과__ (fisheye effect) 는 실제 거리 값을 사용했을 때 모든 벽이 둥글게 보여서 회전할 때 울렁거릴 수도 있는 현상을 말합니다.
    - 이러한  __어안렌즈 효과__ 를 피하기 위해, 플레이어 위치까지의 유클리드 거리 대신에, 카메라 평면까지의 거리 (또는 카메라 쪽으로 플레이어에 투사된 지점의 거리)를 사용할 것입니다.

　　![13](https://user-images.githubusercontent.com/60066472/83316383-6f888700-a260-11ea-98a4-8f6bac2606bd.png)
- 위의 이미지는 플레이어 대신 카메라 평면까지 거리를 사용하는 이유를 보여줍니다. P는 플레이어의 위치, 흑색 선은 카메라평면을 나타냅니다.
    - 플레이어 기준 왼쪽에 있는 적색 선은, 벽의 적중지점(hit point)에서 플레이어까지 유클리드 거리를 나타내는 광선을 나타냅니다. 
    - 플레이어의 오른쪽의 녹색 선은, 벽의 적중지점(hit point)에서 플레이어가 아닌 카메라 평면으로 바로 이동하는 광선을 나타냅니다. 이 녹색선의 길이가 바로 우리가 유클리드 거리(실제 거리) 대신 사용할 수직거리입니다.
    - 이미지를 보시면, 플레이어는 벽을 정면으로 바라보고 있어 이 경우 벽의 윗선과 아랫선이 화면에서 완전히 수평을 이뤄야 합니다.
    - 이때 적색 선의 길이를 적용하면 적색 선의 길이가 다 다르기 때문에 벽의 높이가 일정하지 않게되고 결국 벽이 둥글게 보이는 게 되는 것입니다.
    - 반면에 오른쪽의 녹색 선은 모두 길이가 같아서 녹색 선을 적용하면 올바른 결과를 얻을 수 있습니다. 
    - 플레이어가 회전할 때 (카메라평면이 수평이 아니게 되고 녹색 선의 길이도 서로 달라지지만 서로 일정한 차이를 유지하면서 달라짐)에도 동일하게 적용되어 벽은 화면에 대각선이긴 하지만 직선으로 보이게 됩니다.
    - 이 설명은 완벽하진 않지만 어느 정도의 이해를 돕습니다.
<br>
<br>

```cpp
      //Calculate distance projected on camera direction (Euclidean distance will give fisheye effect!)
      if (side == 0) perpWallDist = (mapX - posX + (1 - stepX) / 2) / rayDirX;
      else           perpWallDist = (mapY - posY + (1 - stepY) / 2) / rayDirY;
```
- 여기서 사용되는 __레이캐스팅 방법__ 은 광선의 이동거리를 계산하면서, 어안렌즈 효과를 보정하는 코드를 따로 추가하지 않고도 간단히 방지할 수 있는 방법입니다.

    - 이 수직거리를 구하는 방식은 실제 이동거리를 구하는 방식보다 훨씬 쉽고, 벽에 어느 위치에 정확히 부딪혔는지 몰라도 구할 수 있습니다.
    - 위의 예제코드에서, __(1 - stepX) / 2__ 는 __stepX__ 가 -1 이면 1, __stepX__ 가 1 이면 0 이 됩니다. 이는 __rayDirX < 0__ 일 때 길이에 1을 더해주기 위한 코드입니다. 위에서 __sideDistX__ 의 초기값을 설정할 때 rayDirX의 방향에 따라 1을 더해주거나 말거나 했던 것과 같은 이유입니다.
- __수직거리를 계산하는 방법__ 은 다음과 같습니다.

    - 만약 광선이 처음으로 부딪힌 면이 x면이라면, __mapX - posX + (1 - stepX) / 2)__ 는 광선이 x방향으로 몇 칸이나 지나갔는지를 나타내는 수입니다 (정수일 필요는 없음).
    - 만약 광선의 방향이 x면에 수직이면 이미 정확한 수직거리의 값이지만 대부분의 경우 광선의 방향이 있고 이 때 구해진 값은 실제 수직거리보다 큰 값이므로 __rayDirX__ 로 나누어줍니다.
    - y면에 부딪힌 경우에도 같은 방식으로 계산해줍니다.
    - __mapX - posX__ 가 음수이더라도 역시 음수인 __rayDirX__ 로 나누어 계산된 값은 항상 양수가 됩니다.
<br>
<br>

　　![14](https://user-images.githubusercontent.com/60066472/83316384-6f888700-a260-11ea-94a4-313994efae2f.png)
  
- __perpWallDist__ 은 벽의 적중지점(hit point)와 플레이어의 카메라평면을 사용해서, 점에서 선까지의 거리를 구하는 공식을 적용해서 계산할 수도 있습니다. 하지만 이 공식은 앞의 더 간단한 공식보다는 계산량이 많습니다. 위의 이미지는 더 간단한 공식이 어떻게 도출되는지 보여줍니다.
- 이 설명은 y면에 부딪힌 경우(side == 1) 를 보여줍니다. x면에 부딪힌 경우도 같은 원리로 설명할 수 있습니다.

<details>
<summary> perpWallDist 공식유도 (눌러서 내용보기) </summary>
<div markdown="1">

```
The image shows:
P: position of the player
H: hitpoint of the ray on the wall
perpWallDist: the length of this line is the value to compute now, the distance perpenducilar from the wall hit point to the camera plane instead of Euclidean distance to the player point, to avoid making straight walls look rounded.
yDist matches what is "(mapY - posY + (1 - stepY) / 2)" in the code above, this is the y coordinate of the Euclidean distance vector, in world coordinates.
Euclidean is the Euclidean distance from the player P to the exact hit point H. Its direction is the rayDir, but its length is all the way to the wall.
rayDir: the direction of the ray marked "Euclidean", matching the rayDirX and rayDirY variables in the code. Note that its length |rayDir| is not 1 but slightly higher, due to how we added a value to (dirX,dirY) (the dir vector, which is normalized to 1) in the code.
rayDirX and rayDirY: the X and Y components of rayDir, matching the rayDirX and rayDirY variables in the code.
dir: the main player looking direction, given by dirX,dirY in the code. The length of this vector is always exactly 1. This matches the looking direction in the center of the screen, as opposed to the direction of the current ray. It is perpendicular to the camera plane, and perpWallDist is parallel to this.
orange dotted line (may be hard to see, use CTRL+scrollwheel or CTRL+plus to zoom in a desktop browser to see it better): the value that was added to dir to get rayDir. Importantly, this is parallel to the camera plane, perpendicular to dir.
camera plane: this is the camera plane, the line given by cameraX and cameraY, perpendicular to the main player's looking direction.

A: point of the camera plane closest to H, the point where perpWallDist intersects with camera plane
B: point of X-axis through player closest to H, point where yDist crosses X-axis through the player
C: point at player position + rayDirX
D: point at player position + rayDir.
E: This is point D with the dir vector subtracted, in other words, E + dir = D.
points A, B, C, D, E, H and P are used in the explanation below: they form triangles which are considered: BHP, CDP, AHP and DEP.

And the derivation of the perpWallDist computation above then is:

1: "(mapY - posY + (1 - stepY) / 2) / rayDirY" is "yDist / rayDirY" in the picture.
2: Triangles PBH and PCD have the same shape but different size, so same ratios of edges
3: Given step 2, the triangles show that the ratio yDist / rayDirY is equal to the ratio Euclidean / |rayDir|, so now we can derive perpWallDist = Euclidean / |rayDir| instead.
4: Triangles AHP and EDP have the same shape but different size, so same ratios of edges. Length of edge ED, that is |ED|, equals length of dir, |dir|, which is 1. Similarly, |DP| equals |rayDir|.
5: Given step 4, the triangles show that the ratio Euclidean / |rayDir| = perpWallDist / |dir| = perpWallDist / 1.
6: Combining steps 5 and 3 shows that perpWallDist = yDist / rayDirY, the computation used in the code above

[Thanks to Roux Morgan for helping to clarify the explanation of perpWallDist in 2020, the tutorial was lacking some information before this]
```

</div>
</details>
<br>
<br>

```cpp
      //Calculate height of line to draw on screen
      int lineHeight = (int)(h / perpWallDist);

      //calculate lowest and highest pixel to fill in current stripe
      int drawStart = -lineHeight / 2 + h / 2;
      if(drawStart < 0)drawStart = 0;
      int drawEnd = lineHeight / 2 + h / 2;
      if(drawEnd >= h)drawEnd = h - 1;
```
- 이제 계산한 거리 (perpWallDist)로, __화면에 그려야하는 선의 높이__ 를 구할 수 있습니다.

    - __perpWallDist__ 를 역수로 취하고, 픽셀단위로 맞춰주기 위해 픽셀 단위의 화면높이 __h__ 를 곱해서 구할 수 있습니다.
    - 벽을 더 높게 그리거나 낮게 그리거나 하고 싶으면 2 * h와 같은 다른 값을 넣을 수도 있습니다.
    - h값은 일정한 벽의 높이, 너비 및 깊이를 가진 박스처럼 보이게 해주고, 값이 클수록 높이가 높은 박스를 만들어줍니다.
    - 이렇게 구한 __lineHeight__ (화면에 그려야 할 수직선의 높이)에서, 실제로 선을 그릴 위치의 시작 및 끝 위치를 알 수 있습니다.
    - 벽의 중심은 화면의 중심에 있어야 하고, 이 중심점이 화면 범위 아래에 놓여있다면 0 으로, 화면 범위 위에 놓여있다면 h-1으로 덮어씌웁니다.
<br>
<br>

```cpp
      //choose wall color
      ColorRGB color;
      switch(worldMap[mapX][mapY])
      {
        case 1:  color = RGB_Red;  break; //red
        case 2:  color = RGB_Green;  break; //green
        case 3:  color = RGB_Blue;   break; //blue
        case 4:  color = RGB_White;  break; //white
        default: color = RGB_Yellow; break; //yellow
      }

      //give x and y sides different brightness
      if (side == 1) {color = color / 2;}

      //draw the pixels of the stripe as a vertical line
      verLine(x, drawStart, drawEnd, color);
    }
```
- 마지막으로, 광선이 부딪힌 벽의 색상값에 따라 __표현할 색상을 선택__ 해줍니다.

    - y면에 부딪힌 경우에 색상을 더 어둡게 설정하면 더 그럴듯하게 표현해 줄 수 있습니다.
    - 그리고 verLine() 함수로 수직선을 그려줍니다. 
    - 여기까지의 과정을 모든 x값에 대해 반복한 후 이것으로 raycasting loop가 종료됩니다.
<br>
<br>

```cpp
    //timing for input and FPS counter
    oldTime = time;
    time = getTicks();
    double frameTime = (time - oldTime) / 1000.0; //frameTime is the time this frame has taken, in seconds
    print(1.0 / frameTime); //FPS counter
    redraw();
    cls();

    //speed modifiers
    double moveSpeed = frameTime * 5.0; //the constant value is in squares/second
    double rotSpeed = frameTime * 3.0; //the constant value is in radians/second
```
- 레이캐스팅 loop를 마친 후, 현재 프레임과 이전 프레임의 시간을 계산합니다.

    - FPS (frame per second, 초당 프레임)가 계산되고 출력됩니다. 그리고
    - 벽과 FPS카운터의 값을 포함한 모든 것이 화면에 표시될 수 있도록 다시 그려집니다.
    루프가 완료된 후 현재 및 이전 프레임의 시간이 계산되고 FPS (초당 프레임)가 계산 및 인쇄되며 모든 화면 (모든 벽 및 fps 값)이되도록 화면이 다시 그려집니다.
    - cls()를 실행해서 backbuffer 가 비워서, 다시 다음프레임을 그릴 때 바닥과 천장이 그전의 프레임의 픽셀로 보이는 것이 아니라 검은색이 되도록 합니다.
    
- 속도조정자 (speed modifier) 는 frameTime 과 상수를 이용해서 입력키로 인한 이동속도 또는 회전속도를 결정합니다.

    - frameTime을 사용하면 이동속도 또는 회전속도를 프로세서의 속도와는 독립적으로 설정할 수 있습니다.
<br>
<br>

```cpp
    readKeys();
    //move forward if no wall in front of you
    if (keyDown(SDLK_UP))
    {
      if(worldMap[int(posX + dirX * moveSpeed)][int(posY)] == false) posX += dirX * moveSpeed;
      if(worldMap[int(posX)][int(posY + dirY * moveSpeed)] == false) posY += dirY * moveSpeed;
    }
    //move backwards if no wall behind you
    if (keyDown(SDLK_DOWN))
    {
      if(worldMap[int(posX - dirX * moveSpeed)][int(posY)] == false) posX -= dirX * moveSpeed;
      if(worldMap[int(posX)][int(posY - dirY * moveSpeed)] == false) posY -= dirY * moveSpeed;
    }
    //rotate to the right
    if (keyDown(SDLK_RIGHT))
    {
      //both camera direction and camera plane must be rotated
      double oldDirX = dirX;
      dirX = dirX * cos(-rotSpeed) - dirY * sin(-rotSpeed);
      dirY = oldDirX * sin(-rotSpeed) + dirY * cos(-rotSpeed);
      double oldPlaneX = planeX;
      planeX = planeX * cos(-rotSpeed) - planeY * sin(-rotSpeed);
      planeY = oldPlaneX * sin(-rotSpeed) + planeY * cos(-rotSpeed);
    }
    //rotate to the left
    if (keyDown(SDLK_LEFT))
    {
      //both camera direction and camera plane must be rotated
      double oldDirX = dirX;
      dirX = dirX * cos(rotSpeed) - dirY * sin(rotSpeed);
      dirY = oldDirX * sin(rotSpeed) + dirY * cos(rotSpeed);
      double oldPlaneX = planeX;
      planeX = planeX * cos(rotSpeed) - planeY * sin(rotSpeed);
      planeY = oldPlaneX * sin(rotSpeed) + planeY * cos(rotSpeed);
    }
  }
}
```
- 예제코드의 마지막 부분은 입력키를 다룹니다. 우선 readKeys()로 입력된 키값을 읽어옵니다.

    - 위쪽 화살표( ↑ )가 눌렸다면 플레이어는 앞으로 이동시켜야 하므로 posX에는 dirX를, posY에는 dirY를 더해줍니다.
    - 이것은 dirX와 dirY가 정규화된 벡터(길이 1)인 것을 전제로 합니다. 이 예제코드에서는 처음에 그렇게 설정되었습니다.
    - 위 코드에는 간단한 충돌감지 기능도 포함되어있습니다. 새로운 위치가 벽에 포함된다면 플레이어가 움직이지 않도록 합니다.
    - 이 충돌감지 기능은 플레이어 위치 단일지점 대신에 플레이어를 중심으로 하는 원을 그려, 이 원이 벽 내부로 들어가는지 확인하는 방식으로 개선할 수도 있습니다.
    - 아래쪽 화살표( ↓ )를 눌렀을 때와 동일한 방식으로 하되 값을 빼주면 됩니다.
    - 왼쪽 화살표( ← ) 또는 오른쪽 화살표( → )가 눌려서 회전시켜야 한다면, 회전행렬을 곱하는 공식을 이용해서 방향벡터와 카메라평면벡터 둘 다 회전시켜줍니다.

<br>
<br>

![15](https://user-images.githubusercontent.com/60066472/83316385-70211d80-a260-11ea-9342-996e6bdda09a.gif)

- 이것으로 색상만있는 레이캐스터의 예제코드가 끝났습니다. 플레이어는 위의 이미지와 같은 맵을 돌아다닐 수 있습니다.
<br>
<br>

![16](https://user-images.githubusercontent.com/60066472/83316387-70211d80-a260-11ea-8c0a-1df7e4ab0700.gif)

- 위의 이미지는 카메라 평면이 방향 벡터와 수직이 아닌 경우의 예로, 화면이 왜곡되어 보이게 됩니다.

<br>
<br>


[원문링크](https://lodev.org/cgtutor/raycasting.html#Untextured_Raycaster_)
<br>
<br>
[[전편으로] 기초 : 아주 기본적인 원리](https://github.com/365kim/raycasting_tutorial/blob/master/2_basics.md)<br>
[__[다음편으로] 고급 : 예제코드로 이해하는 레이캐스터 구현 (textured)__](https://github.com/365kim/raycasting_tutorial/blob/master/4_textured_raycaster.md)
