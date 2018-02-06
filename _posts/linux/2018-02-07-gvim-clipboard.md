---
layout: post
title:  'gvim 윈도우 클립보드 연동'
date:   2018-02-07 06:50 +0900
tags: gvim 
---

윈도우나 x11에서 gvim을 사용할 때 가장 아쉬운 건 delete, yank, put 할 때 시스템 clipboard를 사용하려면 `"*`이나 `"+`로 항상 레지스터를 지정 하줘야 한다.

# ctrl+x,c,v 맵핑
윈도우용 vim을 설치하면 mswin.vim 파일에 `ctrl+x,c,v`를 `"+x`,`"+y`,`"+p` 로 map 해주는 부분이 있다. 거기서 필요한 부분만 때서 개인 설정 파일에 붙여 넣어 주는 방법도 있고

# unnamed register를 시스템 클립보드와 연동
vim에서 틀별히 어떤 register를 사용할지 정하지 않고 delete, yank, put을 하면 unnamed register(`"`)가 사용된다. 

아래와 같은 옵션을 사용하면 delete, yank, put이 기본을 `+`, `*`을 사용하도록 바꿀수 있다.

```
if has("clipboard")
	set clipboard^=unnamed,unnamedplus
endif
```
