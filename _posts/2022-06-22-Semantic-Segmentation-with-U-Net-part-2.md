---
layout: post
title: Semantic Segmentation with U-Net (Part 2)
categories: SemanticSegmentation
---

### Transpose Convolutions
Transpose convulution là một phần quan trọng của kiến trúc U – Net, làm thế nào khi đầu vào là 2x2 và biến nó thành đầu ra 4x4, transpose convolution sẽ làm điều đó cho bạn. <br>
Với convolution thông thường trong mạng ConvNet có input 6x6x3 convolve với 3x3x3 filters thì sẽ có output là 4x4x5 (5 filters, stride=1, padding= “same”).

![image](https://user-images.githubusercontent.com/79956682/175001593-2433f956-2704-41f9-b401-fc7ba5e41d4a.png)

Transpose convolution thì khác đi một chút, input 2x2 convolve với 3x3 filter thì output sẽ là 4x4, output này lớn hơn với kích thước input. <br>
Chi tiết hơn cách tính toán Transpose convolution

![image](https://user-images.githubusercontent.com/79956682/175001635-efa0c4f9-6e50-4fea-8867-fb9b4c90b181.png)

Theo convolution thông thường, bạn sẽ lấy filter và trượt nó trên input, sau đó nhân và cộng chúng lại còn đối với transpose convolution bạn sẽ đặt filter trên output.

### U-Net Architecture Intuition

Chúng ta cùng xem nhanh qua kiến trúc của U- Net từ đó có thể cảm nhận được phần nào các U-Net hoạt động

![image](https://user-images.githubusercontent.com/79956682/175001716-b3809411-a21b-4a20-8852-8a76db902288.png)

Ảnh trên là sơ đồ về kiến trúc mạng neural cho Semantic Segmentation, từ một hình ảnh với H,W lớn và D nhỏ đến W,H nhỏ và D lớn (sâu hơn) ở đoạn này mình có vẻ mất nhiều thông tin vì kích thước nhỏ nhưng nó sâu hơn nhiều. Sau đó ở nửa sau của mạng neural, sử dụng transpose convolution để đưa trở lại với kích thước ban đầu. Sửa đổi mạng trên 1 xíu làm cho mạng hoạt động tốt hơn là sử dụng **skip connection**, mạng này được gọi là U-Net. 

![image](https://user-images.githubusercontent.com/79956682/175001806-fdbaeb29-886f-44e4-b57a-e811e69e8585.png)

#### Vậy tại sao ta làm điều này?  

Các lớp layer càng về phía sau để quyết định vùng nào là một con mèo cần có 2 loại thông tin. Thứ nhất là high level spatial, high level contextual information mà nó nhận được từ layer ở ngay phía trước nó và hy vọng rằng mạng neural tìm thấy thứ gì đó của con mèo nằm ở trong ảnh nhưng vẫn còn thiếu chi tiết fine gained spatial information bởi vì ở lớp đó là lower spatial resolution với height và width là lower. Vì vậy, sử dụng skip connection cho phép mạng Neural lấy layer ở phía Encoder với high resolution, low level feature information nó có thể nắm được các pixel nào là hữu ích.
Với cách làm này, ở layer phía sau có cả lower resolution nhưng high level spatial, high level contextual information  cũng như là low level nhưng detailed texture để quyết định xem pixel đó có phải là 1 phần của con mèo hay không.

### U-Net Architecture

Bây giờ mình cùng tìm hiểu chi tiết chính xác U-Net hoạt động như thế nào, Kiến trúc mạng U-net trông như thế này: 

![image](https://user-images.githubusercontent.com/79956682/175001937-5c5ad986-f9e0-4e6d-9812-6c3a9b4ad2c0.png)

Input là một ảnh giả sử là HxWx3 đi qua các lớp convolution, activation tăng kích thước channel nhưng vẫn giữ H và W, tiếp theo sử dụng Max Pool để giảm H và W, sau đó qua tiếp tục các lớp convolution, activation tăng kích thước channel nhưng vẫn giữ H và W và sử djng Max Pooling lần nữa, cứ tiếp tục như thế đến khi H và W rất nhỏ thì mình sử dụng Transpose convolution layer để đưa kích thước này trở lại như ban đầu, một điều nữa là sử dụng thêm skip connection copy từ phía bên trái sang bên phải như trong hình. Và layer cuối cùng sẽ ảnh xạ vào segmentation map bằng cách sử dụng 1x1 convolution, kích thước đầu ra này sẽ là HxWx#numberClasses
