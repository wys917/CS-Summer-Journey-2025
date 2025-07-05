# 使用git的工作流程

**1.在main分支里检查是否有更新**     `git checkout main`

**2.拉取更新**     `git pull origin main`

**3.创建并切换到新分支**			`git checkout -b （your new branch)`

**4.在新分支上进行你想要的操作**

**5.把所有改动先加到“待提交区”**      `git add` .

**6.提交，后面跟上简短的说明**     `git commit -m "whatever you want to note"`

> 关于说明,通常有以下几种规范：
>
> `feat: Add iterative solution for LeetCode 94` (feat = 新功能)
>
> `fix: Correct variable scope error in maxDepth function` (fix = 修复bug)
>
> `docs: Update README with repository navigation` (docs = 只改了文档)
>
> `style: Format C code according to Google Style Guide` (style = 代码格式)
>
> `refactor: Improve directory structure for better organization` (refactor = 重构)

**7.把结果推送到云端**				`git push origin main`

**8.一键部署网站**                    `mkdocs gh-deploy`
