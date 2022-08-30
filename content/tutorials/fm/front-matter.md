---
title: "一份使用手册（CV战士用"
subtitle: "复制自官网粘贴，自己写点中文注释"
description: 记录自己不理解的内容、格式。
draft: false #是否为草稿，false则不是草稿、会发布
slug: fm     #文章别名，用来做永久链接，下方会详细说明
---
## front-matter
Name|Description|Notes
|---|---|---|
title|*|string
linkTitle|*|string
subtitle|displayed below the title|string, Markdown supported
date|*|string
lastmod|*|string
publishDate发布时间（--buildFuture）|*|string
expiryDate//过期时间（--buildExpired）|*|string
<taxonomies> eg: categories, tags, series|*|array
description|*|string, Markdown supported
summary|*|string, Markdown supported
images|*|array
slug|*|string
url|*|string
draft草稿（--buildDrafts\|-D可见）|*|boolean
isCJKLanguage|*|boolean
weight|*|integer
type|*|string, if equal to "poetry", will use a special layout for it
layout|*|string
outputs|*|array
aliases|*|array
markup|*|string
meta|set false to disable post-meta|boolean
toc|display TOC|boolean, override enableTOC in config.toml
tocNum|display TOC number|boolean, override displayTOCNum in config.toml
anchor|enable headings anchor|boolean, override enableHeadingsAnchor in config.toml
displayCopyright|display post-copyright|boolean, override displayPostCopyright in config.toml
badge|display updated-badge|boolean, override displayUpdatedBadge in config.toml
gitinfo|display post-gitinfo|boolean, override displayPostGitInfo in config.toml
share|display post-share|boolean, override displayPostShare in config.toml
related|display related-posts|boolean, override displayRelatedPosts in config.toml
katex|add KaTeX support|boolean, override enableKaTeX in config.toml
mathjax|add MathJax support|boolean, override enableMathJax in config.toml
mermaid|add Mermaid support|boolean, override enableMermaid in config.toml
comments|set false to disable comments in mainSections or set true to enable comments in non-mainSections|boolean
smallCaps|small caps?|boolean, override enableSmallCaps in config.toml
dropCap|drop cap?|boolean, override enableDropCap in config.toml
dropCapAfterHr|drop cap after every horizontal rule tag?|boolean, override enableDropCapAfterHr in config.toml
deleteHrBeforeDropCap|delete horizontal rule tag before drop cap?|boolean, override deleteHrBeforeDropCap in config.toml
indent|indent instead of margin?|boolean, override paragraphStyle in config.toml
indentFirstParagraph|indent the first paragraph?|boolean, override indentFirstParagraph in config.toml
align|normal, justify, center|string, if equal to "normal", will override enableJustify in config.toml
original|original? You can add the following 8 terms if you set false. The author is required, other optional|boolean, override original in config.toml
author|author of original post|string
link|link of original post|string, URL
copyright|license of the post|string, Markdown supported
website|author’s website|string
email|author’s email|string
motto|author’s description|string
avatar|author’s avatar|string, URL
twitter|author’s twitter id|string
disqus_url|*|string, if not set, will use Permalink as default
disqus_identifier|*|string, if not set, will use RelPermalink as default
disqus_title|*|string, if not set, will use Title as default

## 热重载支持部分（关hugo server --watch=false)
```
/static/*
/content/*
/data/*
/i18n/*
/layouts/*
/themes/<CURRENT-THEME>/*
config
```
Hugo 在模板中关闭 </body > 之前注入 LiveRelload < script > ，因此，如果没有这个标记，就不能工作。（鸡饭
## 
待续
