# 위치
마인크래프트에서 위치를 구성하는 요소는 다음과 같습니다.
- 차원(`Dimension`)
- 좌표(`Pos`)
- 회전(`Rotation`)

## 1. 차원
차원은 기본적으로 세 종류가 존재합니다.
- 오버월드(`minecraft:overworld`)
- 네더(`minecraft:the_nether`)
- 엔드(`minecraft:the_end`)

이외에도 데이터팩, 플러그인, 모드 등으로 인해 차원이 추가될 수 있습니다.

## 2. 좌표
마인크래프트에서는 3차원 좌표계를 사용합니다.
따라서 좌표는 다음과 같이 구성됩니다.
- `x`
- `y`
- `z`

각 좌표는 `double`자료형입니다.

## 3. 회전
회전은 두개의 `float`값으로 구성됩니다.
- yaw(`y_rotation`)
- pitch(`x_rotation`)

yaw가 가로축 회전값이며, pitch가 세로축 회전값입니다.
