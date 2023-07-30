
# 문자열 합치기

문자열 여럿이 주어졌을 때 문자열을 합치는 기술입니다.

예를 들어 저장소에 저장된 두 문자열을 하나로 합쳐서 다시 저장소에 저장할 수 있습니다. 
```
{input: ["abc", "def"]}
 >>> {output: "abcdef"}
```
동적 명령어 실행이나 동적 JSON 출력에 활용이 가능합니다.

## 1. `/enchant` 버그 이용

`enchant` 명령어를 대상에게 사용시, 대상의 이름이 flatten되어 출력되는 버그를 이용합니다. ([MC-170355](https://bugs.mojang.com/browse/MC-170355))

문자열 합치기 기술 중 비교적 쉽고 간단한 편이지만, 대신 반드시 커맨드블록을 이용해야 한다는 단점이 존재합니다.

### 기술 설명

마커 하나를 다음과 같이 복잡한 이름을 지정하여 소환해봅니다.
```mcfunction
summon marker ~ ~ ~ {CustomName:'["a","b","c","d","e"]'}
```
그리고 커맨드블록을 통해 이 마커에 `enchant` 명령어를 써봅니다.
```mcfunction
enchant @e[name=abcde,limit=1] lure
```
그리고 해당 커맨드블록의 출력을 `data get` 명령어로 확인해보면 이름이 합쳐져 있는것을 확인할 수 있습니다.
```mcfunction
data get block <XYZ> LastOutput
```
`data modify ... set string` 명령어를 이용해 이곳의 문자열을 추출하면, 합쳐진 문자열을 최종적으로 얻을수 있습니다.

이 버그를 이용하여 다음과 같은 과정으로 문자열 합치기가 가능합니다.

1. 합치고자 하는 문자열을 리스트에 담아 준비
2. 리스트를 표지판에 `"interpret":"true"`로 복사
3. 표지판 내용을 마커(엔티티)의 `CustomName`에 복사
4. 커맨드블록에서 마커를 대상으로 `enchant`
5. 커맨드블록의 출력에서 합쳐진 문자열을 복사

### 구현

```mcfunction
# load
forceload add 0 0
summon marker 0. 0 0. {UUID:[I;0,0,0,0]}
setblock <표지판XYZ> oak_sign
```

```mcfunction
# command block
data modify storage str: input set value ["aaa","bbb","ccc"]
data modify block <표지판XYZ> front_text.messages[0] set value '{"storage":"str:","nbt":"input","interpret":true}'
data modify entity 0-0-0-0-0 CustomName set from block <표지판XYZ> front_text.messages[0]
enchant 0-0-0-0-0 lure
data modify storage str: output set string block 7 -60 181 LastOutput 89 -38
tellraw @a {"storage":"str:","nbt":"output"}
```

### 주의사항

기술 특성상 반드시 커맨드블록의 출력에 접근해야 하기에 반드시 커맨드블록을 이용해야 한다는 단점이 존재합니다.

큰따옴표(`"`) 기호가 들어가면 고장나며,
역슬래시(`\`) 기호가 들어가면 역슬래시 두개로 불어납니다.
커맨드블록의 출력이 JSON 텍스트이기에 이 두 기호는 아주 불안정합니다.
반대로 작은따옴표 기호는(`'`) 아무런 부작용 없이 멀쩡하니 가능하다면 이것을 이용하세요.

## 2. 독서대 + 호버이벤트 이용

표지판에서 리스트 전체 선택 경로(`list[]`)를 사용하면 콤마가 삽입되며 문자열이 합쳐진다는 점과

독서대의 책 내용이 JSON 문법의 Lenient(느슨한?) 모드를 지원한다는 점과

호버이벤트에서 `value`를 쓸 시 `contents`로 변환되는 과정에 문자열이 합쳐진다는 점을 이용합니다.

커맨드블록이 필요없기에 함수에서도 활용이 가능합니다.

### 기술 설명

우선 표지판의 문자열 합치기 기능을 소개합니다. 다음과 같은 리스트를 준비하고
```
{list: ["a", "b", "c", "d", "e"]}
```
다음 JSON을 이용하여 위 리스트를 표지판에 불러오면 표지판은 이걸 콤마가 삽입되어있는 합쳐진 JSON으로 변환합니다.
```
'{"storage":"...","nbt":"list[]"}'
 >>> '{"text":"a, b, c, d, e"}'
```

<hr>

그 다음, 호버이벤트의 문자열 합치기 기능을 소개합니다. 과거의 문법인 `values`를 이용할 때 문자열 대신 리스트를 넣을수가 있는데
```
'{"text":"a","hoverEvent":{"action":"show_item","value":"{tag:{asdf:1b}}"}}'
'{"text":"a","hoverEvent":{"action":"show_item","value":["{tag:{asdf:1b}}"]}}'
 >>> '{"text":"a","hoverEvent":{"action":"show_item","contents":{"id":"minecraft:air","count":0,"tag":"{asdf:1b}"}}}'
```
리스트 안에 여러 문자열이 들어있을때 호버이벤트는 모든 문자열을 합친 결과를 하나의 NBT로 보고 `contents`로 변환합니다.
이 과정에서 문자열이 합쳐집니다.
```
'{"text":"a","hoverEvent":{"action":"show_item","value":["{tag:{asdf:","a","b","c","d","e","}}"]}}'
 >>> '{"text":"a","hoverEvent":{"action":"show_item","contents":{"id":"minecraft:air","count":0,"tag":"{asdf:\\"abcde\\"}"}}}'
```

<hr>

마지막으로, 독서대의 Lenient(느슨한?) 문법을 소개합니다.
독서대 위에 놓여있는 글이 쓰인 책(`written_book`)에서는 일부 문법이 틀린 JSON 표현을 받아들여, 올바른 문법으로 교정까지 해줍니다.
```
"{text:hello,color:'gold'}"
 >>> '{"text":"hello","color":"gold"}'
```
교정되는 문법의 종류는 [이곳](https://www.javadoc.io/doc/com.google.code.gson/gson/2.8.3/com/google/gson/stream/JsonReader.html#setLenient-boolean-)에서 확인할 수 있습니다.
정확히는 독서대가 아닌 책이 resolve 하는거라 손에 들고있는 책으로도 이 현상을 일으킬 수 있습니다만,
책을 자동으로 resolve하는 블록은 독서대뿐인지라 이 기술은 독서대를 이용한다고 보통은 말합니다.

<hr>

위에서 설명한 세가지 기술을 이용하여 다음과 같은 과정으로 문자열 합치기가 가능합니다.

1. 합치고자 하는 문자열을 리스트에 담아 준비
2. 문법이 불완전한 호버이벤트 JSON 준비 (`"{text:a,hoverEvent:{action:show_item,value:['{tag:{d:\"','\"}}']}}"`)
3. 표지판 콤마 합치기를 이용해 리스트의 모든 항목이 호버이벤트의 `hoverEvent.action.value[0]`과 `hoverEvent.action.value[1]` 사이에 들어오도록 유도
4. 완성된 호버이벤트를 독서대로 복사하고 독서대 resolve 발동
5. 독서대의 데이터로부터 합쳐진 문자열을 복사

### 구현

```mcfunction
# load
setblock <표지판XYZ> oak_sign
setblock <독서대XYZ> air
setblock <독서대XYZ> lectern{Book:{id:"minecraft:written_book",Count:1b,tag:{pages:['""'],author:-,title:-}}}
data modify storage str: booktag set value {pages:[''],resolved:0b}
```

```mcfunction
# command block / function
data modify storage str: input set value ["aaaa","bbbb","cccc"]
data modify storage str: input prepend value "{text:a,hoverEvent:{action:show_item,value:['{tag:{d:\"'"
data modify storage str: input append value "'\"}}']}}"
data modify block <표지판XYZ> front_text.messages[0] set value '{"storage":"str:","nbt":"input[]"}'
data modify storage str: booktag.pages[0] set string block <표지판XYZ> front_text.messages[0] 9 -2
data modify block <독서대XYZ> Book.tag merge from storage str: booktag
data modify storage str: output set string block <독서대XYZ> Book.tag.pages[0] 91 -18
tellraw @a {"storage":"str:","nbt":"output"}
```

### 주의사항

큰따옴표(`"`), 역슬래시(`\`), 대괄호(`[`,`]`), 중괄호(`{`,`}`), 등호(`=`), 콜론(`:`), 슬래시(`/`), 해시(`#`), 공백(스페이스바) 기호중 하나라도 들어가면 고장나며,
세미콜론(`;`)과 콤마(`,`) 기호는 자동으로 제거됩니다.
작은따옴표(`'`) 기호 또한 들어가면 고장나지만 다른 일반적인 문자와 붙어있으면 고장내지 않습니다. (`abc'abc`)

이중에서 큰따옴표(`"`), 역슬래시(`\`), 작은따옴표(`'`) 기호를 제외하면은 고장 또는 제거를 피할수 있는 방법이 존재합니다.
해당 기호가 포함되어있는 문자열을 글자단위로 분해한뒤, 해당 기호를 작은따옴표로 묶으면 됩니다.
예를 들어 다음 문자열을 합치고자 한다면
```
["hello#world","123"]
```
올바르지 않은 기호, 여기에선 해시(`#`) 기호가 들어있는 문자열을 글자단위로 분해합니다.
어느 문자열인지 감지가 안된다면 모든 문자열을 분해해도 좋으며, 최적화를 위해 분해 안해도 되는 문자열은 남겨도 좋습니다.
```
["h","e","l","l","o","#","w","o","r","l","d","123"]
```
이곳에서 공백(`"#"`) 기호를 찾아서 작은따옴표로 묶은 문자열(`"'#'"`)로 교체합니다.
```
["h","e","l","l","o","'#'","w","o","r","l","d","123"]
```
그리고 이상태에서 위 독서대 문자열 합치기 기술을 사용합니다. 그러면 정상적으로 합쳐지는것을 확인하실 수 있습니다.
```
["h","e","l","l","o","''#'","w","o","r","l","d","123"]
 >>> "hello#world123"
```

## 3. 동적 JSON

`enchant` 버그와 독서대 기술을 적절히 활용하면 동적으로 JSON 텍스트를 구성하는게 가능합니다.

예를 들어, 출력할 텍스트 색상을 스코어보드 값으로 결정할 수 있다거나,

저장소에 저장된 명령어 문자열을 클릭이벤트로 등록할 수 있습니다.

### 기술 설명

예를 들어, 동적 헥스 컬러를 만드는 것을 목표로 해봅시다.

위에서 설명한 "enchant"와 "독서대"를 알고 있다는 전제 하에 진행합니다.

다음과 같이 Lenient 문법으로 된 JSON 문자열을 `enchant`로 잘 합쳐서 만들고
```
"{text:ASDFSDAFASDFSAFD,color:'#123456'}"
```
이 문자열을 그대로 독서대에 넣으면 문법이 교정되니
```
"{text:ASDFSDAFASDFSAFD,color:'#123456'}"
 >>> '{"text":"ASDFSDAFASDFSAFD","color":"#123456"}'
```
`tellraw` 등의 명령어에서 `"interpret":true`로 출력이 가능해집니다.

클릭이벤트의 경우도 과정이 동일합니다.
`enchant`를 이용하여 Lenient 문법의 JSON 텍스트를 생성한뒤 독서대에 넣고 `"interpret":true`로 출력하면 됩니다.

### 구현

```mcfunction
# load
forceload add 0 0
summon marker 0. 0 0. {UUID:[I;0,0,0,0]}
setblock <표지판XYZ> oak_sign
setblock <독서대XYZ> air
setblock <독서대XYZ> lectern{Book:{id:"minecraft:written_book",Count:1b,tag:{pages:['""'],author:-,title:-}}}
data modify storage str: booktag set value {pages:[''],resolved:0b}
```

```mcfunction
# command block
data modify storage str: input set value ["{color:'","#","FF8000","',text:ASDFGHJKL}"]
data modify block <표지판XYZ> front_text.messages[0] set value '{"storage":"str:","nbt":"input","interpret":true}'
data modify entity 0-0-0-0-0 CustomName set from block <표지판XYZ> front_text.messages[0]
enchant 0-0-0-0-0 lure
data modify storage str: booktag.pages[0] set string block -1 -60 181 LastOutput 89 -38
data modify block <독서대XYZ> Book.tag merge from storage str: booktag
data modify storage str: output set from block <독서대XYZ> Book.tag.pages[0]
tellraw @a [{"storage":"str:","nbt":"output"}," >> ",{"storage":"str:","nbt":"output","interpret":true}]
```
