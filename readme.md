
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

When accessing our website, the user will be automatically redirected to the Webroot/index.php thanks to two.htaccess files.

The first one will redirect the user to the Webroot directory.

```
RerwiteEngine on
RewriteRule ^([a-zA-Z0-9\-\_\/]*)$ Webroot/$1
```

And the second one will redirect him/her to the index.php. Notice that we store the parameter (p=$1).

```
RerwiteEngine on
RewriteCond %{REQUEST_URI} !\.(?:css|js|jpe?g|gif|png)$ [NC]
RewriteRule ^([a-zA-Z0-9\-\_\/]*)$ index.php?p=$1
```

The index.php is requiring all the files that we will need for the instantiation of the dispatcher. After creating a instance of the Dispatch class, we are ready to set our routing logic.


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

### Routing system

#### _request.php_

The goal of this file is to get the url requested by the user.

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

The router takes the url captured by the _request.php_ and explode the url into 3 different parts on the "/" character :

```php
$explode_url = explore('/', $url);
$explode_url = array_slice($explode_url, 2);
$request->controller = $explode_url[0];
$request->action = $explode_url[1];
$request->params = array_slice($explode_url, 2);
```

These inputs will be handled by the dispatcher. The dispatcher is doing the same job as an air traffic controller. When a new request is loaded, it selects the controller and the action with parameters. So with only one method (dispatch()), we can launch all this routing logic.

![][9]

### Database

Our model is going to handle the request to our database. So we will have to call our database a lot of time. In a simple way, at each connexion, we can create an instance of the database. This solution is not really efficient. I recommend we create a singleton which will handle the connexion to our database :

```php
class Database
{
    
}
```

![][10]

### MVC

Now that we set up the dispatcher, our website can load an action from a controller.

Here we want to make a todo app, so we have to create a _tasksController.php_. This controller is going to ask for data from the model Task.php and then pass the data to a view. To make this process easier, we are going to create a parent class Controller that will handle this.

![][11]

The _set()_ method is going to merge all the data that we want to pass to the view.

The _render()_ method is going to import the data with the method extract and then load the layout requested in the Views directory. Moreover, this allows us to have a layout in order to avoid the stupid repetition of HTML in our views.

We are ready to work on our tasksController.php. Just to test out our code, I'm going to create an index action :

![][12]

And a quick view with a "Hello" message.

And here is the result :

![][13]

Our MVC framework is set up ! Now we just have to make the CRUD actions about the task resource. If you want more details of this and get the website with the tasks CRUD, you can check out the repo on my Github.

So now, you have developed an MVC structure that is a lot more sustainable than our traditional php website. But there is still a lot of work to do (security, error handling…). These topics are already handled by frameworks like Laravel or Symfony.

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

  