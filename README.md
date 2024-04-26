
# BLOG

## KNOWLEDGE ANNOTION

### @Autowired and @Bean

```java

- bean là những module chính của chương trình, 
được tạo ra và quản lý bởi Spring IoC container.

- Các bean có thể phụ thuộc lẫn nhau, như ví dụ về DisplayHomeController, 
CountService và countComponent.
Sự phụ thuộc này được mô tả cho IoC 
biết nhờ cơ chế Dependency injection.

- Dùng @Component lên class là class đó là một bean.

- Khi ứng dụng Spring chạy, Spring IoC container sẽ quét toàn bộ packages, 
tìm ra các bean và đưa vào ApplicationContext. Cơ chế đó là Component scan


Ví dụ về bean và autowired: 

@Service
@Component
//Khi chúng ta có nhiều bean cùng loại, 
//chúng ta sử dụng @Primary để ưu tiên cao 
//hơn cho một loại bean cụ thể
@Primary
@Qualifier("load")
public class countComponent implements CountService {

    private int  loadCounter;

    @Bean(name = "load")
    @Override
    public int LoadCount() {

        return loadCounter++;
    }
}

@Service
@Component
@Qualifier("load2")
public class countComponent2 implements CountService {

    private int  loadCounter;
    @Bean(name = "load2")
    @Override
    public int LoadCount() {

        loadCounter++;

        return loadCounter;
    }
}

@Service
@Component
@Qualifier("load3")
public class countComponent3 implements CountService {


    @Bean(name = "load3")
    @Override
    public int LoadCount() {
        return 3;
    }
}

@Getter
@Setter
@Controller
public class DisplayHomeController {

    private CategoryJobService categoryJobService;
    private LaborService laborService;
    private JobService jobService;
    private JobDetailService jobDetailService;
    private JobDetailRepo jobDetailRepo;
    private LaborRepo laborRepo;
    private PostService postService;

    //countComponent countComponent2, countComponent3 là các bean
    // tương dương new countComponent();, countComponent2();, countComponent3();
    @Qualifier("load")
    private CountService countService;
    @Qualifier("load2")
    private CountService countService2;
    @Qualifier("load3")
    private CountService countService3;

    //Sử dụng annotation @Autowired để báo cho Spring 
    //biết tự động tìm và inject bean phù hợp vào vị 
    //trí đặt annotation.
    @Autowired//inject
    public DisplayHomeController(CountService countService, CountService countService2, CountService countService3) {
        this.countService = countService;
        this.countService2 = countService2;
        this.countService3 = countService3;
     
    }

    @GetMapping("/")
    private String index(){

        System.out.println("Bean 1: "+countService.LoadCount());
        System.out.println("Bean 2: "+countService2.LoadCount());
        System.out.println("Bean 3: "+countService3.LoadCount());
    }
}

```

### Bean life cycle (Vòng đời của bean)

```js

- Khởi tạo (Instantiation): Trong giai đoạn này, 
Spring tạo ra một phiên bản mới của bean. 
Điều này có thể xảy ra thông qua việc tạo một đối 
tượng mới hoặc thông qua việc lấy một đối tượng từ một nơi khác,
chẳng hạn từ một pool đối tượng.

- Thiết lập các thuộc tính (Populate Properties): Sau khi bean được tạo ra, 
Spring sẽ thiết lập các giá trị cho các thuộc tính của bean, 
bao gồm các dependency được inject thông qua các annotation như @Autowired.

- Gọi các callback (Calling Callbacks): Sau khi các thuộc tính đã được thiết lập, 
Spring sẽ gọi các callback để cho phép bean thực hiện các thao tác khởi tạo hoặc tùy chỉnh.
Các callback này có thể là @PostConstruct (được gọi 
sau khi bean được khởi tạo và tất cả các thuộc tính đã được thiết lập) 
hoặc các phương thức khác do người dùng định nghĩa.

- Sử dụng bean (Bean Ready for Use): 
Bean ở trong trạng thái sẵn sàng để sử dụng.
Ở giai đoạn này, bean có thể được chia sẻ và 
sử dụng bởi các thành phần khác trong ứng dụng.

- Hủy bỏ bean (Bean Destruction): Khi ứng dụng hoặc container Spring bị đóng, 
hoặc khi không còn cần thiết nữa, Spring sẽ hủy bỏ bean. 
Trong giai đoạn này, Spring gọi các callback hủy bỏ để cho
phép bean thực hiện các tác vụ dọn dẹp hoặc giải phóng tài nguyên.

ví dụ:

import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;
import org.springframework.stereotype.Service;

@Service
public class TaskService {

    private ExternalNotificationService notificationService;

    @PostConstruct
    public void init() {
        // Kết nối đến hệ thống thông báo khi bean được khởi tạo
        notificationService = new ExternalNotificationService();
        notificationService.connect();
        System.out.println("TaskService initialized");
    }

    public void createTask(Task task) {
        // Thực hiện tạo tác vụ và gửi thông báo
        // Sử dụng notificationService để gửi thông báo
        notificationService.sendNotification("New task created: " + task.getTitle());
    }

    // Các phương thức khác của TaskService
    @PreDestroy
    public void cleanup() {
        // Đóng kết nối đến hệ thống thông báo khi bean bị hủy bỏ
        if (notificationService != null) {
            notificationService.disconnect();
        }
        System.out.println("TaskService cleanup");
    }
}


```

## KNOWLEDGE RESTFUL API 

### Life cycle restful api (Vòng đời của restful api)
```javascript

- Khởi tạo (Initialization):
+ Trong giai đoạn này, server khởi động và chuẩn bị các tài nguyên cần thiết, 
bao gồm kết nối tới cơ sở dữ liệu, thiết lập các middleware, 
và các tài nguyên khác cần thiết cho hoạt động của API.

- Xử lý Yêu cầu (Request Processing):
+ Khi một yêu cầu được gửi đến API, server nhận và xử lý yêu cầu đó. 
Trong quá trình này, các middleware có thể được áp dụng để thực hiện xác thực, 
phân quyền, kiểm tra lỗi, và các chức năng khác ...

- Xử lý Logic Kinh doanh (Business Logic Processing):
+ Sau khi yêu cầu được xác thực và xác định là hợp lệ, 
server sẽ thực hiện logic theo yêu cầu trong ứng dụng

- Trả về Phản hồi (Response Return):
+ Khi xử lý logic xong, server sẽ trả về phản hồi, dữ liệu cho client. 

- Kết thúc (Termination):
+ Sau khi trả về phản hồi server hoàn thành quá trình xử lý 
yêu cầu và chờ đợi yêu cầu tiếp theo từ client.

```

## KNOWLEDGE REACT COMPONENT

### Component Life cyles (Vòng đời của Component)
```js

- Life cycle của component trong reactjs là quá trình từ khi tạo ra, 
thay đổi và hủy bỏ component. Gồm 3 giai đoạn:

+ Tạo ra (Mounting)
- Ví dụ:
import React, { useState } from 'react';

const View = () => {
  

  return (
    <div>
        Hello world!
    </div>

  );
};

const App = () => {

  const [show, setShow] = useState(0);

  return (
    <div>
      <button onClick={()=> setShow(!show)}>Show</button>
      {show && <View/>}
    </div>

  );
};

export default Counter;

- Trong ví dụ này khi bấm Show component
View sẽ được mounted và render ra giao diện


+ Thay đổi (Updating)
- Ví dụ:
import React, { useState } from 'react';

const Counter = () => {
  const [count, setCount] = useState(0);

  const increment = () => {
    setCount(prevCount => prevCount + 1);
  };

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>Increment</button>
    </div>
  );
};

export default Counter;
- Trong trường hợp này mỗi khi click Increment biến count sẽ
được update giá trị mới và component sẽ updating (re render)
lại và thay đổi giá trị biến count trên giao diện


+ Hủy bỏ (UnMounting)
- Ví dụ: 

import React, { useState } from 'react';

import React, { useState, useEffect } from 'react';

const SlowComponent = () => {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const fetchData = async () => {
      try {
        // Một tác vụ chậm, ví dụ: gửi request mạng
        const response = await fetch('https://api.example.com/slow-request');
        const data = await response.json();
        console.log(data);
        setCount(count + 1);
      } catch (error) {
        console.error('Error:', error);
      }
    };

    fetchData();

    // Cleanup khi component unmount
    return () => {
      // Hủy bỏ các tài nguyên không cần thiết ở đây, ví dụ: event listeners, subscriptions
      console.log('Cleanup executed');
    };
  }, []); // Tham số thứ hai là mảng rỗng, chỉ chạy một lần sau khi component mount

  return (
    <div>
      <p>Count: {count}</p>
    </div>
  );
};

const App = () => {

  const [show, setShow] = useState(0);

  return (
    <div>
      <button onClick={()=> setShow(!show)}>Show</button>
      {show && <SlowComponent/>}
    </div>

  );
};

- Trong ví dụ này khi bấm "Show" thì SlowComponent
sẽ được mount và render ra giao diện sau đó 
bấm tiếp "Make Slow Request" thì hàm fetchData sẽ chạy
và 1 request sẽ gửi đi trong thời gian request gửi đi
nếu bấm tiếp "Show" thì SlowComponent sẽ bị (unmount) ẩn đi 
lúc này nếu k có cleanup trong hàm useEffect thì
hàm fetchData vẫn hoạt động điều này sẽ làm tiêu
tốn tài nguyên của ứng dụng

```

## KNOWLEDGE HOOKS IN REACT JS

### useState hook
```javascript

- Khi sử dụng useState react sẽ cho phép component render lại 
và thay đổi lại giá trị mà chúng ta muốn hiển thị ra giao diện.
Còn nếu thực hiện kiểu gán biến bình thường thì component sẽ 
không re render và giá trị mà chúng ta muốn hiển thị 
ra giao diện sẽ không thay đổi
ví dụ:

import React, { useState } from 'react';

function View() {
  //giá trị mặc định của biến name sẽ là Tuong
  const [name, setName] = useState("Tuong");

  return (
    <div>
      <p>My name: {name}</p>
      <button onClick={() =>{

         setName("Tran The Tuong");// sử dụng setName thì biến name sẽ cập nhật
         // lại thành Tran The Tuong và component sẽ re render lại và hiển thị 
         // ra Tran The Tuong
         //name = "Tran The Tuong" còn nếu làm thế này thì biến name vẫn sẽ
         //cập nhật thành Tran The Tuong tuy nhiên component sẽ không re render 
         // nên giá trị hiển thị ra vẫn là Tuong
      }}>
        Click me
      </button>
    </div>
  );
}


```

### useEffect hook

```js
- Khi sử dụng useEffect chúng sẽ phải truyền vào hàm callback
- Hàm callback trong useEffect sẽ luôn được gọi khi
component mounted re render
- Khi dùng useEffect sẽ có 3 trường hợp mà chúng ta sử dụng đến đó 
là những trg hợp sau:
useEffect(()=>{});
+ sử dụng useEffect chỉ truyền vào hàm callback

useEffect(()=>{}, []);
+ sử dụng useEffect truyền vào 1 mảng rỗng

useEffect(()=>{}, [depen]);
+ sử dụng useEffect truyền vào 1 mảng chứa các dependency
(sự phụ thuộc)
+ Hàm callback sẽ được gọi lại mỗi khi dependency thay đổi

+ ví dụ: 

import React, { useState } from 'react';

function View() {
  const [name, setName] = useState("");
   const [type, setType] = useState("posts");

  // Mỗi lần gõ vào input component 
  // sẽ (mounted) re render lại và
  // hàm callback sẽ được gọi
  // Hàm useEffect sẽ thực thi sau khối lệnh return
  useEffect(()=>{
    // gõ vào input 10 lần Mounted sẽ in ra 10 lần
    console.log("Mounted");
  });

  useEffect(() => {
      //Mounted sẽ in ra 1 lần mỗi khi component mounted và render
      console.log("Mounted!!!");
  }, []);

   useEffect(() => {
        //api sẽ được gọi mỗi khi component mount
        // và mỗi lần biến type sẽ thay đổi
        // api sẽ được gọi trong trg hợp này
        // biến type đã đc truyền vào là dependency
        //api sẽ mặc định gọi posts vì biến type
        //có giá trị mặc định là posts
        //khi type thay đổi component sẽ re render ra
        //giá trị thay đổi theo biến type
        fetch("https://jsonplaceholder.typicode.com/" + type)
        .then(res=> res.json())
        .then(apis=>{
            setApi(apis);
        });
        //trường hợp này change name vẫn sẽ in ra mỗi khi
        //component mount tuy nhiên khi 1 biến khác không
        //phải dependency là biến name mà thay đổi thì
        //component sẽ re render nhưng change name không được
        //in ra còn nếu biến name thay đổi thì change name
        //sẽ được in ra ví dụ:
        // gõ vào input name 10 lần thì change name sẽ in ra 10 lần
        console.log("change name");

  }, [type]);

  //trường hợp này Mounted vẫn sẽ in ra mỗi khi
  //component re render nó sẽ được gọi
  // trước khối lệnh return
  // Tuy nhiên nếu khối lệnh gọi trc này lỗi
  // điều đó sẽ làm lỗi ứng dụng
  //đây là 1 ví dụ:
  console.log(a); 
  let a = 1;
  console.log("Mounted");

  return (
    <div>
        {tabs.map(tab=>
              <button style={type === tab ? {
                  background: "#333",
                  color: 'white'
                  } : {}} onClick={()=>{
                      setType(tab);
                  }} key={tab}>
                  {tab}
              </button>
        )}
        {/*Khối lệnh thực thi này sẽ thực thi trước useEffect*/}
        {console.log(name)}
       <input value={title} onChange={e => setName(e.target.value)}/>
    </div>
  );
}

```

## Ask and Answer
```js
1. Onclick trên html khác gì với onclick trên JS
=>
2. Submit form có những cách nào ? (Onsubmit và onclick)
=> Onsubmit và onclick
3. Xử lý bất đồng bộ trong call API như thế nào ?
=> cách xử lý bất đồng bộ trong call API là sử dụng promise trong promise sẽ có 3 hàm hander là .then(), .catch() và .finally() 
- .then() sẽ chạy khi Promise thành công. Đôi khi bạn cũng có thể sử dụng để xử lý trạng thái thất bại của Promise.
- .catch() sẽ chạy chỉ khi Promise thất bại.
- .finally() sẽ chạy khi Promise chạy xong, không quan trọng là thành công hay thất bại.
- sử dụng Async/await
- Cách cuối cùng để xử lý bất đồng bộ là sử dụng async/await. Async/await được giới thiệu ở ES8, chúng được cấu tạo từ 2 phần. Phần đầu tiên là function async. Hàm này sẽ được tự động thực thi bất đồng bộ. Giá trị nó trả về là một Promise. Vì trả về Promise nên bạn sẽ phải sử dụng các handler của Promise để xử lý giá trị này.

- Phần thứ hai của async/await là operator await. Operator này sẽ được dùng cùng với một Promise. Nó sẽ khiến cho function async tạm dừng cho đến khi Promise đó chạy xong. Ngay sau đó nó sẽ lấy gía trị của Promise mà cho function async tiếp tục chạy.

- Các function async đều bất đồng bộ, khi bị pause bởi await thì phần code còn lại vẫn chạy bình thường vì function đấy không block main thread. Khi Promise chạy xong thì hàm async sẽ chạy tiếp và trả về giá trị của Promise.

- Một điều quan trọng hơn là await phải được viết trong hàm async nếu không thì sẽ gặp lỗi syntax error.
- example : 
        async function makeAPICall() {
        // Show notification about API call
            console.log('Bắt đầu gọi API.');

        // Create a Promise to make the API call
        const dataPromise = new Promise((resolve, reject) => {
            setTimeout(() => {
            console.log('Nhận data từ API.');
            resolve('Gọi API thành công.');
            }, 2000);
        });

        return dataReceived = await dataPromise
        }

        function processAPIData() {
        console.log('Xử lý data');
        };

        function readTheData() {
        console.log('Đọc data.');
        };

        // Hàm này sẽ chạy ngay sau async makeAPICall
        // và chạy trước tất cả các hàm khác.
        function someOtherFunction() {
        console.log('Some other function not related to API.')
        }

        makeAPICall
        // add handle cho status thành công
        .then(result => {
            console.log(result);
            processAPIData();
            readTheData();
        })
        // add handle cho status thất bại
        .catch(error => {
            console.log(`Có lỗi xảy ra: ${error}.`)
        })
        // bạn có thể thêm finally() ở đây
        // .finally(() => {})

        someOtherFunction();

4. Submit form bằng onsubmit thì data gửi về server được tạo ra bằng cách nào ?
=>
5. Cho bài toán : a = [1,2,3,4] ; b =a ; a[0] = 5 ; hỏi b[0] = mấy ? Giải thích ?
=> b gán cho a và a[0] = 5 thì b[0] = 5
6. GET khác POST chỗ nào ? Có thể dùng POST bằng GET được không ?
=> GET khác POST ở chỗ là GET là dùng để lấy dữ liệu (data) còn POST là push thêm dữ liệu
7. Truthy và falsy ?
=>
8. Kết quả của (true && false || true || false && true) là gì ?
=> true, vì sẽ ưu tiên lấy true nên cặp 1 là true && false lấy true, 2 là true, 3 là false && true sẽ lấy true nên => true || true || true. Kết quả bằng = true
```