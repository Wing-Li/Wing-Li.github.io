Github:https://github.com/arkadianriver/arkadianriver.com
Url:http://arkadianriver.com/index.html

### 0. 安装 Jekyll (version 3.1.2 or higher).

### 1.调整网站，使其成为你自己的。Jekyll 使用 [YAML文件](http://www.yaml.org/start.html) 作为变量:

a. 编辑 _config.yml 文件，用您的信息替换每个键的值。

b.  _data/tokens.yml 用你的ID和邮件程序添加一个文件。可以看这个例子： _data/tokens-template.yml 。.

c. 为自己添加作者信息 _data/authors.yml 作为文件中的第一个作者条目。

d. 提供您自己的 images 。

e. 继续按照你的想法去调整，当然，也可以不用调整。

### 2. 创建您的帖子：

a. 使用 posts in the 31st century 作为你的指南。 只有在使用该--future选项时，它们才由jekyll构建。

b. 您可以运行ruby compose.rb以创建新的草稿文章。

### 3. 测试并发布您的网站：

If you're building your site on Windows (like me) and you use WinSCP to sync with your remote site, you can use the site.bat file. Set up a _site.env file as described in the comments of site.bat and change the excludes list for your site.
如果您在Windows上构建您的站点（如我），并使用WinSCP与远程站点进行同步，则可以使用该site.bat文件。_site.env按照评论中的描述设置文件，site.bat并更改您网站的排除列表。

site dev runs jekyll serve --future --drafts in development mode.
site devnof runs jekyll serve --drafts in development mode.
site preview runs jekyll serve in production mode.
site prod simply builds with jekyll build in production mode (no serve).
site publish uses WinSCP's synchronize feature to mirror to a remote site.