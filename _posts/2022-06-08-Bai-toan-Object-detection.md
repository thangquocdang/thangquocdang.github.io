# Bài toán Object detection

Bài toán Object detection là một trong những bài toán nhận được nhiều sự quan tâm trong lĩnh vực thị giác máy tính, nó được cải tiến rất nhiều so với trước đây. Trước khi đi tìm hiểu bài toán nhận dạng đối tượng ta cùng xem xét bài toán phân loại ảnh (Image classification) và xác định vị trí đối trượng (Object localization).
Note: tất cả các ví dụ bên dưới sử dụng labels: người đi bộ, xe máy, oto.
Image classification: input nhận vào một ảnh và nhiệm vụ của nó cho biết ảnh đó thuộc lớp ô tô, xe đạp hay người đi bộ
Classification with localization: input nhận vào một ảnh và nhiệm vụ không chỉ cho biết ảnh đó thuộc lớp nào, mà còn có nhiệm vụ vẽ một bouding box bao quanh đối tượng đó. Localization có nghĩa là chỉ ra vị trí đối tượng trên bức ảnh.
Object Detection: bài toán mà có nhiều đối tượng xuất hiện trong ảnh và nhiệm vụ nhận diện xem chúng thuộc lớp nào cùng với vị trí của chúng.
![image](https://user-images.githubusercontent.com/79956682/172671341-37f9aab5-feb5-44a7-a4e3-695b00c09696.png)
Sự khác biệt về dữ liệu của Image classification và Classification with localization so với Object Detection:
-	Image classification và Classification with localization: chỉ có một đối tượng và đối tượng này khá lớn nằm ngay giữa bức ảnh.
-	Object Detection: có nhiều đối tượng trong cùng một ảnh, với kích thước to nhỏ khác nhau

Classification with localization:
Đối với bài toán phân lớp thông thường, cho bức ảnh đi qua một mạng tích chập nhiều lớp và đầu ra là một vector đặc trưng (vector features) sau đó đưa qua hàm softmax để đưa ra dự đoán cuối cùng.
![image](https://user-images.githubusercontent.com/79956682/172671506-8cf8fd40-da72-4d10-b019-55911a1f2c11.png)
The standard classification pipeline
Ngoài xác định đối tượng đó thuộc lớp nào, ta muốn xác định thêm vị trí của chúng thì làm như thế nào?
Thay đổi kiến trúc trên lại một chút, thay vì đầu ra là xác suất của mỗi lớp, ta thêm thông tin của bouding box cụ thể là b¬x, by, bh, bw.
![image](https://user-images.githubusercontent.com/79956682/172671543-e7fe36b6-6177-408d-b364-40d85ee9ac59.png)
Xác định target label đầu ra y:
y = [p¬c, b¬x, by, bh, bw, c1, c2, c3]
Với:
p¬c: cho biết có bất kỳ đối tượng nào trong ảnh hay không 
b¬x, by, bh, bw¬: sẽ cho biết vị trí của bouding box của đối tượng
c1, c2, c3: nhãn của đối tượng trong ảnh tương ứng với class 1, class 2, class 3
![image](https://user-images.githubusercontent.com/79956682/172671619-fc19eff8-37cf-4ee5-881c-de0582702290.png)
Định nghĩa label  target y. Nếu không có đối tượng nào trong ảnh tương ứng với pc=0 các thành phần còn lại  “don’t care”.
Loss function:
f(ŷ,y)={█(〖((y_1 ) ̂- y_1)〗^2  + 〖((y_2 ) ̂- y_2)〗^2  + ...+ 〖((y_n ) ̂- y_n)〗^2,& if y_1=1@〖((y_1 ) ̂- y_1)〗^2  ,& if y_1=0)┤
