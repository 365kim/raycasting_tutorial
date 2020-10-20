[목록으로](https://github.com/365kim/raycasting_tutorial)

## :crystal_ball: 예제코드로 이해하는 레이캐스터 구현 (textured)
- 예제코드 전체보기 : [raycaster_textured.cpp](https://lodev.org/cgtutor/files/raycaster_textured.cpp)

- __텍스처를 표현한 레이캐스터__ 는 마지막 즈음에 텍스처와 관련된 계산을 좀 더해주는 것과 각 픽셀이 어떤 __텍셀(texel, texture pixel)__ 값을 갖는지 결정해주기 위해 모든 픽셀을 통과하는 y방향 반복문 외에는, 텍스처 없이 색상만 표현한 레이캐스터와 그 핵심은 거의 같습니다.

    - 이번에는 verLine() 함수로 수직선을 그리는 방식은 사용할 수 없습니다.
    - 대신 픽셀을 하나하나 그려주는 방식으로 해서 텍스처를 적용할 것입니다.
    - 가장 좋은 방법은 __스크린버퍼__ 로 2차원 배열을 사용해서 화면에 한 번 출력해주는 것입니다. 이 방법이 pset을 사용한 것보다 훨씬 빠릅니다.
    - 텍스처를 위한 별도의 배열 또한 필요하고, __drawbuffer()__ 함수가 R,G,B의 3개의 값이 아닌 int 1개로 작동하기 때문에 이 포맷에 텍스처도 저장해주어야 합니다.
    - 보통 별도의 텍스처 파일로부터 텍스처를 불러오는데 이 예제코드는 간단한 버전이니까 대충 만든 텍스처로 하겠습니다.
    - 이 예제코드는 이전의 예제와 거의 똑같고 굵게(bold) 표시된 부분이 새로 추가된 부분입니다. 아래 이어지는 설명에는 추가된 부분만 다룰 것입니다. <br>_(주. 에서 bold표시 방법을 찾지못해 bold표시가 되어있지 않습니다.)_
<br>
<br>

```cpp
#define screenWidth 640
#define screenHeight 480
#define texWidth 64
#define texHeight 64
#define mapWidth 24
#define mapHeight 24

int worldMap[mapWidth][mapHeight]=
{
  {4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,7,7,7,7,7,7,7,7},
  {4,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,7,0,0,0,0,0,0,7},
  {4,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,7},
  {4,0,2,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,7},
  {4,0,3,0,0,0,0,0,0,0,0,0,0,0,0,0,7,0,0,0,0,0,0,7},
  {4,0,4,0,0,0,0,5,5,5,5,5,5,5,5,5,7,7,0,7,7,7,7,7},
  {4,0,5,0,0,0,0,5,0,5,0,5,0,5,0,5,7,0,0,0,7,7,7,1},
  {4,0,6,0,0,0,0,5,0,0,0,0,0,0,0,5,7,0,0,0,0,0,0,8},
  {4,0,7,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,7,7,7,1},
  {4,0,8,0,0,0,0,5,0,0,0,0,0,0,0,5,7,0,0,0,0,0,0,8},
  {4,0,0,0,0,0,0,5,0,0,0,0,0,0,0,5,7,0,0,0,7,7,7,1},
  {4,0,0,0,0,0,0,5,5,5,5,0,5,5,5,5,7,7,7,7,7,7,7,1},
  {6,6,6,6,6,6,6,6,6,6,6,0,6,6,6,6,6,6,6,6,6,6,6,6},
  {8,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,4},
  {6,6,6,6,6,6,0,6,6,6,6,0,6,6,6,6,6,6,6,6,6,6,6,6},
  {4,4,4,4,4,4,0,4,4,4,6,0,6,2,2,2,2,2,2,2,3,3,3,3},
  {4,0,0,0,0,0,0,0,0,4,6,0,6,2,0,0,0,0,0,2,0,0,0,2},
  {4,0,0,0,0,0,0,0,0,0,0,0,6,2,0,0,5,0,0,2,0,0,0,2},
  {4,0,0,0,0,0,0,0,0,4,6,0,6,2,0,0,0,0,0,2,2,0,2,2},
  {4,0,6,0,6,0,0,0,0,4,6,0,0,0,0,0,5,0,0,0,0,0,0,2},
  {4,0,0,5,0,0,0,0,0,4,6,0,6,2,0,0,0,0,0,2,2,0,2,2},
  {4,0,6,0,6,0,0,0,0,4,6,0,6,2,0,0,5,0,0,2,0,0,0,2},
  {4,0,0,0,0,0,0,0,0,4,6,0,6,2,0,0,0,0,0,2,0,0,0,2},
  {4,4,4,4,4,4,4,4,4,4,1,1,1,2,2,2,2,2,2,3,3,3,3,3}
};
```
- 이번에는 screen()함수에서, 그리고 스크린버퍼를 생성할 때 그 값이 필요하기 때문에 __screenWidth, screenHeight__ 를 초기에 정의 해줄것입니다.
- __texWidth, texHeight__ 도 새로 정의해줍니다. 이 값은 텍스쳐의 텍셀(texel) 이 갖는 너비와 높이 값이겠죠.
- 맵도 조금 바뀌었습니다. 다양한 텍스처를 보여주기 위해 복도(corridor)와 방(room)이 추가되었습니다. 아시다시피, 0은 플레이어가 지나갈 수 있는 빈 공간이고 양수값은 각 값에 대응되는 텍스처를 나타냅니다.
<br>
<br>

```cpp
int main(int argc, char *argv[])
{
  double posX = 22.0, posY = 11.5;  //x and y start position
  double dirX = -1.0, dirY = 0.0; //initial direction vector
  double planeX = 0.0, planeY = 0.66; //the 2d raycaster version of camera plane

  double time = 0; //time of current frame
  double oldTime = 0; //time of previous frame

  Uint32 buffer[screenHeight][screenWidth]; // y-coordinate first because it works per scanline
  std::vector texture[8];
  for(int i = 0; i < 8; i++) texture[i].resize(texWidth * texHeight);
```
- 스크린버퍼 배열 __buffer__ 와 텍스처 배열 __texture__ 를 선언해줍니다. 

    - std::vectors로 텍스처 배열은 총 8가지 종류의 텍스처를 저장할 수 있고, 위에서 define한 texWidth, texHeight 만큼의 크기를 갖게합니다.

<br>
<br>

```cpp
screen(screenWidth,screenHeight, 0, "Raycaster");

  //generate some textures
  for(int x = 0; x < texWidth; x++)
  for(int y = 0; y < texHeight; y++)
  {
    int xorcolor = (x * 256 / texWidth) ^ (y * 256 / texHeight);
    //int xcolor = x * 256 / texWidth;
    int ycolor = y * 256 / texHeight;
    int xycolor = y * 128 / texHeight + x * 128 / texWidth;
    texture[0][texWidth * y + x] = 65536 * 254 * (x != y && x != texWidth - y); //flat red texture with black cross
    texture[1][texWidth * y + x] = xycolor + 256 * xycolor + 65536 * xycolor; //sloped greyscale
    texture[2][texWidth * y + x] = 256 * xycolor + 65536 * xycolor; //sloped yellow gradient
    texture[3][texWidth * y + x] = xorcolor + 256 * xorcolor + 65536 * xorcolor; //xor greyscale
    texture[4][texWidth * y + x] = 256 * xorcolor; //xor green
    texture[5][texWidth * y + x] = 65536 * 192 * (x % 16 && y % 16); //red bricks
    texture[6][texWidth * y + x] = 65536 * ycolor; //red gradient
    texture[7][texWidth * y + x] = 128 + 256 * 128 + 65536 * 128; //flat grey texture
  }
```
- 이제 메인함수를 시작합니다. 제일 먼저 텍스처 생성부터 해줍니다.

    - 텍스처 크기의 모든 픽셀은 텍스처 높이와 너비만큼 이중 반복문을 통과하면서, 텍스처 번호마다 x, y값으로 만든 특정한 값을 갖게됩니다.
    - 이 값들은 XOR 패턴, 그라데이션, 또는 벽돌st 패턴을 나타내게 됩니다.
    - 이런 간단하게 생성한 패턴 말고 좀 더 예쁜 패턴을 사용하고 싶다면 [다음 편](https://github.com/365kim/42_la_piscine/blob/master/blob/master/5_wolfenstein_texture.md)을 참고해주세요.
<br>
<br>

```cpp
 //start the main loop
  while(!done())
  {
    for(int x = 0; x < w; x++)
    {
      //calculate ray position and direction
      double cameraX = 2*x/double(w)-1; //x-coordinate in camera space
      double rayDirX = dirX + planeX*cameraX;
      double rayDirY = dirY + planeY*cameraX;

      //which box of the map we're in
      int mapX = int(posX);
      int mapY = int(posY);

      //length of ray from current position to next x or y-side
      double sideDistX;
      double sideDistY;

      //length of ray from one x or y-side to next x or y-side
      double deltaDistX = sqrt(1 + (rayDirY * rayDirY) / (rayDirX * rayDirX));
      double deltaDistY = sqrt(1 + (rayDirX * rayDirX) / (rayDirY * rayDirY));
      double perpWallDist;

      //what direction to step in x or y-direction (either +1 or -1)
      int stepX;
      int stepY;

      int hit = 0; //was there a wall hit?
      int side; //was a NS or a EW wall hit?

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
- 이전 편에서 봤던 것과 같이 게임루프를 시작하고 DDA 알고리즘 전에 변수를 선언, 초기화하고 필요한 계산을 해줍니다. 바뀐 내용은 없습니다.
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

      //Calculate distance of perpendicular ray (Euclidean distance will give fisheye effect!)
      if (side == 0) perpWallDist = (mapX - posX + (1 - stepX) / 2) / rayDirX;
      else           perpWallDist = (mapY - posY + (1 - stepY) / 2) / rayDirY;

      //Calculate height of line to draw on screen
      int lineHeight = (int)(h / perpWallDist);

      //calculate lowest and highest pixel to fill in current stripe
      int drawStart = -lineHeight / 2 + h / 2;
      if(drawStart < 0) drawStart = 0;
      int drawEnd = lineHeight / 2 + h / 2;
      if(drawEnd >= h) drawEnd = h - 1;
```
- 이제 DDA 알고리즘을 시작해서 거리와 높이를 계산해줍니다. 역시 바뀐 내용은 없습니다.
<br>
<br>

```cpp
   //texturing calculations
      int texNum = worldMap[mapX][mapY] - 1; //1 subtracted from it so that texture 0 can be used!

      //calculate value of wallX
      double wallX; //where exactly the wall was hit
      if (side == 0) wallX = posY + perpWallDist * rayDirY;
      else           wallX = posX + perpWallDist * rayDirX;
      wallX -= floor((wallX));

      //x coordinate on the texture
      int texX = int(wallX * double(texWidth));
      if(side == 0 && rayDirX > 0) texX = texWidth - texX - 1;
      if(side == 1 && rayDirY < 0) texX = texWidth - texX - 1;
```
- 이전 코드에서 벽의 색상을 선택해 주었다면, 이번 코드에서는 벽의 텍스처를 선택해줄 것입니다.
- 선택된 텍스처종류를 나타내는 변수 __texNum__ 은 맵에서 광선이 부딪힌 벽 한 칸(square)이 가진 값에서 1을 빼서 구할 수 있습니다.

    - 1을 빼주는 이유는 0번째 텍스처도 0으로 표현되고 벽이 없는 것도 0으로 표현되기 때문입니다. 
    - 그러므로 worldMap[mapX][mapY]의 값이 1이면 texNum이 0이 되서 텍스처 종류 중 맨 첫번째 텍스처를 사용할 수 있게 해주는 것입니다.

- __wallX__ 의 값은 벽의 int형 좌표가 아닌 double형 좌표로 벽의 정확히 어디에 부딪혔는지를 나타냅니다.

    - 이 값은 텍스처를 적용할 때 어떤 x좌표를 사용해야 하는지 판단할 때 사용할 것입니다.
    - 우선 부딪힌 곳의 정확한 x, y값(double)에서 벽의 x, y값(int)을 빼서 판단할 수 있습니다.
    - 변수 __wallX__ 는 x면과 부딪힌 경우(side == 0)인 경우 이름에서 유추할 수 있듯 벽의 x좌표가 맞지만, y면에 부딪힌 경우(side == 1)에는 벽의 y좌표가 된다는 점에 유의하세요. 이러나 저러나 텍스처를 적용할 때 __wallX__ 의 값은 텍스처의 x좌표에 사용됩니다.

- 코드의 마지막 부분에서 __wallX__ 로, 텍스처의 x좌표를 나타내는 __texX__ 를 계산해 주었습니다.

    - 이제 우리는 벽에 텍스처표현을 해주기 위해 텍스처의 어떤 x좌표 __texX__ 를 적용해야 하는지 알아냈습니다. 이 x좌표는 해당 수직선 상에서 그대로 유지됩니다.
<br>
<br>

```cpp
            // How much to increase the texture coordinate per screen pixel
      double step = 1.0 * texHeight / lineHeight;
      // Starting texture coordinate
      double texPos = (drawStart - h / 2 + lineHeight / 2) * step;
      for(int y = drawStart; y<drawEnd; y++)
      {
        // Cast the texture coordinate to integer, and mask with (texHeight - 1) in case of overflow
        int texY = (int)texPos & (texHeight - 1);
        texPos += step;
        Uint32 color = texture[texNum][texHeight * texY + texX];
        //make color darker for y-sides: R, G and B byte each divided through two with a "shift" and an "and"
        if(side == 1) color = (color >> 1) & 8355711;
        buffer[y][x] = color;
      }
    }
```

- 이제 수직선 상 각 픽셀이 텍스처의 어떤 y좌표 __texY__ 을 갖게할건지 정해주기 위해 y방향 반복문을 돌 차례입니다. 반복문을 돌면서 최종적으로 구한 값을 화면버퍼 __buffer[y][x]__ 에 하나하나 넣어줄 것입니다.
    - __texY__ 의 값은 첫 줄에서 계산한 __step__ 의 크기만큼 증가하면서 계산됩니다.
    - __step__ 의 크기는 텍스처의 좌표를 수직선 상에 있는 좌표에 대해 얼마나 늘려야 하는지에 따라 결정됩니다. 그리고 부동소수점 수인 double형에서 int형으로 캐스팅해주어 텍스처 픽셀값을 선택할 수 있도록 합니다.
    - 화면버퍼 buffer[y][x]에 넣을 픽셀의 색상 __color__ 는 __texture[texNum][texX][texY]__ 로 쉽게 가져올 수 있습니다.

- 색상만 있는 레이캐스터 때와 마찬가지로 광선이 벽의 y면에 부딪힌 경우(side == 1)에 색상값은 좀 더 어둡게 표현해 줄 겁니다. (그렇게 해야 좀 더 자연스러운 조명표현이 되기 때문입니다.)

    - 그런데 색상값이 R, G, B 3개로 나뉘어져 있는게 아니라 하나의 int 값으로 되어있어서, 계산방식이 직관적이지는 않습니다.
    - R, G, B를 2로 나누어서 색상을 어둡게 해주려고 합니다.
    - 십진수를 10으로 나누면 마지막 자리를 없앨 수 있는 것처럼 (300 / 10 하면 30으로 마지막 숫자 0이 제거됨) 이진수를 2로 나누는 것은 마지막 비트를 제거하는 것과 같습니다.
    - 여기서는 비트연산 중 오른쪽 시프트연산 __>>1__ 을 사용해서 마지막 비트를 제거해주었습니다.
    - 그런데 여기서는 24비트 int형(실제로 32 비트이지만 첫 8비트는 사용되지 않음)을 시프트연산 해주는 것이라 이전 8비트의 마지막 비트가 다음 8비트의 첫 번째 비트가 되어 색상 값 전체가 엉망이 될 수 있습니다!
    - 따라서 시프트연산 후, 모든 바이트의 첫 번째 비트는 0으로 설정해주어야 하고, 방법은 이진수 01111111 01111111 01111111(10진수로는 8355711)를 AND연산해주면 됩니다.
    - 따라서 위 예제코드처럼 해주면 실제보다 어두운 색으로 설정해줄 수 있습니다.
    - 최종적으로 계산된 __color__ 의 값이 현재 버퍼픽셀의 색상으로 설정되면 반복문의 한 사이클을 마치고 다음 y로 넘어갑니다.


__참고__ : 브레즌햄(bresenham) 알고리즘 또는 DDA 알고리즘을 이용해서 더 빠르게 하는 방법도 있겠습니다.

__참고__ : 여기서 step을 사용한 방식은 [__아핀 텍스처매핑__](https://en.wikipedia.org/wiki/Texture_mapping#Affine_texture_mapping) 방법입니다. 각 픽셀에 대해 각각 나눗셈을 하지않고 두 점 사이를 선형보간하는 방식입니다. 이 방법은 일반적으로 원근법을 정확하게 표현해주지 못하지만 지금처럼 완벽하게 수직인 벽(그리고 완벽하게 수평인 천장과 바닥)인 경우에는 올바르게 나타납니다.

<br>
<br>

```cpp
 drawBuffer(buffer[0]);
    for(int y = 0; y < h; y++) for(int x = 0; x < w; x++) buffer[y][x] = 0; //clear the buffer instead of cls()
    //timing for input and FPS counter
    oldTime = time;
    time = getTicks();
    double frameTime = (time - oldTime) / 1000.0; //frametime is the time this frame has taken, in seconds
    print(1.0 / frameTime); //FPS counter
    redraw();

    //speed modifiers
    double moveSpeed = frameTime * 5.0; //the constant value is in squares/second
    double rotSpeed = frameTime * 3.0; //the constant value is in radians/second
```

- 이제 화면버퍼를 화면에 그려주고, 그 후에 버퍼를 비워주면 됩니다. (색상만 있는 버전에서는 단순히 "cls"를 사용하면 됐습니다).
- 이 코드의 나머지 부분은 색상만 있는 버전과 동일합니다.
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

- 그리고 여기 입력키가 다시 있는데, 여기에서도 아무것도 바뀌지 않았습니다.
- 연사하는 기능(strafe, 왼쪽 또는 오른쪽에 연사)을 추가하고 싶다면 원하는 경우 위, 아래 키와 같은 방식으로 만들되, __dirX, dirY__ 대신 __planeX, planeY__ 를 사용하십시오.
<br>
<br>

　　　![17](https://user-images.githubusercontent.com/60066472/83316388-70b9b400-a260-11ea-8904-3578892c9a32.gif)
![17-2](https://user-images.githubusercontent.com/60066472/83316389-70b9b400-a260-11ea-8998-f0ad7881ce51.gif)

　　　![17-3](https://user-images.githubusercontent.com/60066472/83316391-71524a80-a260-11ea-9829-59b85a0c8281.gif)<br>
   
- 예제코드로 구현한 결과의 스크린 샷입니다.
<br>
<br>

```cpp
  //swap texture X/Y since they'll be used as vertical stripes
  for(size_t i = 0; i < 8; i++)
  for(size_t x = 0; x < texSize; x++)
  for(size_t y = 0; y < x; y++)
  std::swap(texture[i][texSize * y + x], texture[i][texSize * x + y]);
```

__참고__ : 일반적으로 이미지는 수평선 단위로 저장되지만 __레이캐스터__ 에서는 텍스처가 수직선으로 그려집니다. 그러므로 페이지 누락을 피하고 CPU 캐시를 최적화 하기 위해서는 텍스처도 수평선이 아니라 수직선으로 저장해주는게 효과적일 것입니다. 이를 위해서는 텍스처를 생성해준 후 X와 Y를 다음과 같이 swap해주세요.  (이 코드는 texWidth와 texHeight가 동일한 경우에만 작동합니다). 아예 텍스처를 생성하면서 바로 X와 Y를 바꿔줄 수도 있습니다. 그러나 대부분의 경우에 다른 포맷의 이미지나 텍스처를 가져오면 수평선으로 저장되어 있을 것이기 때문에 이 방법으로 swap 해주어야 할 것입니다.

- 텍스처에서 픽셀을 가져올 때는 다음 코드를 대신 사용하십시오.
```cpp
Uint32 color = texture[texNum][texSize * texX + texY];
```

<br>
<br>

[[원문링크]](https://lodev.org/cgtutor/raycasting.html#Textured_Raycaster)
<br>
<br>
[[전편으로] 중급 : 예제코드로 이해하는 레이캐스터 구현 (untextured)](https://github.com/365kim/raycasting_tutorial/blob/master/3_untextured_raycaster.md)
<br>
[__[다음편으로] 보충 : Wolfenstein의 3D 텍스처__](https://github.com/365kim/raycasting_tutorial/blob/master/5_wolfenstein_texture.md)
