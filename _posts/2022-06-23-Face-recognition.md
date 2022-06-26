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

### Siamese Network
Nhiệm vụ của hàm d mà tìm hiểu trước đó cho biết 2 khuôn mặt giống nhau hay khác nhau như thế nào, một cách để làm tốt điều này là sử dụng Siamese network. <br>

Giả sử ta có ảnh đầu vào $$\mathrm{X}^{(1)}$$ đi qua một chuỗi Conv -> pooling -> Conv -> Pooling -> FC -> FC và output một feature vector f(x^{(1)})  $$ f\left(x^{(1)}\right) $$, giả sử feature vector này có kích thước là 128,  và $$ f\left(x^{(1)}\right) $$ được xem như encoding với đầu vào là $$\mathrm{X}^{(1)}$$ . <br>

Cách để xây dựng face recognition system là bạn so sánh 2 hình ảnh, 2 ảnh này cùng đi qua cùng mạng Neural giống nhau về kiến trúc, parameters như hình phía bên dưới

![image](https://user-images.githubusercontent.com/79956682/175820548-989ba3b6-9337-4c38-a9c4-c1549783df32.png)

$$\mathrm{X}^{(1)}$$ -> CovNet -> $$ f\left(x^{(1)}\right) $$
$$\mathrm{X}^{(2)}$$ -> CovNet -> $$ f\left(x^{(2)}\right) $$

Nếu encoding ra vector biểu diễn tốt 2 hình ảnh trên thì bạn có thể định nghĩa hàm như sau (có thể sử dụng norm 1 hoặc norm 2 để tính khoảng cách 2 vector):

$$d\left(x^{(1)}, x^{(2)}\right)=\left\|\mathrm{f}\left(x^{(1)}\right)\mathrm{f}\left(x^{(1)}\right)\right\|$$

Vậy ý tưởng là sử dụng 2 model giống hệt nhau trên 2 đầu vào khác nhau và sau đó so sánh vector đầu ra của chúng, được gọi là Siamese neural network architecture. <br?
Train mạng Siamese này như thế nào hay nói cách khác model phải học được hàm d?, nên nhớ rằng ta có 2 mạng neural nhưng có cùng parameters.  <br>
Tóm lại: <br>

- Các parameters của mạng Neural sẽ xác định một encode cho mỗi đầu vào x^{\left(i\right)} được biễu diễn bởi 1 vector.
- Và những gì phải làm là model học sao cho các parameters phân biệt được rằng:

$$\text { Nếu } x^{(i)}, x^{(j)} \text { là cùng một người thì}\left\|\mathrm{f}\left(x^{(i)}\right)-\mathrm{f}\left(x^{(j)}\right)\right\| \text { là nhỏ. }
$$
$$
\text { Nếu } x^{(i)}, x^{(j)} \text { là cùng một người thì }\left\|\mathrm{f}\left(x^{(i)}\right)-\mathrm{f}\left(x^{(j)}\right)\right\| \text { là lớn. }
$$

Tiếp theo mình cùng đi xây dựng hàm loss cho Siamese network. <br>

### Triplet loss
Một cách để learn parameters của mạng Neural để cho ra một encode (vector presentation) tốt với hình ảnh của bạn là áp dụng gradient descent trên triplet loss function. Cùng tìm hiểu nó như thế nào nhé :) <br>
Để áp dụng triplet loss là bạn cần so sánh các cặp ảnh với nhau, ví dụ: <br>

![image](https://user-images.githubusercontent.com/79956682/175820676-a9f0ab96-87cb-46be-b6b5-3255e8da462c.png)

Trong triplet loss, những gì bạn cần làm là nhìn vào ảnh Anchor sau đó tính khoảng cách với các ảnh khác, trong đó khoảng cách giữa Anchor và Positive và khoảng cách giữa Anchor và Negative  phải xa hơn nhiều so với khoảng cách giữa Anchor và Positive. <br>
Vậy tại một thời điểm, bạn tìm 3 bức ảnh gồm Anchor (A), Positive (P), Negative (N). Để formalize điều này, bạn muốn d(A,P) phải nhỏ hơn d(A,N). <br>

$$
\begin{gathered}
d(A, P) \leq d(A, N) \\
\leftrightarrow\|f(A)-f(P)\| \leq\|f(A)-f(N)\| \\
\leftrightarrow\|f(A)-f(P)\|-\|f(A)-f(N)\| \leq 0(1)
\end{gathered}
$$

<br>Ý nghĩa thêm hyperparameter α: 
Nếu $$\|f(A)-f(P)\|=0$$ và $$\|f(A)-f(N)\|=0$$ hay nói cách khác $$f(img)\ =\ \vec{0}$$ , thì mạng Neural xem như không học được gì, và để đảm bảo mạng Neural không output bằng 0 cho tất cả các encoding hoặc để chắc chắn rằng nó không đặt tất cả các encoding bằng nhau, thì ta thêm - $$\alpha$$ để nó phải nhỏ hơn một số âm, và $$\alpha$$ này cũng được gọi là margin. <br>

Ví dụ ta đặt $$\alpha\ (margin)\ =\ 0.2 , d\left(A,P\right) = 0.5$$ thì  $$d\left(A,P\right)+\ \alpha\ \le\ d\left(A,N\right)$$

Và sẽ không thỏa mãn nếu $$d\left(A,N\right)$$ chỉ là 0.51, mặc dù khoảng cách $$d\left(A,N\right)$$ lớn hơn $$d\left(A,P\right)$$ nhưng vẫn không đủ tốt. Chúng ta muốn $$d\left(A,N\right)$$ lớn hơn $$d\left(A,P\right)$$ một khoảng $$\alpha$$ (margin). Cụ thể ở ví dụ trên $$d\left(A,N\right)$$ phải ít nhất là 0.7 hoặc lớn hơn để thoải mãn khoảng cách giữa chúng ít nhất là 0.2. Bạn có thể đẩy $$d\left(A,N\right)$$ lên hoặc hạ $$d\left(A,P\right)$$ xuống để thỏa mãn margin = 0.2.

### Loss function
Triplet function được xác định trên bộ ba hình ảnh A, P, N <br>

 $$£(A,P ,N) = max(||f(A) - f(P)|| - ||f(A) - f(N)|| +  α,0) $$
 
Trên toàn bộ training set: <br>

$$ i = 1m£(A(i),P(i),N(i)) $$

Nếu bạn có trainning set với 10k ảnh của 1k người, những gì bạn làm là lấy 10k ảnh đó và tạo ra bộ ba A, P, N sao đó train sử dụng gradient descent trên loss function. 
Vậy bạn chọn triplets A, P, N như thế nào? <br>

Suốt quá trình tranning, nếu A, P, N được chọn ngẫu nhiên, lúc đó (A, N) rất khác so với (A, P) thì biểu thức (2) rất dễ dàng được thỏa mãn, do đó mạng Neural sẽ không học hỏi được gì nhiều từ nó. <br>
Chọn A, P, N sao cho nó “hard” hơn để model học “hard” hơn nếu bạn chọn sao cho $$d\left(A,P\right)\approx d\left(A,N\right)$$ khi đó mạng Neural sẽ cố gắng đẩy $$d\left(A,N\right)$$ hoặc giảm $$d\left(A,P\right)$$ xuống.

### Face Verification and Binary Classification
Triplet loss là một cách tốt để học các parameters của CNN để nhận diện khuôn mặt, còn có một cách khác để làm việc này, với bài toán face recognition cũng có thể coi là như một bài toán nhị phân. <br>

![image](https://user-images.githubusercontent.com/79956682/175821367-46b6d930-444b-4030-97a0-4ec3b2f1f545.png)

Một cách khác để train một mạng Neural, là lấy output một cặp mạng Neural này như Siamese Network sau đó input vào một unit logistic regression để đưa ra dự đoán, target output sẽ là 1 nếu “same” và 0 nếu “different” , đây là cách xem bài toán face regcognition như một bài toán phân loại nhị phân và cũng là một giải pháp thay thế cho triplet loss để tranning một hệ thống như thế này. <br>
Vậy Unit logistic regression ở đây thực sự làm gì? <br>
Output $$\hat{y}$$ sẽ là một hàm sigmoid cụ thể: <br>

$$\hat{y}\ =\ \delta(\sum_{k=1}^{K}{|f(x^{(i)})}_k\ -\ {f(x^{(j)})}_k|) $$

Với k là thành phần thứ K trong vector feature, và bạn thêm wi và b tương tự như một unit logistic regression thông thường, và sẽ train để tính weight thích hợp cho k feature này để xem 2 ảnh đó là “same” hay “different” <br>

$$\hat{y}\ =\ \delta(w_k\sum_{k=1}^{K}{|f(x^{(i)})}_k\ -\ {f(x^{(j)})}_k|\ +\ b) $$

Để triển khai hệ thống này là nếu có ảnh mới $$x^{(i)}$$ với hy vọng rằng recognition được người đó và $$x^{(j)}$$ là dữ liệu trong database, $$x^{(j)}$$ đã được tính toán từ trước và được lưu lại. Khi có ảnh mới x¬¬(i) thì chỉ cần đi qua mạng CNN sẽ có được vector feature, lấy vector này và cùng với vetor feature đã được lưu trong database đưa qua Unit logistic regression đưa đưa ra dự đoán. Bạn không cần lưu trữ hình ảnh thô, và nếu có thêm ảnh mới trong database của bạn thì bạn không cần tính lại encode toàn bộ, chỉ cần encoding cho ảnh mới.  
