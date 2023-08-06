# execute as 이해하기
1. 순차적 실행이란?
2. execute에서의 순차적 실행
3. 결론

## 순차적 실행이란?
보통 커맨드를 다룰 때 순차의 개념은 필수적이다

```mcfunction
tp @p ~ ~100 ~
scoreboard players add @p asdf 1
```
위와 같은 명령어가 있다고 해보자.   
그러면 우린 가장 가까운 플레이어가 tp되고나서 가장 가까운 플레이어의 스코어가 오를 것이라 예측할 수 있다.   
이때 tp의 @p와 scoreboard의 @p는 다른 플레이어를 가리킬 수 있다   
하지만 두 명령어의 순서를 바꾼다면?
```mcfunction
scoreboard players add @p asdf 1
tp @p ~ ~100 ~
```
scoreboard 명령어는 위치에 영향을 주지 않기에 scoreboard의 @p와 tp의 @p는 같을 수 밖에 없다   
이것이 "순차적 실행"이라는 개념이다


## execute에서의 순차적 실행행
1번 명령어
```mcfunction
execute as @p at @s as @e[type=item] run setblock ~ ~ ~ stone
```
2번 명령어
```mcfunction
execute as @p as @e[type=item] at @s run setblock ~ ~ ~ stone
```
1번 명령어를 실행시키면 플레이어(@p)의 위치에 돌이 설치될 것이다.   
하지만 2번 명령어처럼 at @s를 옮겨버리면 아이템(@e[type=item])의 위치에 돌이 설치될 것이다.   
@s(실행자)가 바뀌었기 때문이다

1번 명령어는 {플레이어 선택 -> 실행자(플레이어)의 위치로 이동 -> 아이템 선택 -> 돌 설치}의 과정을 따르지만   
2번 명령어는 {플레이어 선택 -> 아이템 선택 -> 실행자(아이템)의 위치로 이동 -> 돌 설치}의 과정을 따르기 때문이다

이처럼 execute라는 한 줄의 명령어 내에서도 순차적 실행이라는 개념은 적용된다

이를 몰라서 발생하는 대표적인 오류는 다음 명령어를 살펴보면 바로 알 수 있다.
```mcfunction
execute as @a[tag=player] at @a[tag=player] run tp @s ~ ~10 ~
```
얼핏보면 player 태그를 가진 모든 플레이어를 각자의 위치에서 10칸 올리는 것 같지만, 실상은 그렇지 않다.   
지금부터 하나하나 뜯어보자​
```mcfunction
execute as @a[tag=player]
```
실행자를 player태그를 가진 모든 플레이어로 정했다.   
하지만, ```as @a[tag=player]```로 선택할 때, 해당 조건에 부합하는 플레이어 ```A, B, C```가 있다고 생각해보자   
들어온 순서는 ```A->B->C``` 순이다

<img src="https://cdn.discordapp.com/attachments/1132411312790577262/1137658882005680188/ABC.png" width="30%" height="30%"/>
마인크래프트는 기본적으로 플레이어를 들어온 순서로 정렬시키기 때문에 ```@a[tag=player] = {A, B, C}```가 될 것이다

그러므로 가장 먼저 @a[tag=player]가 A일 때 run까지의 명령어를 실행하고, B일 때 run까지의 명령어를 실행한 뒤에야 C가 실행자일때 run까지 명령어를 실행시킬 것이다.
```
execute as @a[tag=player] at @a[tag=player] run tp @s ~ ~10 ~
```
```
A: execute as @s at @a[tag=player] run tp @s ~ ~10 ~
B: execute as @s at @a[tag=player] run tp @s ~ ~10 ~
C: execute as @s at @a[tag=player] run tp @s ~ ~10 ~
````
이 두가지 경우가 같다는 것이다.

그럼, 가장 먼저 들어온 A가 실행자일 때라고 가정하자

```at @a[tag=player]``` 또한 {A, B, C}이므로, A는 A의 위치에서 10칸 위로 tp되고, B의 위치에서 10칸 위 장소로 tp되고, 마지막으로 C의 위치에서 10칸 위 장소로 tp될 것이다.

<img src="https://cdn.discordapp.com/attachments/1132411312790577262/1137659792794591292/ABC.png" width="30%" height="30%"/> -> <img src="https://cdn.discordapp.com/attachments/1132411312790577262/1137659826458066974/ABC.png" width="30%" height="30%"/> -> <img src="https://cdn.discordapp.com/attachments/1132411312790577262/1137659873740476488/ABC.png" width="30%" height="30%"/>

결과적으론 A는 C의 10칸 위로 tp 되었다.   
B도 마찬가지로 A와 같은 위치로 tp된다.   
C의 경우에도 자신의 10칸 위로 tp된다.

따라서 이 명령어는 모든 플레이어를 각자의 위치를 기준으로 10칸 위로 tp시키는 것이 아닌,   
모든 플레이어를 C의 위치를 기준으로 10칸 위로 tp시키는 명령어인 것이다.   
<img src="https://cdn.discordapp.com/attachments/1132411312790577262/1137661381685026907/ABC.png" width="30%" height="30%"/>​

그렇다면 각자의 위치를 기준으로 10칸 위로 tp시키기 위해선 어떻게 해야할까?
```mcfunction
execute as @a[tag=player] at @a[tag=player] run tp @s ~ ~10 ~
```
```mcfunction
execute as @a[tag=player] at @s run tp @s ~ ~10 ~
```
두 명령어의 차이가 보이는가?   
```at @a[tag=player]```를 ```at @s```로 바꾸었다.   
이렇게 된다면 as에서 A가 실행자일 때, @s는 A를 가리키므로 A는 A의 위치를 기준으로 10칸 위로 tp될 것이다.   
B, C도 마찬가지다.

## 결론
이처럼 execute도 순차적으로 실행됨을 알지 못하면 커맨드를 만지는 것에 있어 오류는 필연적이다   
항상 주의하자
