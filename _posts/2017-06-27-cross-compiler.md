---
layout: post
title: "Cross compiler"
date: 2017-06-27
excerpt: "Tìm hiểu về compiler/toolchain"
tags: [sample post, images, test]
comments: true
---

#### Định nghĩa trình biên dịch
Trình biên dịch, còn gọi là phần mềm biên dịch, compiler, là một chương trình máy tính làm công việc dịch một chuỗi các câu lệnh được viết bằng một ngôn ngữ lập trình (gọi là ngôn ngữ nguồn hay mã nguồn), thành một chương trình tương đương nhưng ở dưới dạng một ngôn ngữ máy tính mới (gọi là ngôn ngữ đích) và thường là ngôn ngữ ở cấp thấp hơn, như ngôn ngữ máy. Chương trình mới được dịch này gọi mã đối tượng.
#### Qúa trình biên dịch
Ví dụ có chương trình sau được lưu dưới tên mytest.c
#include <stdio.h>
#include <math.h>
#define PI 3.14
int main(int argc, char **argv)
{
    float x, y;
    x = PI / 3;
    y = cos(x);
    printf("gia tri cos(%f) = %f \n", x, y);
    return 1;
}
Trong chương trình có sử dụng thư viện math.h. Trong quá trình linker sẽ tiến hành lấy mã của hàm printf() trong thư viện để kết hợp với mã thông thường khác.
<figure>
	<a href="http://farm9.staticflickr.com/8426/7758832526_cc8f681e48_b.jpg"><img src="http://farm9.staticflickr.com/8426/7758832526_cc8f681e48_c.jpg"></a>
	<figcaption><a href="http://www.flickr.com/photos/80901381@N04/7758832526/" title="Qúa trình biên dịch">Qúa trình biên dịch</a>.</figcaption>
</figure>
#### Cross compiler / Toolchain là gì?
Có thể hiểu Cross compiler hay còn gọi là Toolchain như sau:
Một source code được viết trên máy tính chạy chíp Intel (x86 platform), thông qua một cross compiler sẽ cho ra file nhị phân (mã máy) có khả năng chạy được trên một nên tảng chip khác là ARM (ARM platform). Trên hình native compiler là trình biên dịch để tạo ra file nhị phân chạy trên chính máy tính đang dùng để viết source code.
#### Hệ thống Cross compiler
Quá trình tạo và sử dụng Toolchain gồm 3 thành phần:
+ Build System: Hệ thống/platform tạo ra toolchain
+ Host System: Hệ thống chạy chương trình toolchain để compile source của chương trình ứng dụng.
+ Target: Là hệ thống chạy các chương trình do hệ thống host tạo ra.
#### Cách sử dụng cross compiler trên một hệ thống build với poky
- Chỉnh sửa sdk, thêm các gói cần thiết dùng để compile source. Ví dụ:
$ {path}/package-core-standalone-sdk-target.bbappend
RDEPENDS_${PN} += "\
linux-libc-headers-base-dev \
tzcslib-dev \
libtzcs-security-dev \
gtk+-dev \
"
- Build meta-toolchain sau khi chỉnh sửa sdk bằng poky:
$ bitbake meta-toolchain

- Copy sdk sang máy host để cài đặt:
$ cp debian-eglibc-x86_64-meta-toolchain-x86_64-toolchain-8.0.sh <host_machine>

- Run script để cài đặt toolchain trên máy host. Script này sẽ set lại biến PATH để sử dụng build source
$ source /opt/debian/8.0/environment-setup-x86_64-debian-linux

- Cuối cùng sử dụng ${CC} thay vì gcc trên máy host để compiler source:
VD: ${CC} -g abc.c -o  -lpthr -lX11 -ltzcs -ltzcs-s `pkg-config --cflags --libs gtk+-2.0`