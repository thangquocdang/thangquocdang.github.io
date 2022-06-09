---
layout: post
title: Bounding Box Predictions
---

# Bounding Box Predictions

Một cách để output các bounding box chính xác hơn là sử dụng thuật toán YOLO. Cách làm như sau chia ảnh thành các lưới (grid), về cơ bản sử dụng thuật toán Classification with localization và áp dụng cho từng ô (cell) trong grid. <br>
Ví dụ hình ảnh bên dưới với Grid 3x3 có 9 cell:

![image](https://user-images.githubusercontent.com/79956682/172772077-e80f32bc-bb42-4657-89ba-2454522ffe57.png)

_Ví dụ hình ảnh với Grid 3x3 có 9 cell_

### How do we define labels y?
Đối với mỗi grid cell label y được định nghĩa như sau:
$$ y=[p_c,b_x,b_y,b_w,b_h,c_1,c_2,c_3] $$

Trong đó: <br>
$$p_c$$: cho biết có bất kỳ đối tượng nào trong grid cell hay không <br>
$$b_x,b_y,b_w,b_h$$: sẽ cho biết vị trí của bouding box của đối tượng <br>
$$c_1,c_2,c_3$$: nhãn của đối tượng trong grid cell tương ứng với class 1, class 2, class 3 <br>

Nếu trung tâm của bounding box của đối tượng ở grid cell nào thì grid cell đó nhận giá trị $$p_c = 1$$ tương ứng với việc grid cell đó có nhiệm vụ phải nhận diện được đối tượng.

![image](https://user-images.githubusercontent.com/79956682/172772480-56543db3-e9f5-4b58-8093-38054a0826b6.png)
_Labels y trong thuật toán YOLO_

Cuối cùng, cho mỗi grid cell trong 9 cell, chúng ta sẽ có kích thước đầu ra 8 chiều. Ta có tổng cộng 9 cell nên kích thước đầu ra sẽ là 3x3x8. Với mỗi kích thước 1x1x8 ở phía trên bên trái tương ứng với vectơ đầu ra target cho phía trên bên trái của 9 grid cell.

### How do we encode these bounding boxes?
Chuẩn hóa $$b_x,b_y$$ theo grid cell và $$b_w,b_h$$ theo ảnh. 
![image](https://user-images.githubusercontent.com/79956682/172772633-8ab0de63-618e-4a94-b991-db97b0746e13.png)

$$  b_x=\frac{b_x}{w_{cell}}\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ b_y=\frac{b_y}{h_{cell}} $$ <br>
$$ b_w=\frac{b_w}{w_{img}}\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ b_y=\frac{b_y}{h_{cell}} $$
