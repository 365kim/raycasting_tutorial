[목록으로](https://github.com/365kim/raycasting_tutorial)

## :crystal_ball: Wolfenstein의 3D 텍스처

- 텍스처를 생성해서 사용하는 대신 이미지를 가져와 봅시다! 

    - 다음 8 개의 텍스처는 실제 Wolfenstein 3D에서 가져온 것으로, ID Software에 저작권이 있습니다.
    - [여기](https://lodev.org/cgtutor/files/wolftex.zip)에서 텍스처를 다운로드 할 수 있습니다.
　　　![18](https://user-images.githubusercontent.com/60066472/83316392-71524a80-a260-11ea-9920-08d1fd135f9c.png)

- 텍스처 패턴을 생성하는 코드만 다음과 같이 바꿔주면 사용할 수 있습니다. 

    - 사용 시 텍스처 파일의 경로가 올바르게 설정되었는지 체크하세요.
```cpp
  //generate some textures
  unsigned long tw, th;
  loadImage(texture[0], tw, th, "pics/eagle.png");
  loadImage(texture[1], tw, th, "pics/redbrick.png");
  loadImage(texture[2], tw, th, "pics/purplestone.png");
  loadImage(texture[3], tw, th, "pics/greystone.png");
  loadImage(texture[4], tw, th, "pics/bluestone.png");
  loadImage(texture[5], tw, th, "pics/mossy.png");
  loadImage(texture[6], tw, th, "pics/wood.png");
  loadImage(texture[7], tw, th, "pics/colorstone.png");
```
<br>
<br>

　　　![19](https://user-images.githubusercontent.com/60066472/83316393-71eae100-a260-11ea-80f8-ccafdb01a76f.gif)
![19-2](https://user-images.githubusercontent.com/60066472/83316394-71eae100-a260-11ea-8963-82142bff89ea.gif)

- 원래 Wolfenstein 3D 게임에서도 그림자 효과를 주기 위해서 한쪽 벽의 색상을 반대쪽 면보다 어둡게 해주었는데, 같은 텍스처에 대해 밝은 버전, 어두운 버전으로 나누어서 매번 다른 버전을 적용하는 방식을 채택했습니다.
- 우리가 사용한 방식은 하나의 텍스처만으로 R, G, B 값을 2로 나누어 y면을 더 어둡게 해주는 방식입니다.
<br>
<br>


[[원문링크]](https://lodev.org/cgtutor/raycasting.html#Wolfenstein_3D_Textures_)
<br>
<br>
[[전편으로] 고급 : 예제코드로 이해하는 레이캐스터 구현 (textured)](https://github.com/365kim/raycasting_tutorial/blob/master/4_textured_raycaster.md)
