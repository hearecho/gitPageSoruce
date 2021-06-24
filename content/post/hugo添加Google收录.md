---
title: "Hugo添加Google收录"
date: 2021-06-24T16:51:49+08:00
draft: false
description: "hugo将网站添加到google收录中，使得网站可以被用户搜索到"
tags:
- Hugo
- Google Console
- 杂记
categories:
- blog
---

### 添加谷歌收录

添加谷歌收录的方式主要有四种方式：

1. Google Analytics
2. HTML file
3. HTML tag
4. Google Tag Manager
5. Domain name provider

最简单的方式就是使用HTML TAG直接在模板`head`标签里面加上网站给出的验证标签即可。

### google 分析

添加谷歌分析，可以获知自己网站的各项数据。同时也可以用于上述使得网站被谷歌收录。

#### 注册Google 分析

> 1. 打开[Google Analytics官网](https://analytics.google.com/analytics/web/)注册账户并添加自己的网站域名
> 2. 打开主页，添加数据流，之后记录衡量ID。

#### 修改配置文件

在config.toml中新建googleAnalytics参数并设置成自己的衡量ID

```toml
googleAnalytics = "xx-xxxxxxxxx-x" # Enable Google Analytics by entering your tracking id
```

#### 新建模板

在Hugo站点根目录下新建模板文件(./layouts/_internal/google_analytics_async.html)并添加如下代码.

```html
<!-- Global Site Tag (gtag.js) - Google Analytics -->
<script async src="https://www.googletagmanager.com/gtag/js?id={{ .Site.GoogleAnalytics }}"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());

  gtag('config', '{{ .Site.GoogleAnalytics }}');
</script>
```

#### 引用模板

在baseof.html基础模板文件中的head标签尾部添加如下代码, 这样站点发布到非Hugo Server后就会自动引用Google Analytics模板.或者也可以将上述的模板内容直接粘贴复制到baseof.html相应的位置。

```html
<head>
{{- if not .Site.IsServer }}
	{{ template "_internal/google_analytics_async.html" . }}
{{- end }}
</head>
```
### 每日一题
#### 下一个排列
实现获取 下一个排列 的函数，算法需要将给定数字序列重新排列成字典序中下一个更大的排列。如果不存在下一个更大的排列，则将数字重新排列成最小的排列（即升序排列）。必须 原地 修改，只允许使用额外常数空间。

> 目的是寻求按照字典序来说，下一个排列，当然可以找到全部排列，但是不太靠谱。我们可以自己通过该排列的顺序，找到下一个排列。
>
> 例如：
>
> [1,2,3]
>
> [1,3,2]
>
> [2,1,3]
>
> [2,3,1]
>
> [3,1,2]
>
> [3,2,1]
>
> 所以我们的目标是将左边一个较小的数和右边的一个较大的数进行交换，如此下一个排列才会更大，同时由于不能打太多。同时我们要让这个「较小数」尽量靠右，而「较大数」尽可能小。当交换完成后，「较大数」右边的数需要按照升序重新排列。这样可以在保证新排列大于原来排列的情况下，使变大的幅度尽可能小。所以可以使用两次排序来确定两个数字的位置，然后进行交换.
>
> 以排列 [4,5,2,6,3,1][4,5,2,6,3,1] 为例：
>
> 我们能找到的符合条件的一对「较小数」与「较大数」的组合为 2 与 3，满足「较小数」尽量靠右，而「较大数」尽可能小。
>
> 当我们完成交换后排列变为 [4,5,3,6,2,1][4,5,3,6,2,1]，此时我们可以重排「较小数」右边的序列，序列变为 [4,5,3,1,2,6][4,5,3,1,2,6]。
>

```go
func nextPermutation(nums []int) {
    // 先找到一个最小数，然后找到比这个较小数稍微大的较大数
	n := len(nums)
	i := n - 2
    //找到此时分割左右边的位置
    // 也就是较小数尽可能的小
	for i >= 0 && nums[i] >= nums[i+1] {
		i--
	}
	
	if i >= 0 {
		j := n - 1
        //找到较小数
		for j >= 0 && nums[i] >= nums[j] {
			j--
		}
        //交换
		nums[i], nums[j] = nums[j], nums[i]
	}
    //之后反转右边的序列
	reverse(nums[i+1:])
}

func reverse(a []int) {
	for i, n := 0, len(a); i < n/2; i++ {
		a[i], a[n-1-i] = a[n-1-i], a[i]
	}
}
```

