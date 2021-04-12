### 情况

当主分支(master)更新代码而feat分支落后于主分支时

###  步骤

- gco master —&gt;gl

- gco feat-xx —&gt;git rebase master 

- 遇到冲突

> 此时分支名变成冲突对应的commit id
>
> ![image-20210412205328234](/Users/xiaoxin/Library/Application Support/typora-user-images/image-20210412205328234.png)
>
> ![image-20210412205354597](/Users/xiaoxin/Library/Application Support/typora-user-images/image-20210412205354597.png)
>
>  

### 解决冲突

- 到编辑器解决冲突

-  解决完后git add

- 此时有两种可能的情况，对应的操作时git rebase --continue / git rebase --skip。跟着提示来就好。

  - Continue （git add 之后有modified）

    ![image-20210412220430142](/Users/xiaoxin/Library/Application Support/typora-user-images/image-20210412220430142.png)

  - skip（git add 之后没有modified，似乎是Mac os的一个bug：https://stackoverflow.com/questions/21889741/git-rebase-continue-wont-work）![image-20210412210427408](/Users/xiaoxin/Library/Application Support/typora-user-images/image-20210412210427408.png)

- 可能有几轮git rebase --contintue，直到分支名变成feat-xx

- 此时，iterm会提示你git pull,让你把远程分支合到你的本地分支（如果git pull了需要再次处理冲突），你只需要git push --force 

  ![image-20210412220744792](/Users/xiaoxin/Library/Application Support/typora-user-images/image-20210412220744792.png)

- 完成✅