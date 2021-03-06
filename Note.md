## Năm 2018
- Tốt nghiệp đại học.
- Theo vào một công ty để học tập và nghiên cứu Openstack, python
- Các mục tiêu năm 2018
	1. Tìm hiểu và cố gắng làm chủ Openstack
	2. Code Python ở level khá
	3. Làm chủ và phát triển giải pháp KVM-VDI




# Mticket App
## Công nghệ sử dụng
1. Back-end: Python-Django
2. Font-end: Html, css, jquery
3. Database: Mysql
## Thành phần đặc biệt
1. Xử lý real-time,chat
- Sử dụng công nghệ websocket django channel [link](http://channels.readthedocs.io/en/latest/introduction.html)
- Sử dụng redis để lưu message qua lại của websocket (client - server)
2. Phơi API cho các service khác sử dụng (tạo ticket,..)
## Một số lưu ý.
1. Tìm hiểu thêm vể redis server: Redis là một hệ quản trị CSDL noSQL, lưu database theo kiểu key-value, truy vấn rất nhanh vì nó lưu tạm vào cache trước khi lưu xuống disk
2. Để dữ liệu được update real-time, cần sử dụng websocket và load lại các table thông tin. Để load lại table mà không ảnh hưởng gì tới các hiệu ứng css, javascript của table đó, cần sử dụng công nghê Datatables.
**Chi tiết**:
- Phía back-end sẽ trả về chuỗi json là thành phần html của table.

```
def manage_user_data(request):
    if request.session.has_key('agent')and(Agents.objects.get(username=request.session['agent'])).status == 1:
        users = Users.objects.all()
        data = []
        for us in users:
            if us.status == 0:
                st = r'''<p id="stt''' + str(us.id) +'''"><span class="label label-danger">inactive</span></p>'''
                option = r'''<p id="button''' + str(us.id) +'''"><button id="''' + str(us.id) + '''" class="unblock btn btn-success" type="button" data-toggle="tooltip" title="unblock" ><span class="glyphicon glyphicon glyphicon-ok" ></span> Unblock</button></p>'''
            else:
                st = r'''<p id="stt''' + str(us.id) +'''"><span class="label label-success">active</span></p>'''
                option = r'''<p id="button''' + str(us.id) +'''"><button id="''' + str(us.id) + '''" class="block btn btn-danger" type="button" data-toggle="tooltip" title="block" ><span class="glyphicon glyphicon-lock" ></span> Block</button></p>'''
            data.append([us.id, us.fullname, us.email, us.username, st, str(us.created)[:-13], option])
        ticket = {"data": data}
        tickets = json.loads(json.dumps(ticket))
        return JsonResponse(tickets, safe=False)
```

- Phía font-end sẽ dùng ajax để get chuỗi json đó và show ra.

```
$(document).ready(function(){
    $('#list_user').DataTable({
        "columnDefs": [
            { "width": "5%", "targets": 0 },
            { "width": "20%", "targets": 1 },
            { "width": "25%", "targets": 2 },
            { "width": "10%", "targets": 3 },
            { "width": "5%", "targets": 4 },
            { "width": "20%", "targets": 5 },
            { "width": "15%", "targets": 6 },
        ],
        "ajax": {
            "type": "GET",
            "url": location.href +"data",
            "contentType": "application/json; charset=utf-8",
            "data": function(result){
                return JSON.stringify(result);
            }
        },
        "lengthMenu": [[10, 25, 50, -1], [10, 25, 50, "All"]],
        "order": [[ 0, "desc" ]],
        "displayLength": 25,
    });
```

- Khi có message từ websocket thông báo có sự thay đổi,cần load lại tables thì dùng lệnh `$("#list_user").DataTable().ajax.reload();`

3. Để project có thể kết nối tới database,cần cấu hình trong file setting.py và tạo user,cấp quyền cho user trên database server, check port 3306

4. Để sử dụng được các event jquery sau khi load lại trang,cần gọi từ thẻ cha của chúng trước
- Ví dụ: 
```
$("body").on('click', '.handle_done', function(){
        var id = $(this).attr('id');
        var token = $("input[name=csrfmiddlewaretoken]").val();
        if(confirm("Are you sure ?")){
...
```
5. Quá trình deploy bằng docker
- Gồm 4 container: nginx, database, redis và web
- Trong đó, nginx nat port 80 của nó ra bên ngoài, database mởi port 3306, redis mở port 6379 cho web chọc vào, web mở port 8000 chạy code để service gunicorn liên kết giữa nginx và web, web mởi port 8001 để chạy `daphne` phục vụ websocket service 

## Một số lưu ý javascrip
### 1. Để load lại 1 table, cần load lại đúng id table bảng đó và đặc biệt các bảng không được trùng id.
cú pháp: $("body #list_agent_leader").load(location.href + " #list_agent_leader");
### 2. Dùng tùy chọn 'complete' ở đoạn ajax get datatables để chạy một số function nếu cầu sau khi get data thành công.
ví dụ:
```
$('body .tk_table').each( function(){
        var topicname = $(this).attr('id').split('__')[1];
        $(this).DataTable({
            "columnDefs": [
                { "width": "5%", "targets": 0 },
                { "width": "12%", "targets": 1 },
                { "width": "10%", "targets": 2 },
                { "width": "10%", "targets": 3 },
                { "width": "8%", "targets": 4 },
                { "width": "11%", "targets": 5 },
                { "width": "8%", "targets": 6 },
                { "width": "8%", "targets": 7 },
                { "width": "10%", "targets": 8 },
            ],
            "ajax": {
                "type": "GET",
                "url": location.href +"data/" + topicname,
                "contentType": "application/json; charset=utf-8",
                "data": function(result){
                    return JSON.stringify(result);
                },
                "complete": function(){
                    setTimeout(function(){
                        countdowntime();
                    }, 1000);
                }
            },
            "lengthMenu": [[10, 25, 50, -1], [10, 25, 50, "All"]],
            "order": [[ 0, "desc" ]],
            "displayLength": 25,
            'dom': 'Rlfrtip',
        });
        
    });
```
### 3. Một ví dụ nếu muốn lại file script nào đó

```
function loadScript(src, onload) {
        var script = document.createElement('script');
        script.src = src;
        script.async = true;
        document.documentElement.appendChild(script);
        console.log('loaded', src);
    }

    typeof getScreenId === 'undefined' && loadScript('https://cdn.webrtc-experiment.com/getScreenId.js');
 ```
 
 ### 4. Thêm 1 cách show data phía client.
- Server đẩy dữ liệu html về qua HttpResponse. Ví dụ:
```
def group_data(request, lop):
    user = request.user
    if user.is_authenticated and user.position == 1:
        # data = []
        ls_nhom = Nhom.objects.filter(myuser_id=user, lop_id=Lop.objects.get(ten=lop))
        html = ''
        for lsg in ls_nhom:
            html += '''
                <div class="col-md-4 col-sm-4 col-xs-12 profile_details" >
                            <div class="well profile_view">
                                <div class="col-sm-12">
                                <h4 class="brief"><i>'''+lsg.ten_nhom+'''</i></h4>
                                <div class="left col-xs-7">
                                    <h2>Thành viên</h2>
                                    <ul class="list-unstyled">
            '''
            for std in ChiTietNhom.objects.filter(nhom_id=lsg):
                html += '''<li id="drag1" draggable="true"><i class="fa fa-user"></i>'''+std.myuser_id.fullname+'''</li>'''
            html += '''</ul>
                              </div>
                              <div class="right col-xs-5 text-center">
                                <img src="/static/images/img.jpg" alt="" class="img-circle img-responsive">
                              </div>
                            </div>
                            <div class="col-xs-12 bottom text-center">
                              <div class="col-xs-12 col-sm-6 emphasis">
                              </div>
                              <div class="col-xs-12 col-sm-6 emphasis">
                                <button type="button" class="btn btn-danger btn-xs delete_gr" name="'''+str(lsg.id)+'''">Xóa</button>
                                <button type="button" class="btn btn-primary btn-xs" data-toggle="modal" data-target="#chinhsua" name="dang">
                                  Chỉnh sửa
                                </button>
                              </div>
                            </div>
                          </div>
                        </div>'''
            # data.append(html)

        # json_data = json.loads(json.dumps({"data": data}))
        return HttpResponse(html)
```
- `urls.py`: `path('group_data/<str:lop>', views.group_data, name='group_data'),`
- phía template client tạo 1 thẻ div rỗng để nhận dữ liệu. ví dụ:
```
<div id="list_group">
</div>
```
- Phía client load Jquery như sau:
```
function reload(){
        $('body #list_group').html('');
        $.ajax({
            type:'GET',
            url: "/group_data/"+class_,
            success: function(data){
                $('body #list_group').html(data);
                $('body .delete_gr').on('click',function(){
                    var token = $("input[name=csrfmiddlewaretoken]").val();
                    var groupid = $(this).attr('name');
                    var r = confirm('Bạn chắc chắn xóa?');
                    if (r == true){
                        $.ajax({
                            type:'POST',
                            url:location.href,
                            data: {'delete_group':groupid, 'csrfmiddlewaretoken':token},
                            success: function(){
                                reload();
                            }
                       });
                    }
                });
            }
        });
    }
    reload();
```
### 5. Tạo sự kiện chỉ của thẻ cha, không ảnh hưởng tới thẻ con. Ví dụ như sau:
- html:
```
<div class='foobar'> .foobar (alert) 
  <span>child (no alert)</span>
</div>
```
- Jquery:
```
$('.foobar').on('click', function(e) {
  if (e.target !== this)
    return;
  
  alert( 'clicked the foobar' );
});
```
### 6. Tạo sự kiện chỉ thẻ con, không ảnh hưởng tới thẻ cha. Vi dụ:
- html:
```
<div class="header">
    <a href="link.html">some link</a>
    <ul class="children">
        <li>some list</li>
    </ul>
</div>
```
- Jquery:
```
$(document).ready(function(){
    $(".header").click(function(){
        $(this).children(".children").toggle();
    });
   $(".header a").click(function(e) {
        e.stopPropagation();
   });
});
```
- Link tham khhttp://api.jquery.com/event.stopPropagation/
### 7. Cơ chế check user online trên Django.
- Sử dụng cache để lưu lại request từ user tới server. cache sẽ lưu các thông tin của request: key (thường là username của user), thời điểm server nhận request, thời gian user không hoạt đông trước khi kết luận user đó offline.
- Cài đặt:

`pip3 install python-memcached`
`sudo apt install memcached`

- Tạo file `middleware.py` tại một app trong project
```
import datetime
from django.core.cache import cache
from django.conf import settings

class ActiveUserMiddleware(object):
    def __init__(self, get_response):
        self.get_response = get_response
        # One-time configuration and initialization.

    def __call__(self, request):
        # Code to be executed for each request before
        # the view (and later middleware) are called.

        response = self.get_response(request)

        # Code to be executed for each request/response after
        # the view is called.
        try:
            current_user = request.user
            if request.user.is_authenticated:
                now = datetime.datetime.now()
                cache.set('seen_%s' % (current_user.username), now, settings.USER_LASTSEEN_TIMEOUT)
        except:
            pass
        return response
```
- thêm vào file `setting.py`
```
MIDDLEWARE = [
    'SmartClass.middleware.ActiveUserMiddleware',
    ...
]
```

```
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
        'LOCATION': '127.0.0.1:11211',              
    }
}
# Number of seconds of inactivity before a user is marked offline
USER_ONLINE_TIMEOUT = 20

# Number of seconds that we will keep track of inactive users for before 
# their last seen is removed from the cache
USER_LASTSEEN_TIMEOUT = 60 * 60 * 24 * 7
```

- Thêm các hàm vào class định nghĩa user. Ví dụ:
```
class MyUser(AbstractBaseUser):
    email = models.EmailField(
        verbose_name='email address',
        max_length=255,
        unique=True,
    )
    fullname = models.CharField(max_length=100)
    username = models.CharField(max_length=100, unique=True)
    is_active = models.BooleanField(default=True)
    position = models.IntegerField(default=0)
    # 0 : student
    # 1 : teacher
    # 2 : admin
    truong_id = models.ForeignKey('Truong', models.CASCADE, db_column='truong_id', null=True)
    gioi_tinh = models.IntegerField(null=True)

    objects = MyUserManager()

    USERNAME_FIELD = 'username'

    class Meta:
        managed = True
        db_table = 'my_user'

    def __str__(self):
        return self.email

    def has_perm(self, perm, obj=None):
        "Does the user have a specific permission?"
        # Simplest possible answer: Yes, always
        return True

    def has_module_perms(self, app_label):
        "Does the user have permissions to view the app `app_label`?"
        # Simplest possible answer: Yes, always
        return True

    @property
    def is_staff(self):
        "Is the user a member of staff?"
        # Simplest possible answer: All admins are staff
        return self.is_admin

    def last_seen(self):
        return cache.get('seen_%s' % self.username)

    def online(self):
        if self.last_seen():
            now = datetime.datetime.now()
            if now > self.last_seen() + datetime.timedelta(
                        seconds=settings.USER_ONLINE_TIMEOUT):
                return False
            else:
                return True
        else:
            return False
```
- **Lưu ý: phải import cái này vào**
```
from django.core.cache import cache 
import datetime
from SmartClass import settings
```
- Show ở phía template:
```
{{}}
```
