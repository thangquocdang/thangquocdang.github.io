---
layout: post
title: [Course CNN] YOLO Algorithm
categories: ObjectDetection
---

Chúng ta đã tìm hiểu qua các thành phần của một thuật toán object detection như là Convolutional implementation of sliding window, Intersection Over Union, Non-max suppression. Bây giờ, ta lắp các mảnh ghép đó lại với nhau để tạo thành thuật toán object detection gọi là YOLO.

### Training

Đầu tiên mình đi xây dựng tập huấn luyện, giả sử mình đang huấn luyện model để detect 3 object:
1	- Người đi bộ
2	- Ô tô
3	- Xe đạp

Mình sử dụng 2 anchor boxes, dùng grid cell 3x3 thì output y có dạng 3x3x2x8, trong đó:
- 3x3 là grid cell sử dụng
- 2 là số anchor boxes
- 8 là bao gồm $$ p_c,b_x,b_y,b_w,b_h,c_1,c_2,c_3 $$

![image](https://user-images.githubusercontent.com/79956682/174419728-90cde537-fcdb-43fb-b64b-00112a3b8cae.png)

Vậy để xây dựng tập dữ liệu, mình phải đi qua từng grid cell và định nghĩ vector output y.
Ở bước training, bạn sẽ training một mạng ConvNet với ảnh đầo vào là 100x100x3 và đầu ra của mạng như ở ví dụ ảnh trên là 3x3x2x8 hoặc 3x3x16.

![image](https://user-images.githubusercontent.com/79956682/174419749-9a6ebf99-a9a5-4ab7-a4dc-5a368dfff826.png)

### Marking predections
Tiếp theo hãy xem thuật toán thực hiện dự đoán như thế nào.
Đưa vào một bức ảnh, mạng ConvNet sẽ cho ra 1 ma trận 3x3x2x8, với mỗi cell trong 9 cell bạn sẽ có vector đầu ra $$ y=[p_c,b_x,b_y,b_w,b_h,c_1,c_2,c_3, p_c,b_x,b_y,b_w,b_h,c_1,c_2,c_3] $$

![image](https://user-images.githubusercontent.com/79956682/174419777-2bd343a6-f672-4f5b-8ba2-7e70be9d7b0f.png)

Ở hình trên, cell màu xanh biển có vector y đầu ra màu xanh biển tương ứng với $$p_c$$ của 2 anchor box là 0, và sẽ cho ra một số giá trị nào đó cho các thành phần còn lại (don’t care) nhưng các giá trị này sẽ bị bỏ qua vì ConvNet biết rằng không có object nào ở đó cả vì thế cho dù nó có ra vị trí của bounding box hay predict đó là xe ô tô hay gì cũng không quan trọng. Ngược lại đối với cell màu xanh lá  có vector đầu ra y màu xanh lá tương ứng với hy vọng rằng $$p_c$$ của anchor box thứ nhất là 0 và mong muốn với anchor box thứ 2 sẽ dự đoán ra chính xác bounding box của xe ô tô.

### Outputting the non-max supressed outputs
Cuối cùng, tất cả phải trải qua quá trình gọi là non-max suppression, hãy xem nó như thế nào nàooo. <br>
Nếu bạn sử dụng 2 anchor box như ở ví dụ trên thì mỗi cell trong grid sẽ nhận được 2 bounding boxes. Vậy trong grid bạn sẽ nhận được tất cả là 18 bounding box. 

![image](https://user-images.githubusercontent.com/79956682/174419818-3fa51546-bbfb-4807-b3c5-613f69f56528.png)

Hình. Mỗi cell có 2 bounding box để dự đoán



Tiếp theo ta sẽ loại bỏ các boxes có giá trị $$p_c$$ thấp mà ConvNet cho rằng không có object ở đó. 

![image](https://user-images.githubusercontent.com/79956682/174419831-e027177c-0597-4460-afa6-0f8d1b42add6.png)

Hình. Sau khi loại bỏ các bounding box có giá trị xác suất thấp

Và cuối cùng nếu ta có 3 classes để detect, đối với mỗi class sẽ thực hiện non-max supperession một cách độc lập cho các object được predict là thuộc class đó.

![image](https://user-images.githubusercontent.com/79956682/174419841-5fa299b5-61b6-4044-badd-e5ecf189e0ef.png)

Hình. Sử dụng độc lập non-max supression cho mỗi class
