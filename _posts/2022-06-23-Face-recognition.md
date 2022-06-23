---
layout: post
title: Face Recognition 
categories: FaceRecognition 
---

### What is Face Recognition?

Trước tiên mình cùng xem qua thuật ngữ sử dụng trong face recognition là Face verification vs Face recognition <br>

#### Verification: <br>
-	Input image, name/ID <br>
-	Output: Xác minh xem ảnh đầu vào có phải là người đó hay không <br>
-	Bài toán này thường được gọi là 1:1 problem, bạn chỉ muốn biết liệu người đó có phải thật hay không <br>

#### Recognition <br>
-	Có database của k người <br>
-	Input: một ảnh <br>
-	Output ID của ảnh nếu ảnh là một trong k người (hoặc “not recognized”) <br>
-	Là bài toán 1:k problem, khó hơn nhiều so với verification vì giả sử bạn có hệ thống verification đạt 99% accuracy, và bây giờ bạn có k=100 trong hệ thống recognition, nếu bạn áp dụng hệ thống này vào bài toán recognition với 100 trong database của bạn, bạn có 100 lần dự đoán thì có 1 lần mắc lỗi. . Vì vậy, Nếu bạn có một database gồm 100 người và nếu bạn muốn một lỗi nhận dạng có thể chấp nhận được thì accuracy phải 99% hoặc thậm chí cao hơn 99.99%. <br> 

### One-shot learning problem

Một trong những thách thức của face recognition cần phải giải quyết đó là one-shot learning problem. Hầu hếu các ứng dụng face recognition cần phải có khả năng recognize một người khi đầu vào chỉ là một ảnh duy nhất của người đó. <br>
Giả sử bạn có một database gồm 4 hình ảnh về nhân viên trong công ty, và có ai đó xuất hiện tại văn phòng và họ muốn đi qua cánh cửa, những gì hệ thống của bạn phải làm là phải recognize người đó có trong database của bạn hay không, mặt dù chỉ có một hình ảnh của người đó có trong database. <br>
Một cách tiếp cận có thể thử là input ảnh của người đó và đi qua ConvNet và output là một 5 unit sử dụng softmax để dự đoán người đó có phải 1 trong 4 người trong database hoặc không phải người nào ở trên. Nhưng điều này hoạt động không thực sự tốt nếu bạn không có bộ dataset tranning đủ lớn để trainning một mạng ConvNet cho tác vụ này. <br>
Thêm một điều nữa là điều gì sẽ xảy ra nếu ta có thêm một người mới gia nhập vào công ty của bạn? Giờ bạn có 5 người để recognize vậy output của mạng ConvNet trên phải là 6, do đó phải tranning lại ConvNet một lần nữa, dường như cách tiếp cận này không hiệu quả. <br>
Thay vào đó, để hoạt động hiệu quả hơn, không cần phải tranning lại model khi có thêm dữ liệu mới, ta sẽ cho model học một similarity function. <br>
_d(img1, img2) = degree of difference between images_ <br>
Similarity function được biểu thị là d, nhận 2 hình ảnh img1 và img2 và cho ra mức độ khác nhau giữa 2 hình ảnh đó.  <br>
Vì vậy, nếu hai hình ảnh nhận vào cùng là 1 người thì mong muốn output là 1 số nhỏ, ngược lại nếu 2 ảnh input là khác nhau (không phải cùng 1 người) thì mong muốn output là một số lớn. <br>
Trong suốt quá trình recognition, nếu độ khác nhau giữa 2 bức ảnh nhỏ hơn 1 ngưỡng τ (hyperparameter) thì bạn sẽ dự đoán 2 bức ảnh này cùng 1 người <br>

$$\left\{\begin{array}{c}\text { if d }(\text { img } 1, \text { img } 2) \leq \tau \rightarrow \text { "same" } \\\ { if  d(img1,img } 2)>\tau \rightarrow \text { "difference" }\end{array}\right.$$  <br>

Đó là cách giải quyết bài toán face verification, để sử dụng điều này cho face recognition bạn cần sử dụng hàm d này để so sánh 2 hình ảnh theo từng cặp bao gồm ảnh cần recognition và ảnh có trong database, nếu ảnh mà người không có database thì mong muốn hàm d output một số rất lớn để chứng tỏ người này là “unknow”. <br>
Điều này giải quyết được one-shot learning problem, miễn là model có thể học được hàm d nhận vào một cặp ảnh và output là cùng 1 người hay 2 người khác nhau. Và nếu có người mới trong database thì nó vẫn chạy tốt. <br>
