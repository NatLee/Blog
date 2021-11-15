# Blog

![](https://api.travis-ci.com/NatLee/Blog.svg)

Just a blog.

## Modification

- Modify `themes/next/_config.yml`

- Add LikeCoin
    
    - Create folder `_custom` and add file `like_coin.ejs`

        Content of `themes/next/layout/_custom/like_coin.ejs`

        ```html
        <div>
            <script type="text/javascript">
                document.write(
                "<iframe scrolling='no' frameborder='0' sandbox='allow-scripts allow-same-origin allow-popups allow-popups-to-escape-sandbox allow-storage-access-by-user-activation' style='height: 212px; width: 100%;' src='https://button.like.co/in/embed/natlee_/button?referrer=" + encodeURIComponent(location.href.split("?")[0].split("#")[0]) + "'></iframe>");
            </script>
        <div>
        ```

    - Modify `themes/next/layout/_macro/post.swig` with adding `{% include '../_custom/like_coin.ejs' %}` after `post.content`.

