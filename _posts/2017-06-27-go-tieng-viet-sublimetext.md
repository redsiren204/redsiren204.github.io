---
layout: post
title: "Gõ tiếng Việt trong Sublime Text"
date: 2017-06-27
excerpt: "Cách gõ tiếng Việt trong Sublime Text"
tags: [sample post, images, test]
comments: true
---

# Vietnamease IME - Bộ gõ tiếng Việt

Gõ tiếng Việt trong Sublime Text
Phiên bản có 2 nhánh cho ki ểu gõ VNI và TELEX được lưu lại tại branch VN_IME và branch TELEX.

## Cài đặt

Package Control: Install Package -> Vn Ime
Cài đặt bằng tay:
* Trên Sublime Text, chọn Preferences -> Browse Packages để vào thư mục chứa Package
* Mở Terminal và chạy dòng lệnh sau để clone Vietnamease IME:
{% highlight sh %}
$ git clone https://github.com/yehnkay/VN_IME
{% endhighlight %}

## Hướng dẫn sử dụng

Nhấn phím F2 để bật/tắt gõ tiếng Việt, mặc định là kiểu VNI.
Khi thanh status hiện chữ VN IME : ON là đang bật, VN IME: OFF là đang tắt.

Để dùng kỉêu gõ TELEX, thêm giá trị telex trong tập tin cấu hình Preferences.sublime-settings tại Preferences -> Settings - User với giá trị true, như sau:
{% highlight c %}
{
	"telex": true
}
{% endhighlight %}
Hoặc đến thư mục chứa VN_IME và checkout branch TELEX.
VD:
{% highlight sh %}
$ cd {$HOME}/.config/sublime-text-3/Packages/VN_IME/
$ checkout branch TELEX
{% endhighlight %}