---
title: 시야 그리고 FOV에 관하여
description : kyoulee blog
preview: /growth%20log/images/pasted%20image%2020260314091705.jpg
created: 2026-03-14T09:03:29+09:00
updated: 2026-03-14T09:03:29+09:00
tags:
  - FOV
  - Graphics
status: public
id : 0
writer : kyoulee
---

## 눈과 시야

![[Growth Log/images/Pasted image 20260314091705.jpg]]
> *[이미지 출처 : [displaymodule.com](https://www.displaymodule.com/)]*

이번에 방통대 그래픽스 수업을 마치며 몇 가지 의문점 중 시야각에 대한 설명을 듣고 FOV의 한계가 있는 방식의 로직이라는 것이 보이며 한계가 없는 FOV에 대하여 고민하고 설계 하였던 방식을 작성해 보려 한다.

## Pinhole Camera Model

![[Growth Log/images/Pasted image 20260314100143.png]]
> *[이미지 출처 : [wiki](https://en.wikipedia.org/wiki/Pinhole_camera_model)]*

기존의 보편적으로 사용하는 방식이다. 
간단히 설명하면 일정한 거리를 사용하여 하나의 판으로서 시야를 나타네는 방식이다. 
이는 시야점과 실제 디스플레이의 픽셀방식을 고려하여 만들었을 것이라 추축한다. 
디스플레이의 픽셀당 하나의 [[RayTracing]]을 계산하여 화면에 출력하는 방식으로 할 수 있기 때문이다.

3차원 공간의 점 $P(X, Y, Z)$가 카메라 렌즈 중심(Optical Center)을 지나 이미지 평면의 $p(x, y)$에 투영될 때 원하는 FOV(화각)값을 입력하여 필요한 화각만큼을 계산한다고 가정한다면 이는 $tan$를 사용하는 것이 좋아보이고 최대 화각만을 구해 범위 내의 값들을 픽셀만큼 분할 하여 [[RayTracing]]을 생성한다면 우리가 보는 디스플레이가 될 것이라 예측한다.

이는 계산을 용이하기위해 $tan$ 밑변을 1로 즉 중심점과 평면의 거리를 1로 하고 나머지 x 혹은 y 의 값으로 최대 픽셀을 계산하여 픽셀간의 간격을 구하면 될거같다.

### Develop

계산 효율성을 위해 시점(Pinhole)과 평면 사이의 거리를 **1**로 고정

* **초점 거리 ($f$):** 1
* **시점 (Origin):** $(0, 0, 0)$
* **이미지 평면 위치:** $Z = -1$ (카메라 전방)

원하는 화각을 입력하여 이미지 평면의 최대 경계값($x_{max}, y_{max}$)을 설정하여

$$x_{max} = \tan\left(\frac{FOV_h}{2}\right)$$
$$y_{max} = \tan\left(\frac{FOV_v}{2}\right)$$

디스플레이 해상도($W \times H$)를 고려하여 자를 것인지 아님 축소할 것인지도 고민해 봐야할 거 같다.

- **수평 픽셀 간격:** $\Delta x = \frac{2 \cdot x_{max}}{W}$
- **수직 픽셀 간격:** $\Delta y = \frac{2 \cdot y_{max}}{H}$

> [!Tip]
> 이중 자르는 것을 택하였다면 $x$의 픽셀간격으로 그려주는 것을 선택하는 것도 나쁘지 않다 생각한다.

이를 코드로 표현하면 다음과 같이 표현이 가능하며

```cpp
void Render(int W, int H, float fov_h, Vector3* rayDir) {
    const float PI = 3.1415926535f;
    float x_max = std::tan((fov_h * PI / 180.0f) / 2.0f);
    float delta = (2.0f * x_max) / W;

    for (int j = 0; j < H; ++j) {
        for (int i = 0; i < W; ++i) {
            rayDir[j * W + i] = CalculateRayDirection(i, j, W, H, x_max, delta);
        }
    }
}
```

특정 픽셀 $(i, j)$를 통과하는 광선의 방향 벡터 $D$를 구하는 식

$$
\begin{aligned}
D_x &= -x_{max} + (i + 0.5) \cdot \Delta x \\
D_y &= y_{max} - (j + 0.5) \cdot \Delta y \\
D_z &= -1
\end{aligned}
$$

이를 이용하여 픽셀마다의 방향 백터를 구할 수 있게 된다.

```cpp
Vector3 CalculateRayDirection(int i, int j, int W, int H, float x_max, float delta) {
    Vector3 D;
    
    D.x = -x_max + (i + 0.5f) * delta;
    D.y = ((H / 2.0f) - (j + 0.5f)) * delta;
    D.z = -1.0f;

    return D.normalize();
}
```

프로세스는 다음과 같은 내용으로 만들어 질거 같다.

| Step | Brief |
| :--- | :--- |
| **Setup** | $FOV$, $W$, $H$ 값을 입력 |
| **Pre-calc** | $x_{max}, y_{max}$ 및 $\Delta x, \Delta y$를 미리 계산 |
| **Iteration** | 모든 픽셀 $(i, j)$에 대하여:<br> - 방향 벡터 $D$를 생성하고 정규화($\hat{d}$)<br> - $Ray(t) = \vec{O} + t\hat{d}$ 식을 이용해 공간 내 물체와의 교점을 탐색<br> - 충돌 지점의 색상을 해당 픽셀에 렌더링. |

## Spherical Panoramic Camera Model

![[Growth Log/images/Pasted image 20260314113456.png]]
> *[이미지 출처 : [GPPA President](https://www.youtube.com/watch?v=CqWzHQ5gjAs)]*

필자가 생각하는 모습에 가까운 방식으로 보인다. 
구현한 모습은 다음과 같다. 

![[Growth Log/images/fd2fd57869f87597f231646242310ecb.mov]]

시점에 대하여 시각 벡터를 기반으로 FOV 를 계산하는 방식임으로 다음과 같은 순서가 필요하다

시점 좌표 $(x, y, z)$ 에 대하여 방향백터 $\overrightarrow{(x, y, z)}$ 를 기준으로 설정
방향백터기준의 [[Quaternion]]를 생성하여 중심기준으로 최대각에 대한 값과 기준점에 대한 값을 $x$축을 기준으로 실제 필요한 픽셀 수 만큼 분리

[[Quaternion]]의 $x$축 기준으로 $y$축을 추가로 회전하여 동일한 FOV기준으로 회전하는 $y$축을 구현

$$\theta_{rel} = (i - \frac{W}{2}) \cdot \Delta\theta \quad (\text{Yaw})$$
$$\phi_{rel} = (j - \frac{H}{2}) \cdot \Delta\theta \quad (\text{Pitch})$$

$$q_{pixel(i,j)} = \text{RotationXYZ}(\phi_{rel}, \theta_{rel}, 0)$$

코드로는 다음과 같이 표현 가능하다.

```cpp
void	ft_pixel_set_axis(t_scene *scene)
{
	double			angle;
	int				x;
	int				y;
	int				l;

	if (scene->w > scene->h)
		angle = scene->camera_list->camera->fov / 180.0 / scene->w * M_PI;
	else
		angle = scene->camera_list->camera->fov / 180.0 / scene->h * M_PI;
	l = 0;
	y = -scene->h * 0.5;
	while (y < scene->h * 0.5 + scene->h % 2)
	{
		x = -scene->w * 0.5;
		while (x < scene->w * 0.5 + scene->w % 2)
		{
			scene->pixel_q[l++] = \
				ft_quaternion_multiply(scene->camera_list->camera->q_axis, \
					ft_quaternion_rotation_xyz(\
						ft_vector_3(y * angle, x * angle, 0.0)));
			x++;
		}
		y++;
	}
}
```


상위의 결과는 간단히 보면 세계지도와도 같다고 볼 수 있다. 
세계지도 또한 구의 형태를 하나의 직사각형에 넣기 위하여 왜곡된 모습을 하고 있기에 남극과 북극에 가까울 수록 보다 큰 모습을 하고 있기 때문이다.

![[Growth Log/images/Pasted image 20260314123339.jpg]]
> *[이미지 출처 : [Wiki](https://en.wikipedia.org/wiki/Interruption_(map_projection))]*

실제 많은 이들이 이런 왜곡에 대하여 고민을 많이하고 사람의 눈또한 왜곡된 모습을 보기 편한 식으로 뇌가 필터링 하여 보여주고 있기 때문에 오히려 이러한 변환 결과가 실제고 왜곡된 모습이 가짜라 생각하는 경우가 많다. 🤔

| 실제 이미지 | 원하는 이미지 |
| --- | --- |
| ![[Growth Log/images/Pasted image 20260314123532.jpg]] | ![[Growth Log/images/Pasted image 20260314123602.jpg]]|
> *[이미지 출처 : [Nvidia](https://docs.nvidia.com/vpi/2.0/algo_ldc.html)]*

상위에 대한 이미지를 보면 왜곡된 모습 (Lens Distortion Correction)에 대하여 이질감을 느끼기 때문에 왜곡을 펴주는 방식 또한 일부 픽셀을 포기하며 늘리는 방식을 채택하게 되었을 거라 생각한다.

이로서 이러한 방법의 장점으로 다음과 같이 볼 수 있을거 같다.

| 이점 | 내용 |
| --- | --- |
| FOV | 180도 이상의 FOV를 구현 할 수 있다.|
| 짐벌 락(Gimbal Lock) | 축이 맞물리며 왜곡이 시작되는 것 (모델보단 쿼터니언을 사용하여 나오는 이점으로 생각된다) |
| 연산 효율성 |  카메라 시점 변화 시, 미리 계산된 픽셀 쿼터니언에 현재 시점 쿼터니언을 곱하는 것만으로 레이(Ray) 재구성이 가능함. |


상세한 코드는 다음 레포지토리에서 확인이 가능하다

[![minirt](https://opengraph.githubassets.com/1/liebespaar93/minirt)](https://github.com/liebespaar93/minirt)

---

## Reference

- wiki : https://en.wikipedia.org/wiki/Pinhole_camera_model
- wiki : https://en.wikipedia.org/wiki/Interruption_(map_projection)
- github : https://github.com/liebespaar93/minirt
- youtube : https://www.youtube.com/watch?v=CqWzHQ5gjAs
- nvidia : https://docs.nvidia.com/vpi/2.0/algo_ldc.html