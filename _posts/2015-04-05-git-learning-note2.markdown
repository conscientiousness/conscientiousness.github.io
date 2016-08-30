---
layout:     post
title:      "Git 學習筆記 - 基本操作,建立分支,合併,同步至github"
date:       2015-04-05
author:     "Jesse"
catalog:    false
tags:
    - git
---

==本機操作==
<br>
所在目錄建立repository
```
git init
```

檔案放到 staging area
```
git add 檔案名稱
```

commit到repository，使用`-m`作為備註
```
git commit -m "this is memo"
```

對專案進行分支管理
```
git branch 分支名稱
```

切換分支
```
git checkout 分支名稱
```

現在已切換到別的分支，如要把該分支多個檔案加到staging area
```
git add .
```

之後也別忘了`commit`

如要以master作為主幹，切換回master
```
git checkout master
```

合併分支
```
git merge 分支名稱
```
<br>
==將本機 repository 和 Github repository 同步==

有幾種方式
1.`SSH` 安全性佳
```
git remote add origin git@github.com:帳戶名稱/repo名稱.git
```
2.`HTTP/HTTPS`
```
git remote add origin https://github.com/帳戶名稱/repo名稱.git
```

丟commit出去
```
git push -u origin master
```

The `-u` tells Git to remember the parameters,所以之後只需
```
git push
```

將遠端的 repo. 與本地的 repo. 合併
```
 git pull origin master
```
加`--rebase`可以避免無謂的 merge 節點，特別是很多人在同一個 branch 同時開發的情況。
參考：https://ihower.tw/blog/archives/3843