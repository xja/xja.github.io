# First_post


使用主题：[LoveIt](https://github.com/dillonzq/LoveIt)

搭建参考：[Theme Documentation - Basics](https://hugoloveit.com/theme-documentation-basics/)

```Shell
git clone https://github.com/xja/xja.github.io
hugo new site xja.github.io --force
cd xja.github.io
git submodule add https://github.com/dillonzq/LoveIt.git themes/LoveIt
hugo new posts/first_post.zh-cn.md
git add .
git commit -m "First post"
git push
```

