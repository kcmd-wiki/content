
# 문자열 합치기

<hr>

문자열 여럿이 주어졌을 때 문자열을 합치는 기술입니다.

동적 명령어 실행이나 동적 JSON 출력에 활용이 가능합니다.

<br>

## 1. 인첸트 버그 이용

<hr>

인첸트 명령어를 대상에게 사용시, 대상의 이름이 flatten되어 출력되는 버그를 이용합니다. ([MC-170355](https://bugs.mojang.com/browse/MC-170355))

인첸트를 이용하여 문자열을 합치는 기술은 문자열 합치기 기술 중 비교적 쉬운 편입니다.

과정은 다음과 같습니다.

1. 합치고자 하는 문자열을 리스트에 담아 준비
2. 리스트를 표지판에 `"interpret":"true"`로 복사
3. 표지판 내용을 마커(엔티티)의 `CustomName`에 복사
4. 인첸트 버그를 사용하기 위해 마커를 인첸트
5. 인첸트를 한 커맨드블록의 출력에서 합쳐진 문자열을 복사

```mcfunction
# load
forceload add 0 0
summon marker 0. 0 0. {UUID:[I;0,0,0,0]}
setblock 7 -60 176 oak_sign
```
```mcfunction
# command block
data modify storage str: input set value ["aaa","bbb","ccc"]
data modify block 7 -60 176 front_text.messages[0] set value '{"storage":"str:","nbt":"input","interpret":true}'
data modify entity 0-0-0-0-0 CustomName set from block 7 -60 176 front_text.messages[0]
enchant 0-0-0-0-0 lure
data modify storage str: output set string block 7 -60 181 LastOutput 89 -38
tellraw @a {"storage":"str:","nbt":"output"}
```

※ 이 방법은 커맨드블록 사용이 필수입니다.

<br>

## 2. 독서대 + 호버이벤트

독서대의 책 내용이 Lenient(느슨한?) 문법을 지원한다는 점과

호버이벤트의 `value`에서 `contents`로 변환되는 과정에 문자열이 합쳐진다는 점을 이용합니다.

과정은 다음과 같습니다.






