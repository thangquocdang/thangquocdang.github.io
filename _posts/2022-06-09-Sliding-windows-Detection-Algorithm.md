---
layout: post
title: Sliding windows Detection Algorithm [Course CNN]
categories: ObjectDetection
---

## Sliding windows Detection Algorithm
Bài này sẽ sử dụng ConvNet (mạng tích chập Neural) để thực hiện nhận diện đối tượng sử dụng Sliding windows Detection Algorithm (Cửa sổ trượt). <br>

Ý tưởng chính của Sliding windows Detection là lấy window lần lượt trượt trên bức ảnh, mỗi lần trượt đưa vào ConvNet để dự đoán window đó có phải là đối tượng cần tìm hay không?<br>
Nhược điểm: 
-	Khi crop ảnh ra nhiều vùng khác nhau và đưa vào ConvNet một cách độc lập nên chi phí tính toán rất lớn.
-	Thời gian tính toán lâu, hiệu quả kém.

![image](https://user-images.githubusercontent.com/79956682/172757276-660436d1-7b1d-4ef3-85ac-c6e7cd839f7c.png)

Các window với size khác nhau
Vấn đề chi phí tính toán đã có giải pháp khá ổn, cụ thể là Sliding windows object detector được triển khai theo convolutional.

## Convolutional implementation of sliding window

Để xây dựng Convolutional implementation of sliding window, trước tiên hay xem xét ConvNet đơn giản như sau:

![image](https://user-images.githubusercontent.com/79956682/172757376-b8272b09-d4b1-42d6-9343-ae08ccb4a0ba.png)

Đầu vào với kích thước 14x14x3, đầu ra là y với xác suất của mỗi lớp, tại layer 5x5x16 có bước Flatten để đưa qua layer Fully connected (FC), giờ muốn chuyển các layer FC thành các convolutional layer thì làm như thế nào? <br>

![image](https://user-images.githubusercontent.com/79956682/172757423-96687b67-ef69-473f-b165-3625008f8736.png)

Tại layer 5x5x16 sử dụng 400 filter size 5x5 đầu ra sẽ là 1x1x400, tiếp đến sử dụng convolutional 1x1. Việc sử dụng 1x1 convolution có ý nghĩa chỉ lấy đúng pixel đó và xài thông tin chiều sâu của filter đó, mặc khác giúp kiểm soát số kênh (giảm hoặc tăng), tiết kiệm được các phép tính toán trong một số network (ví dụ Inception Network) <br>

## Convolutional implementation of sliding window

Với cách làm như trên thay thế các unit của lớp FC thành các lớp convolutional 
![image](https://user-images.githubusercontent.com/79956682/172757952-f1406637-564c-45d3-b595-fe54cbda0d6b.png)

So với Sliding windows Detection ban đầu, thay vì crop ảnh rồi cho qua ConvNet để dự đoán thì chúng ta triển khai Convolutional ngay trên toàn bộ ảnh để cho ra dự đoán. <br>
Nhược điểm: Vị trí các bouding box không quá chính xác <br>


Nguồn tham khảo: https://www.coursera.org/learn/convolutional-neural-networks
