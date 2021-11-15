# Blog

![](https://api.travis-ci.com/NatLee/Blog.svg)

Just a blog.

## Process

```bash
git clone https://github.com/theme-next/hexo-theme-next.git themes/next
git clone https://github.com/theme-next/theme-next-pjax themes/next/source/lib/pjax
```

## Modification

- Modify `themes/next/_config.yml`

- Add LikeCoin block

    Add below code after `post.content` in `themes/next/layout/_macro/post.swig`.

        ```html
        <div>
            <script type="text/javascript">
                document.write(
                "<iframe scrolling='no' frameborder='0' sandbox='allow-scripts allow-same-origin allow-popups allow-popups-to-escape-sandbox allow-storage-access-by-user-activation' style='height: 212px; width: 100%;' src='https://button.like.co/in/embed/natlee_/button?referrer=" + encodeURIComponent(location.href.split("?")[0].split("#")[0]) + "'></iframe>");
            </script>
        <div>
        ```

