---
layout: post
title: "Cross compiler"
date: 2017-06-27
excerpt: "Tìm hiểu về Cross compiler / Toolchain"
tags: [sample post, images, test]
comments: true
---

### Định nghĩa trình biên dịch

> Trình biên dịch, còn gọi là phần mềm biên dịch, compiler, là một chương trình máy tính làm công việc dịch một chuỗi các câu lệnh được viết bằng một ngôn ngữ lập trình (gọi là ngôn ngữ nguồn hay mã nguồn), thành một chương trình tương đương nhưng ở dưới dạng một ngôn ngữ máy tính mới (gọi là ngôn ngữ đích) và thường là ngôn ngữ ở cấp thấp hơn, như ngôn ngữ máy. Chương trình mới được dịch này gọi mã đối tượng.

### Quá trình biên dịch

Ví dụ có chương trình sau được lưu dưới tên mytest.c
{% highlight c %}
#include <stdio.h>
#include <math.h>
#define PI 3.14
int main(int argc, char **argv)
{
    float x, y;
    x = PI / 3;
    y = cos(x);
    printf("Value cos(%f) = %f \n", x, y);
    return 1;
}
{% endhighlight %}
Trong chương trình có sử dụng thư viện math.h. Trong quá trình linker sẽ tiến hành lấy mã của hàm printf() trong thư viện để kết hợp với mã thông thường khác.
<figure>
	<a href="https://raw.githubusercontent.com/redsiren204/redsiren204.github.io/master/resources/cross-compiler/CompilationProcess.png"><img src="https://raw.githubusercontent.com/redsiren204/redsiren204.github.io/master/resources/cross-compiler/CompilationProcess.png"></a>
	<figcaption><a href="https://raw.githubusercontent.com/redsiren204/redsiren204.github.io/master/resources/cross-compiler/CompilationProcess.png" title="Quá trình biên dịch">Quá trình biên dịch</a>.</figcaption>
</figure>

### Cross compiler / Toolchain là gì?

Có thể hiểu Cross compiler hay còn gọi là Toolchain như sau:
Một source code được viết trên máy tính chạy chíp Intel (x86 platform), thông qua một cross compiler sẽ cho ra file nhị phân (mã máy) có khả năng chạy được trên một nên tảng chip khác là ARM (ARM platform). Trên hình native compiler là trình biên dịch để tạo ra file nhị phân chạy trên chính máy tính đang dùng để viết source code.
<figure>
	<a href="https://raw.githubusercontent.com/redsiren204/redsiren204.github.io/master/resources/cross-compiler/cross-compile.png"><img src="https://raw.githubusercontent.com/redsiren204/redsiren204.github.io/master/resources/cross-compiler/cross-compile.png"></a>
</figure>

### Hệ thống Cross compiler

Quá trình tạo và sử dụng Toolchain gồm 3 thành phần:
<figure>
	<a href="https://raw.githubusercontent.com/redsiren204/redsiren204.github.io/master/resources/cross-compiler/build-gcc-cross-compiler.png"><img src="https://raw.githubusercontent.com/redsiren204/redsiren204.github.io/master/resources/cross-compiler/build-gcc-cross-compiler.png"></a>
</figure>
* Build System: Hệ thống/platform tạo ra toolchain
* Host System: Hệ thống chạy chương trình toolchain để compile source của chương trình ứng dụng.
* Target: Là hệ thống chạy các chương trình do hệ thống host tạo ra.

### Cách sử dụng cross compiler trên một hệ thống build với poky

* Chỉnh sửa sdk, thêm các gói cần thiết dùng để compile source. Ví dụ:
{% highlight c %}
$ {path}/package-core-standalone-sdk-target.bbappend
RDEPENDS_${PN} += "\
linux-libc-headers-base-dev \
tzcslib-dev \
libtzcs-security-dev \
gtk+-dev \
"
{% endhighlight %}
* Build meta-toolchain sau khi chỉnh sửa sdk bằng poky:
{% highlight c %}
$ bitbake meta-toolchain
{% endhighlight %}
* Copy sdk sang máy host để cài đặt:
{% highlight c %}
$ cp debian-eglibc-x86_64-meta-toolchain-x86_64-toolchain-8.0.sh <host_machine>
{% endhighlight %}
* Run script để cài đặt toolchain trên máy host. Script này sẽ set lại biến PATH để sử dụng build source
{% highlight c %}
$ source /opt/debian/8.0/environment-setup-x86_64-debian-linux
{% endhighlight %}
* Cuối cùng sử dụng ${CC} thay vì gcc trên máy host để compiler source:
{% highlight c %}
$ ${CC} -g abc.c -o  -lpthr -lX11 -ltzcs -ltzcs-s `pkg-config --cflags --libs gtk+-2.0`
{% endhighlight %}