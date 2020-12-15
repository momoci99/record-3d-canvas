# Record-3d-Canvas

three.js를 이용한 3d canavs와 2d canvas를 사용한 레코딩을 비교하는 코드입니다.

Inspired with
https://codepen.io/srzxrjnb/pen/XWdjyOd

Thanks for
https://github.com/hissinger

## 간략적인 결과

저해상도 레코딩에서는 큰 차이가 없었으나 1920\*1080 부터는 확연한 차이가 발생함

### 2d canvas

- 저장된 영상의 프레임 드랍 심함(10frame 수준)
- 시간이 경과됨에 따라 Chrome의 CPU 자원 소모율 급격히 증가

### Threejs 3d canvas

- 저장된 영상의 프레임 드랍 거의 없음.
- 시간이 경과함에 상관없이 CPU 및 GPU자원 소모 일정함

## 결론

Threejs 사용합시다.
