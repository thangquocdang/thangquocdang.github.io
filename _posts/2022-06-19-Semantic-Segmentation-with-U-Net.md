---
layout: post
title: Semantic Segmentation with U-Net
---

Mục tiêu đối với bài toán Classification input là 1 bức ảnh và tìm ra nhữnng gì có trong đó chẳng hạn như có con mèo hay không, với bài toán Object detection thì có thêm mục tiêu xem bounding box xung quanh object được tìm thấy. <br>
Bây giờ mình sẽ tìm hiểu thuật toán thậm chí “tinh vi” hơn đó là Semantic segmentation. Mục tiêu là phác thảo cẩn thận xung quanh object đã được detect và biết chính xác cái nào pixel thuộc về object và pixel nào thì không.

### Object detection vs Semantic Segmentation 
Vậy semantic segmentation là gì? Xem ví dụ sau nè: <br>
Giả sử mình đang xây dựng 1 self-driving car với đầu vào là 1 ảnh và mình muốn phát hiện những chiếc xe khác. Nếu mình sử dụng object detection thì output sẽ có các bounding box được vẽ xung quanh các chiếc xe khác. Điều này có vẻ đủ tốt cho self-driving car, nhưng bạn muốn thuật toán của mình tìm ra xem mỗi pixel trong bức ảnh kia là gì bạn có thể sử dụng thuật toán semantic segmentation thì output đầu ra như ảnh bên dưới

![image](https://user-images.githubusercontent.com/79956682/174460855-0e5d3d49-0ce5-4edd-9d56-0538b8cde0bf.png)

Ở đây, nếu ta cố gắng detection con được thì sẽ trông như này nè

![image](https://user-images.githubusercontent.com/79956682/174460861-cce4d7e3-d44f-4ac2-bf9c-27223d04093e.png)

Có vẻ không được hiệu quả, nhưng với thuật toán sematic segmentation sẽ cố gắng dán nhãn cho mỗi pixel như con đường có thể lái được bởi màu xanh lá, những chiếc xe màu đỏ và cả các lề đường hay biển báo.

### Movation for U-Net
Ngoài ví dụ cho xe tự lái phía trên, hãy xem qua vài ứng dụng khác xem 

![image](https://user-images.githubusercontent.com/79956682/174460872-c4dea0a8-3a11-4c05-bbeb-fa50466fdfcc.png)

Có các ảnh X-ray như trên, nh bạn có thể giúp ích cho bác sĩ là nếu bạn có thể sement trong hình ảnh ra chính xác những pixel tương ứng. Ở ảnh bên trái ta có phổi, tim và xương sườn như trên được segment ra được biểu thị với các màu sắc khác nhau. Với segment này có thể dễ dàng phát hiện ra các bất thường và chuẩn đoán bệnh và cũng giúp đỡ cho bác sĩ. <br>
Ví dụ ảnh Brain MRI bên phải, một brain MRI được sử dụng cho việc phát hiện khối u não, việc segment thủ công khối u này ra rất tốn thời gian và công sức nhưng nếu có một thuật toán có thể segment một cách tự độ khối u màu trắng đó thì giúp các bác sĩ đỡ mất thời gian và cũng hữu ích cho việc phẫu thuật khối u. <br>
Thuật toán được sử dụng để tạo ra kết quả như trên được gọi là U – Net. Nào ta hãy xem semantic segmentation thực sự làm gì để có được kết quả như vậy.

### Per-pixel class labels

Để đơn giản, mình hãy xem xét ví dụ segment một chiếc xe từ bức ảnh như sau:

![image](https://user-images.githubusercontent.com/79956682/174460963-6b771f30-a66b-4bf6-baa8-58899f9277b3.png)

Trường hợp, nếu ta chỉ quan tâm duy nhất đến chiếc xe, thì ta sẽ có 2 label cho mỗi pixel: “1” nếu cho chiếc xe và “0” cho không phải là xe. Khi đó ảnh trên được biểu thị như sau: <br>

![image](https://user-images.githubusercontent.com/79956682/174460958-935dbfd5-ff9b-4fcb-9e52-f61d354990dc.png)

Khi đó thuật toán semantic segmentation sẽ output cho mỗi pixel là “1” hoặc “0”. Nếu pixel đó là “1” nếu nó là 1 phần của chiếc xe và “0” thì không phải. <br>
Ngoài ra nếu bạn muốn segment ảnh trên theo cách sau: “1” cho chiếc xe, “2” cho tòa nhà và “3” cho con đường, khi đó ta sẽ có Segmentation Map như ảnh phía bên phải. 

![image](https://user-images.githubusercontent.com/79956682/174460983-44179993-5585-4c83-93e3-7d05c4a6dc96.png)

Vậy ta phải train model sao cho output cho ra một matrix tổng thể của nhãn.

### Deeplearning for Semantic Segmentation
Trước tiên ta hãy bắt đầu với object recognition như sau: 

![image](https://user-images.githubusercontent.com/79956682/174460922-d753ac9f-3ff0-48d3-8ae4-ab4f6c578e0e.png)

Một mạng CNN quá quen thuộc nhỉ, input là một ảnh thông qua layer để cho ra một nhãn lớp y ̂. Để thay đổi kiến trúc này vào kiến trúc Semantic segmentation thì ta hãy xóa vài lớp cuối cùng cụ thể lớp FC và unit đầu ra.

![image](https://user-images.githubusercontent.com/79956682/174460932-b332b46b-a056-4e41-8774-575873a240ea.png)

Bước này quan trọng nè, ta nhận thấy rằng kích thước hình ảnh càng về sâu nhìn chung là nhỏ hơn và bây giờ nó cần phải lớn hơn và dần dần biến nói thành một hình ảnh full-size và đó là kích thước bạn muốn output.

![image](https://user-images.githubusercontent.com/79956682/174460939-2578b4de-59f4-4f1e-ad71-c7d061a1d7f2.png)

Một phép toán để làm cho bức ảnh lớn hơn được thực hiện bằng Transpose Convolution mà ta sẽ tìm hiểu ở bài tiếp. Cuối cùng bạn sẽ nhận được segmentation map của con mèo.


