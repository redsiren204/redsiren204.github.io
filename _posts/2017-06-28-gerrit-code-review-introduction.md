---
layout: post
title: "Gerit code review - Introduction"
date: 2017-06-28
excerpt: "Giới thiệu về Gerrit code review"
tags: [Gerrit, review, introduction]
comments: true
---

Các developer đã quen với các host version control như `Github`, `Bitbucket`, `Gitlab`, `Google source`...
Bài viết này giới thiệu về một công cụ quản lý và review source code cực hữu hiệu, tuy không còn mới lạ nhưng cũng là cái tên ít được nhắc đến - đó là Gerrit.

# Index
* Part 1 - Introduction
* Part 2 - Cài đặt, cấu hình Gerrit và authentication
* Part 3 - Tích hợp CI / CD

## 1. Gerrit là gì?

Gerrit là một công cụ hỗ trợ quản lý và review source code dựa trên nền tảng web, sử dụng git làm version control, được phát triển tại Google bởi Shawn Pearce trong quá trình phát triển dự án Android. Gerrit phiên bản 2.x được viết bằng Java trên nền J2EE servlet container. Gerrit là open source, nó hoàn toàn free và bạn có thể sử dụng nó làm công cụ quản lý và review source code cho team của mình.

Gerrit được thiết kế để cung cấp một framework cho review mọi commit trước khi nó được chấp nhận vào source code chính thức. Những thay đổi  được upload lên Gerrit nhưng không thực sự trở thành một chỉnh sửa của dự án cho đến khi chúng được review và chấp nhận.

Gerrit không đơn thuần chỉ là công cụ review mà nó còn có thể sử dụng để xây dựng một server quản lý source code giống như github, gitbuckit.

Những lý do mà bạn nên sử dụng Gerrit:
* Bạn có thể dễ dàng tìm lỗi trong thay đổi của commit
* Bạn có thể làm việc với Gerrit, nếu bạn có Git client, không cần cài đặt thêm bất kỳ Gerrit client nào
* Gerrit có thể được sử dụng như một trung gian giữa các developers và git repositories.

Home page : https://www.gerritcodereview.com/

## 2. Tổ chức Gerrit

<figure>
	<a href="https://raw.githubusercontent.com/redsiren204/redsiren204.github.io/master/resources/gerrit-part1/intro-quick-central-gerrit.png"><img src="https://raw.githubusercontent.com/redsiren204/redsiren204.github.io/master/resources/gerrit-part1/intro-quick-central-gerrit.png"></a>
</figure>
Gerrit sử dụng `Git` làm version control, nó xây dựng một khối repository - nơi lưu trữ main source (`Authoritative Repository`) và các commit của developers cần được review từ phía reviewers (`Pending Changes`).

Các developers sẽ Fetch source từ `Authoritative Repository`. Mỗi khi hoàn thành một task/change, thay vì push vào repository này, họ phải push vào khối `Pending Changes`. Điều này đảm bảo rằng mọi commit cần được `Approve` từ `Reviewer` (việc push vào `Authoritative Repository` chỉ khi có permission đặc biệt). Một khi change đã được approved, user có permission đặc biệt này có thể submit để merge change sang `Authoritative repository`.

Trong một vài tổ chức team khác nhau, các commit pending changes này cũng có thể được auto verify bởi `CI Build Server` (eg : Jenkins) trước khi thực hiện verify bởi yếu tố con người (Reviewer) - điều này giúp cho developer và reviewer phát hiện được lỗi sớm nhất.

## 3. Vòng đời và các thay đổi của Change

Lấy ví dụ, chúng ta giả sử Gerrit server được chạy trên một server là 192.123.99.7 với HTTP interface trên cổng 8000 và SSH interface trên cổng 29418. Dự án chúng ta đang làm có tên là RecipeBook.

### 1. Clone the Repository
{% highlight sh %}
$ git clone ssh://192.123.99.7:29418/RecipeBook.git
Cloning into RecipeBook…
{% endhighlight %}

Ta phải tạo ra thay đổi và commit nó ở local. Mỗi commit của Gerrit sẽ được thêm 1 Change-Id trong commit massage, mục đích là Gerrit có thể link các version khác nhau của cùng một thay đổi đang được review. Gerrit chứa một tiêu chuẩn Change-Id commit-msg hook mà nó sinh ra một Change-Id duy nhất khi bạn commit.

### 2. Create the Reviewer
Những thay đổi bạn thực hiện được lưu trên local khả dụng để review khi thực hiện lệnh push vào Gerrit server.
{% highlight sh %}
$ git commit -m “Create the Review”
$ git push origin HEAD:refs/for/master                                                  
Counting objects: 6, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (6/6), done.
Writing objects: 100% (6/6), 596 bytes | 0 bytes/s, done.
Total 6 (delta 4), reused 0 (delta 0)
remote: Resolving deltas: 100% (4/4)
remote: Processing changes: new: 1, refs: 1, done    
remote: 
remote: New Changes:
remote:   http://192.123.99.7:8000/1128 Create the Review
remote: 
To ssh://tamtv@192.123.99.7:29418/RecipeBook.git
* [new branch]      HEAD -> refs/for/master
{% endhighlight %}
Tương ứng mỗi branch trên Authoritative repository có một branch tương ứng refs/for/<branch_name> trên Gerrit server mà bạn push lên để tạo ra một review.

### 3. Review the Change
Work-flow mặc định của Gerrit yêu cầu 2 bước kiểm tra trước khi một commit được chấp nhận. Code-review là một ai đó nhìn vào code, để xem code đó có phù hợp với các tiêu chuẩn như project guideline, intent,.. Verifying là để kỉêm tra rằng sự thay đổi này có thể build đư ợc hay không, có thể pass các test case hay không.

Verification thường được thực hịên bởi một server build tự động. Gerrit Triggers Jenkins Plugin sẽ tự động build mỗi khi có thay đổi và cập nhật điểm verified theo đó.

Reviewer có thể download commit từ Gerrit server về local bằng 1 trong 2 cách: checkout hoặc cherry-pick.

Các vote của một commit bao gồm: -2, -1, 0, 1, 2.
-1, +1 chỉ là các ý kiến. -2, +2 là chấp nhận hoặc block commit này.
Để một commit được chấp nhận, phải có ít nhất 1 vote +2 và không có vote -2 nào.
2 vote +1 không tương đương với 1 vote +2.

Trong trư ờng hợp commit không đư ợc chấp nhận, người push commit lên phải thực hiện rework.

### 4. Reworking the Change
Vì có cơ chế Change-Id message hook trước khi upload change, việc reworking trở nên dễ dàng hơn. Chúng ta chỉ cần checkout sau đó chỉnh sửa và commit với cùng Change-Id. Push change lên Gerrit để review lại cũng tương tự như cách chúng ta tạo ra review.
{% highlight sh %}
$ <checkout first commit>
$ <rework>
$ git commit --amend
$ git push origin HEAD:refs/for/master
Counting objects: 5, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 546 bytes, done.
Total 3 (delta 0), reused 0 (delta 0)
remote: Processing changes: updated: 1, done
remote:
remote: Updated Changes:
remote:   http://192.123.99.7:8000/1128
remote:
To ssh://tamtv@192.123.99.7:29418/RecipeBook.git
 * [new branch]      HEAD -> refs/for/master
{% endhighlight %}

### 5. Trying out the Change

### 6. Manually verifying the Change

### 7. Submitting the Change

## 4. Tổng kết