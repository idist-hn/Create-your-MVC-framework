
[Source](https://medium.com/@noufel.gouirhate/create-your-own-mvc-framework-in-php-af7bd1f0ca19 "Permalink to Create your own MVC framework in PHP – Noufel Gouirhate – Medium")

# Tự tạo cho mình một framework MVC bằng PHP - Noufel Gouirhate - Medium

![][https://cdn-images-1.medium.com/max/800/1*2tQcyKoVHazxnISjcyxaCA.jpeg]

Trước khi học về MVC, tôi đã từng phát triển các website theo một con đường. Một cách tự nhiên nhất là tôi cứ tạo mỗi file php cho từng page. Và mỗi file này được trộn lẫn cả php, html,... thực sự rất khó chịu. Chúng tôi càng đi sâu vào dự án thì càng gặp nhiều khó khăn trong việc bảo trì nó: có quá nhiều dư thừa trong cấu trúc html, khó đọc code vì có nhiều đoạn php chèn trực tiếp vào trong đó... tuy nhiên ngay cả khi dự án kết thúc trong tình trạng tương đối tốt thì chúng tôi thực sự cũng bị lẫn lộn trong đống logic của mình.

```
Bạn có thể theo dõi bài hướng dẫn thông qua repo Github của tôi:
 https://github.com/ngrt/MVC_todo
 ```

### Giải thích về MVC 

Để tránh những tình huống này, các nhà phát triển đã bắt đầu suy nghĩ về một cách tổ chức code cho một website. Một cách thực hiện điều đó là _MVC design pattern_. Mục tiêu của nó là phân chia project ra thành 3 phần lớn:

* **Model**: tương tác với database. Nó nhận, lưu trữ và lấy dữ liệu cho người dùng.
* **View**: hiển thị các thông tin tới người dùng và gửi dữ liệu tới controller
* **Controller**: gửi và nhận dữ liệu từ model và chuyển qua view.

![][1]

### Cấu trúc 

Để thiết lập design pattern này, chúng ta sẽ dựng cấu trúc cho project của mình. Chúng ta có thể tìm thấy nhiều khả năng trên internet. Nhưng ở đây tôi đề xuất:

![][2]

Như bạn thấy, chúng ta sử dụng phần khung cho framework MVC với 3 thư mục (Models, Views, controller) và một vài phần khác:

* Folder Webroot chỉ là một thư mục có thể truy cập bởi user.
* Router.php, dispatcher.php, request.php, .htaccess là thành phần của hệ thống điều hướng.
* Config : mọi cài đặt tùy chọn cần thiết cho website của bạn. Chúng ta sẽ sủ dụng lại và file db.php là điểm duy nhất có thể truy cập tới database của bạn (singleton class).

### Kiến trúc mức Global

![][3]


Khi truy cập vào website của chúng ta, user sẽ tự động chuyển hướng tới Webroot/index.php thông qua 2 file .htaccess.

File đầu tiên để chuyển hướng user vào thư mục Webroot.

```
RerwiteEngine on
RewriteRule ^([a-zA-Z0-9\-\_\/]*)$ Webroot/$1
```

Và file thứ 2 để chuyển hướng họ vào index.php. Chú ý là chúng ta lưu trữ ở biến (p=$1).

```
RerwiteEngine on
RewriteCond %{REQUEST_URI} !\.(?:css|js|jpe?g|gif|png)$ [NC]
RewriteRule ^([a-zA-Z0-9\-\_\/]*)$ index.php?p=$1
```

Index.php yêu cầu tất cả các file mà chúng ta cần để cài đặt việc điều phối. Sau khi tạo một instance của class Dispatch, chúng ta đã sẵn sàng để thiết lập logic điều phối của mình.

```php
define('WEBROOT', str_replace("Webroot/index.php", "", $_SERVER["SCRIPT_NAME"]));
define('ROOT', str_replace("Webroot/index.php", "", $_SERVER["SCRIPT_FILENAME"]));

require(ROOT, . 'Config/core.php');

require(ROOT . 'router.php');
require(ROOT . 'request.php');
require(ROOT . 'dispatcher.php');

$dispatch = new Dispatcher();
$dispatch->dispatch();
```

### Hệ thống điều hướng

#### _request.php_

Mục tiêu của file này là nhận các url được yêu cầu từ phía người dùng.

```php
class Request
{
    public $url;
    
    public function __contruct()
    {
        $this->url = $_SERVER["REQUEST_URI"];
    }
}
```

#### router.php

router lấy url từ _request.php_ và phân rã url đó thành 3 phần khác nhau bởi dấu "/":

```php
$explode_url = explore('/', $url);
$explode_url = array_slice($explode_url, 2);
$request->controller = $explode_url[0];
$request->action = $explode_url[1];
$request->params = array_slice($explode_url, 2);
```

Những đầu vào này sẽ được xử lý bởi thành phần điều phối. Thành phần điều phối sẽ thực hiện công việc tương tự như một người kiểm soát việc lưu thông. Khi có một request mới được tải, nó lựa chọn controller và các action với các tham số. Vì thế chỉ cần một phương thức (dispatch()), chúng ta có thể khởi chạy tất cả các logic điều hướng này.

![][9]

### Database

Mô hình này của chúng ta sẽ xủ lý yêu cầu tới database của mình. Vì vậy chúng ta sẽ phải gọi tới cơ sở dữ liệu của chúng ta nhiều lần. Bằng một cách đơn giản, với mỗi kết nối, chúng ta có thể tạo ra một instance của database. Giải pháp này thực sự không hiệu quả. Tôi đề xuất chúng ta tạo một singleton mà sẽ xử lý việc kết nối tới cơ sở dữ liệu của mình:

```php
class Database
{
    
}
```

![][10]

### MVC

Giờ chúng ta thiết lập bộ điều phối, website của chúng ta có thể thực hiện các action từ một controller.

Giờ chúng ta muốn tạo ra một app todo, vì vậy chúng ta phải tạo một _TaskController.php_. Controller này sẽ lấy dữ liệu từ model Task.php và chuyển chúng tới một view. Để giúp quá trình này dễ dàng hơn, chúng ta sẽ tạo một class Controller cha để xử lý nó.

![][11]

Phương thức _set()_ sẽ merge tất cả dũ liệu mà chúng ta muốn vào view.

Phương thức _render()_ sẽ import dữ liệu vào với trích xuất các phương thức và sau đó tải các layout được yêu cầu trong thư mục Views. Thêm nữa là nó cho phép chúng ta có một layout để tránh việc lặp lại một cách ngu ngốc các phần HTML trong view của chúng ta.

Chúng ta đã sẵn sàng làm việc trên TaskController của mình. Hãy thử kiểm tra code của mình, tôi sẽ tạo ra một action index:

![][12]

Và cái view đơn giản chỉ với thông báo "Hello".

Và đây là kết quả:

![][13]

Framework MVC của chúng ta đã hoàn thành, giờ chúng ta phải tạo các action CRUD về quản lý task. Nếu bạn muốn chi tiết hơn về điều này, có một website với các task CRUD, bạn có thể xem qua repo của tôi trên Github.

Vậy bây giờ, bạn đã phát triển xong một cấu trúc MVC bền vững hơn nhiều so với cấu trúc truyền thống của website php thông thường. Nhưng còn rất nhiều việc phải làm(bảo mật, xủ lý lỗi,...). Những topic này xử lý bởi framework như Laravel hay Symfony.

[1]: https://cdn-images-1.medium.com/max/1600/1*xnuMvzXzmAxYXcRrd1Wj5Q.png
[2]: https://cdn-images-1.medium.com/max/1600/1*IA0nHOylfQYxjnGwi1XGaQ.png
[3]: https://cdn-images-1.medium.com/max/1600/1*gRErOZyn7ptn373U9fv0Yg.png
[4]: https://cdn-images-1.medium.com/max/1600/1*_agMehf9fNamnUtWqnv4kg.png
[5]: https://cdn-images-1.medium.com/max/1600/1*I67GugEBv0ONYruFet_wbA.png
[6]: https://cdn-images-1.medium.com/max/1600/1*tPlzi7umbyf6JJ9WSkfx8w.png
[7]: https://cdn-images-1.medium.com/max/1600/1*3m5NfXYUAoDAllbVdS8N1w.png
[8]: https://cdn-images-1.medium.com/max/1600/1*EVNESudstEyfXwvx6b5f1Q.png
[9]: https://cdn-images-1.medium.com/max/1600/1*I9mpgAX_OpaJa35jiQfUVg.png
[10]: https://cdn-images-1.medium.com/max/1600/1*EBlYRwirAwcywwTg0T1waw.png
[11]: https://cdn-images-1.medium.com/max/1600/1*Dmg_0gOYlq5ONFlKRkfbGw.png
[12]: https://cdn-images-1.medium.com/max/1600/1*n6l3kSUruZfOxpUNmKgTHA.png
[13]: https://cdn-images-1.medium.com/max/1600/1*MSUdTGHL_ozUGdBVeixirQ.png

  