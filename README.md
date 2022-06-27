

Chào mừng các bạn tới bài viết **High Performance iOS Apps** và cùng mình tìm hiểu XCode Instrument để optimize sourcecode đạt hiệu suất cao hơn.


# I. **AppPerformance Defining**

Trước khi đi vào khái niệm thì cùng mình xem qua một số thống kê sau:

- 79% người dùng chỉ dùng lại app 1-2 lần nếu nó không hoạt động trong lần đầu tiên.
- 25% người dùng sẽ xóa ứng dụng nếu ứng dụng đó không loading trong 3s.
- 31% người dùng sẽ nói với người khác về trải nghiệm tồi tệ của họ về ứng dụng nào đó.

Các thống kê ở trên chỉ ra rõ ràng một vấn đề là: Nếu người dùng có trải nghiệm không tốt trong lần sử dụng đầu tiên hoặc lần thứ 2, họ khó có thể tiếp tục sử dụng ứng dụng đó.

Hiệu suất ứng dụng là một khái niệm khá mơ hồ và bị thuộc vào nhiều yếu tố ví dụ như phần cứng, bandwitdh, network, hệ thống phản hồi. Các yếu tố đó có thể được đo lường để đánh giá hiệu suất ứng dụng tốt hay xấu.
Ngoài ra còn có một khái niệm Perfomance Metrics là các thuộc tính hướng tới người dùng. Mỗi thuộc tính có thể là một yếu tố hoặc thông số kĩ thuật ảnh hưởng trực tiếp tới hiệu suất của ứng dụng.

Cùng xem qua các nhân tố đó là gì nhé:

1. 	Memory
2. Power Consumption
3. Execution Speed
4. Initialization Time
5. Responsiveness
6. Local Storage
7. Interoperability
8. Network Condition
6. Bandwidth
7. Data Refresh
8. Crashes
9. Security


## I. Memory

Bộ nhớ là một tài nguyên có giới hạn, một ứng dụng có thể bị terminated nếu nó vượt quá tài nguyên cho phép. Do đó quản lý bộ nhớ đóng vai trò trung tâm trong triển khai ứng dụng iOS.

Trong section này chúng ta sẽ nghiên cứu các vấn đề sau:

- Tiêu thụ bộ nhớ
- Quản lý bộ nhớ (ARC)
- Retain Cycles

1. Tiêu thụ bộ nhớ

Có 2 thành phần tiêu thụ bộ nhớ trong ứng dụng là: Stack size mà Heap size.

**StackSize:**

* Phân bổ bộ nhớ theo ngăn xếp(LIFO).
* Khi kết thúc hàm, các biến cục bộ lưu trữ ở stack được tự động giải phóng.
* Kích thước vùng nhớ stack là cố định.
* Thông thường Stack dùng để cấp phát bộ nhớ cho các func parameter hoặc local variables.

**HeapSize:**

* Heap được dùng cho việc cấp phát bộ nhớ động, kích thước vùng nhớ heap có thể tăng giảm nhờ cơ chế của Hệ điều hành.
* Khi nhắc tới Heap chúng ta hay liên tưởng đến việc cấp phát bộ nhớ cho các instance của class.
*

2. Quản lý bộ nhớ (ARC)

Swift sử dụng ARC(Automatic Reference Counting) để quản lý việc sử dụng bộ nhớ. Khi tạo một instance của một class. ARC sẽ tự động cung cấp một vùng nhớ để lưu thông tin và dữ liệu của instance. Khi không còn sử dụng, ARC sẽ giải phóng vùng nhớ đó.

Khi một instance được tạo, thì reference counting sẽ bằng 1. Reference counting này có thể tăng hoặc giảm trong vòng đời của nó. Cuối cùng, khi reference counting bằng 0, đối tượng được giải phóng khỏi bộ nhớ. Trong một số trường hợp, các đối tượng chúng ta đã không còn sử dụng đến nữa, nhưng nó vẫn có reference đến nhau, do đó số reference counting khác 0, và lúc này chúng ta sẽ bị memory leaks.

3. Retain Cycles

Trong Swift, khi một instance có một strong reference với một instance khác, nó sẽ giữ lại nó (retain). Để hiểu rõ Retain Cycle mình có ví dụ sau:

~~~swift
class A {
   var b: B? = nil
   
   init() {
      print("init A")
   } 
   deinit {
      print("deinit A")
   }
}
class B {
   var a: A? = nil
   
   init() {
      print("init B")
   }
   deinit {
      print("deinit B")
   }
}
var a = new A()
var b = new B()
a?.b = b
b?.a = a
~~~

Ta thấy instance của class A sẽ có reference tới instance của class B và ngược lại. Khi set:

~~~swift
a = nil
b = nil
~~~

Ta gọi việc các instance của Class có strong reference lẫn nhau là retain cycle.

Vậy có cách nào để a và b sẽ được giải phóng khỏi bộ nhớ?

> Swift cung cấp hai cách để giải quyết các strong reference: weak reference và unowned reference. weak reference và unowned reference cho phép một instance tham chiếu đến instance khác mà không giữ strong reference trên nó.

* Weak: Một weak reference là khi một biến không sở hữu một đối tượng. Nó luôn được khai báo là optional vì nó có thể là nil. Khi instance được giải phóng, ARC sẽ tự động gán tham chiếu weak là nil. Cho nên, weak được khai báo với var
* 
Unowned: Unowned khá giống với weak khi nó có thể sử dụng để phá vỡ strong reference. Tuy nhiên, không giống như weak, Unowned luôn có một giá trị.

**Một số cách kiểm tra Memory leak**

* Kiểm tra trong hàm deinit() khi ViewController bị đóng.
* Quan sát mức độ bộ nhớ tăng dần trên Xcode.
* Sử dụng tool Xcode Instruments

## II. Initialization Time

Một ứng dụng chỉ nên thực hiện vừa đủ các task tại thời điểm launch app để người dùng có thể sử dụng nó. Dưới đây là một số ví dụ nên thực hiện trong quá trình launch app (không theo thứ tự cụ thể) mà các bạn có thể tham khảo:

* Kiểm tra xem ứng dụng có đang được khởi chạy lần đầu tiên hay không.
* Kiểm tra xem người dùng đã đăng nhập chưa.
* Nếu người dùng đã đăng nhập, hãy tải trạng thái trước đó, nếu có.
* Fetch những thay đổi mới nhất từ server
* Kiểm tra xem app có launch từ deeplink. Nếu có, load UI và trạng thái phù hợp
* Kiểm tra xem có task nào pending từ lần cuối sử dụng app không, tiếp tục task đó nếu cần
* Khởi tạo các object và thread để sử dụng sau này
* Khởi tạo các dependencies (crash analytics, cache, object relational mapping)

## III. Responsiveness

Sau khi người dùng mở một ứng dụng thì họ thường mong muốn ứng dụng phản hồi nhanh nhất có thể. Chúng ta sẽ sử dụng concurrency để làm cho user interface responsive. Một trong những lợi ích của concurrency là giúp chúng ta tối ưu được phần cứng của thiết bị. Ngày nay tất cả iOS devices đều có nhiều core, do đó cho phép những developers chạy nhiều task song song. Điều này không chỉ tối ưu mà còn lấy được những hiệu năng từ hardware. 

# **Thread**
Thread là một chuỗi các lệnh có thể chạy tại một thời điểm

Hiểu đơn giản, threading là việc bạn quản lý những tác vụ nào ưu tiên trong app. Làm cho nhưng dòng code bạn viết ra chạy nhanh hơn thì rất là tuyệt, nhưng nó sẽ còn tuyệt hơn nữa nếu những user của bạn nhận thức được cái app của bạn chạy nhanh hơn so với các app khác.

Là 1 iOS dev thì mục tiêu của chúng ta luôn là ưu tiên về UX và UI (những thứ mà user có thể thấy và tương tác), nó sẽ 1 phần nào đó làm cho user nghĩ rằng: “App này chạy nhanh và xử lý tốt”. Đừng bao giờ để user phải đợi load 1 thứ gì đó lên màn hình quá lâu nhưng mà thứ đó họ lại không quan tâm lắm.


# **GCD**

GCD là viết tắt của Grand Central Dispatch. Đây là một low-level API chịu trách nhiệm quản lý các queue trong Swift. Chúng ta cùng xem qua GCD cung cấp những gì:

* Thực thi các task theo concurrent hoặc serial
* Group các task và notify khi task được hoàn thành
* Semaphores
* Barriers
* Sync/Aysnc

# **Operation && Queues**

Ngoài GCD thì còn có Operation && Operation Queues giúp quản lý các task.

Operation là một abstract class. Mỗi một Operation sẽ thực hiện một task.

OperationQueue: Đưa Operation đó vào hàng đợi

Operation và OperationQueue đều cung cấp quyền kiểm soát số lượng queue được tạo. Bạn cũng có tể kiểm soát số luồng trong mỗi hàng đợi, sử dụng thuộc tính maxConcurrentOperationCount.


# GCD vs OperationQueue

**GCD** viết tắt của Grand Central Dispatch là một api nó ở mức độ thấp nhất được dựa vào nền tảng của **C** tương tác trực tiếp với Unix các que thực hiện theo cơ chế FIFO chi tiết GCD

**NSOperation** là một class của **ObjectiveC** nó được xây dựng tầng bên trên của **GCD** **NSOperation** nó không giống như GCDnó không theo cơ chế theo thứ tự FIFO đây là một số điểm khác biệt

1. Không theo FIFO: trong operation queues, bạn có thể set độ ưu tiên thực hiện cho operations, và bạn có thể thêm dependencies giữa các operations nó có nghĩa là bạn có thể xác định một số operations sẽ chỉ được thực hiện sau khi hoàn thành operations khác. Đấy là lý do mà nó không theo cơ chế FIFO

2. Operation queue chỉ xử lý theo kiểu concurrent queue mà không xử lý theo kiểu serial queue của GCD. Để xử lý theo kiểu serial queue, chúng ta tạo các ràng buộc cho các operation.

3. Operation queue là kiểu hướng đối tượng, mỗi operation queue là một instance của lớp NSOperationQueue, và mỗi task là một instance của lớp NSOperation. NSOperation cần được phân bổ trước khi chúng có thể được sử dụng và huỷ bỏ khi không cần dùng nữa. Mặc dù đây là một quá trình được tối ưu hoá cao, nhưng nó vẫn chậm GCD

## IV. Network Condition

Các thiết bị mobile có thể được sử dụng trong các điều kiện mạng khác nhau. Để đảm bảo trải nghiệm của người dùng tốt nhất, ứng dụng của bạn phải hoạt động ổn định trong tất cả các trường hợp sau:

• Băng thông cao và mạng ổn định

• Băng thông thấp nhưng mạng ổn định

• Băng thông cao nhưng mạng ngắt quãng

• Băng thông thấp và mạng ngắt quãng

• Không có mạng

**Network Reachability** ở đây hiểu nôm na là trạng thái mái kết nối mạng của device. Chúng ta có thể sử dụng để xác định rằng device đang online hay offline, thậm chí là đang dùng mạng wifi hay gói dữ liệu data từ nhà mạng.

Có rất nhiều cách để kiểm tra việc này trong Swift và cũng có khá nhiều framework được viết ra để hỗ trợ. Các bạn có thể tham khảo các lib bên d handle từng case hoặc thông báo cho nguười dùng về tình trạng mạng để có trải nghiệm tốt hơn.

VD: SCNetworkReachability, Alamofire NetworkReachabilityManager, ReachabilitySwift...

# II. Sử dụng Xcode Instruments để kiểm tra hiệu suất ứng dụng

Trong phần này, mình sẽ giới thiệu với các bạn:

* Instruments là gì và nó bao gồm những gì?
* Cách cài đặt và customize Instruments
* Cách kiểm tra các vấn đề về hiệu suất, bộ nhớ, reference cycles...
* Cách Debug những issue trên

### Bắt đầu

Download sample project mình ghim ở đầu bài nhé. Project này sử dụng Flickr API để tìm kiếm hình ảnh. Để sử dụng được api thì các bạn generate API key theo step bên dưới nhé:

* Tạo tài khoản và login tại đây https://identity.flickr.com
* Tiếp theo, vào Flickr API Explorer
* Tìm và click CallMethod ở cuối page
* Nó sẽ tạo ra một đường link giống như vậy:
`
https://www.flickr.com/services/rest/?method=flickr.photos.search
&api_key=f0589d37afc0e29525f51ccb26932a06
&format=rest
&auth_token=72157717064637163-20d89cb35333d1eb
&api_sig=80ada3ca6dba49f7fcc9ced2743de537
`
* Copy API key ở sau text **&api_key=** và update trong file **FlickrAPI.swift**

Chạy và search vài keyword thì bạn sẽ thấy màn hình như thế này:

Có thể bạn nghĩ app đã chạy ok tuy nhiên sau đây chúng ta cùng tìm hiểu xem Xcode Instrument có thể cải thiện những gì nhé

## TimeProfiler

Đây là hình ảnh của TimeProfiler

Màn hình ở trên được gọi là **CallTree**. CallTree sẽ show thời gian excution các method trong app. Mỗi hàng là một phương thức khác nhau, TimeProfiler sẽ ước tính thời gian cho mỗi method bằng cách đếm số lần dừng lại ở mỗi method

Để mở được TimeProfiler từ Xcode thì các bạn chọn **Product > Profile > TimeProfiler** hoặc ấn **Command - I** . 

Tiếp theo click nút Record ở góc trái trên cùng để bắt đầu launch app. Thực hiện search và scroll màn hình result bạn sẽ thấy nó bị giật và không hề mượt mà. Bây giờ thì cùng tìm nguyên nhân và fix nó nhé.

Mở TimeProfiler và bạn sẽ thấy màn hình bên dưới:

1. Recording controls: Bao gồm các nút stop và start
2. Run timer: Bộ đếm sẽ đếm số lần đã chạy. Như ở trên hình là 2 lần.
3. TimerProfile track: Mình sẽ tìm hiểu phần này ở dưới nhé
4. Detail panel: Show các thông tin mà app đang sử dụng. Như ở trên là method tốn nhiều thời gian của CPU nhất.
5. Inspector panel: gồm 2 phần Extended Detail and Run Info.

Sort lại cột Weight để tìm xem method nào đang tốn nhiều thời gian nhất. Bạn sẽ thấy row Main Thread đang tốn khá nhiều, click vào dấu mũi tên > để xem chi tiết. Bạn lại thấy method **Tonal Filter** bên trong MainThread đang cũng tốn rất nhiều thời gian. Click vào dấu mũi tên > bạn sẽ thấy nó được refer tới `(_:cellForItemAt:)` trong file `SearchResultsViewController.swift`

Rõ ràng vấn đề chúng ta gặp phải là tạo UIImage với filter tonal tốn nhiều thời gian và chúng ta lại gọi nó trong hàm `collectionView(_:cellForItemAt:).` sẽ được gọi mỗi khi chúng ta scroll, đó là lý do mỗi lần scroll thì app sẽ rất giật.

Solve: Do apply UIImage với filter tonal là một task nặng nên chúng ta sẽ đưa nó xuống background thread `DispatchQueue.global().async`. Tiếp theo cache lại từng ảnh nếu nó đã được filter. 

Ok bắt đầu sửa thôi, click vào icon và XCode sẽ đưa bạn tới đúng chỗ cần sửa

Bên trong hàm `collectionView(_:cellForItemAt:)`, replace đoạn code `loadThumbnail(for:completion:) ` bằng: 

```swift
ImageCache.shared.loadThumbnail(for: flickrPhoto) { result in
  switch result {
  case .success(let image):
    if resultsCell.flickrPhoto == flickrPhoto {
      if flickrPhoto.isFavorite {
        resultsCell.imageView.image = image
      } else {
        // 1
        if let cachedImage =
          ImageCache.shared.image(forKey: "\(flickrPhoto.id)-filtered") {
          resultsCell.imageView.image = cachedImage
        } else {
          // 2
          DispatchQueue.global().async {
            if let filteredImage = image.withTonalFilter {
              ImageCache.shared
                .set(filteredImage, forKey: "\(flickrPhoto.id)-filtered")

              DispatchQueue.main.async {
                resultsCell.imageView.image = filteredImage
              }
            }
          }
        }
      }
    }
  case .failure(let error):
    print("Error: \(error)")
  }
}
```

Giải thích đoạn code trên:

1. Kiểm tra xem filteredImage có trong cache không? Nếu có thì hiển thị
2. Nếu chưa có thực hiện apply filter tonal dưới background thread. Khi thực hiện xong thì lưu lại trong cache và update lại ảnh ở main queue.

Chúng ta cũng cần update lại logic trong file `Cache.swift`. Update lại `loadThumbnail(for:completion:)` bằng đoạn code

```swift
func loadThumbnail(
  for photo: FlickrPhoto, 
  completion: @escaping FlickrAPI.FetchImageCompletion
) {
  if let image = ImageCache.shared.image(forKey: photo.id) {
    completion(Result.success(image))
  } else {
    FlickrAPI.loadImage(for: photo, withSize: "m") { result in
      if case .success(let image) = result {
        ImageCache.shared.set(image, forKey: photo.id)
      }
      completion(result)
    }
  }
}
```
Đoạn code trên handle giống filteredImage. Nếu ảnh đã có trong cache, completion với ảnh đã cache. Nếu chưa có thì load ảnh sau đó lưu lại vào cache.

Chạy **Command-I** để run TimeProfiler. Thực hiện search một số keyword, app đã có thể scroll mượt mà hơn. App đã thực hiện filter ở background và cache lại kết quả. Bạn có thể thấy rất nhiều `dispatch_worker_threads` trong **CallTree**.

Tới đây thì bạn đã nghĩ app đã ổn rồi nhỉ. Not yet!

## Allocation

Vẫn còn một số bug tiềm ẩn trong project. Có thể bạn đã biết hoặc đã từng nghe về memory leak. Nhưng có thể bạn chưa biết là memory leak có 2 loại:

* **True memory leak**: Điều này xảy ra khi một object không được tham chiếu bởi bất kì thứ gì nhưng vẫn được cấp phát bộ nhớ. Ngay cả khi Swift ARC giúp quản lý bộ nhớ, phổ biến nhất là retain cycle hoặc strong reference. Xảy ra khi 2 đối tượng cùng strong reference tới nhau dẫn tới không thể deallocated 1 trong 2. Kết quả là bộ nhớ không bao giờ đc giải phóng.
* **Unbounded memory growth**: Điều này xảy ra khi bộ nhớ cấp phát liên tục và không bao giờ deallocated, dẫn tới out of memory. Trong iOS khi xảy ra vấn đề này thì hệ thống sẽ terminate app của bạn.

Bắt đầu giống TimeProfiler, các bạn dùng **Command-I** sau đó chọn **Allocations**. Bạn sẽ thấy giao diện khá giống với **TimeProfiler**.
Ấn vào button record và chú ý phần **All Heap and Anonymous VM**, quay lại app và thực hiện search một số keyword

Bạn sẽ nhận thấy biểu đồ trong **All Heap and Anonymous VM** đang tăng lên. Điều này cho thấy app đang phân bổ bộ nhớ (Allocation). Tính năng Allocations sẽ giúp bạn tìm ra **Unbounded memory growth**.

### Simulating a Memory Warning

Memory warning là một cách mà	hệ thống thông báo cho ứng dụng rằng bộ nhớ của ứng dụng đang ở mức báo động và bạn cần phải giải phóng nó.

Quay trở Allocations chọn phần **Growth**, bạn sẽ thấy rất nhiều object và bạn không biết bắt đầu từ đâu?

Khá easy, chọn **Growth** header để sort lại theo kích thước. Phần nặng nhất chắc chắn sẽ ở đầu tiên. Bạn sẽ thấy có một label tên là **VM: CoreImage**.
Click để xem detail trong **Extended Detail**.

Phần màu xám là các thư viện của hệ thống, còn phần màu đen là code của bạn.
Tới đây bạn lại thấy `collectionView(_:cellForItemAt:)`. Khá quen thuộc đúng không nào?. Double click và Instrument sẽ đưa bạn tới file source code đó. Nó gọi tới `set(_:forKey:)` trong `ImageCache.shared`. Nhớ lại thì method này sẽ cache lại ảnh để tái sử dụng. Đó có thể là vấn đề!

Mở lại file **Cache.swift** và nhìn lại implement của function `set(_:forKey:)`

```swift
func set(_ image: UIImage, forKey key: String) {
  images[key] = image
}

```

Hình ảnh sẽ được add vào dictionary thông qua key là id của photo. Tuy nhiên thì hình ảnh chỉd đc add chứ không được remove. Điều này dẫn tới **Unbounded memory growth**. Mọi thứ chỉ được thêm vào chứ không bị remove đi.

Để fix vấn đề trên thì chúng ta sẽ lắng nghe event Memory warning. Mỗi khi nhận được event thì chúng ta sẽ clear cache. Mở file **Cache.swift** và thêm code vào trong hàm khởi tạo

```swift
init() {
  NotificationCenter.default.addObserver(
    forName: UIApplication.didReceiveMemoryWarningNotification,
    object: nil,
    queue: .main) { [weak self] _ in
      self?.images.removeAll(keepingCapacity: false)
  }
}

```

Đoạn code trên sẽ lắng nghe event UIApplication.didReceiveMemoryWarningNotification và sẽ thực hiện code trong block. Tất cả ảnh trong dictionary sẽ bị remove. Điều này đảm bảo không có dữ liệu trong images và nó sẽ bị deallocated. Chạy lại và nhìn vào graph, bạn sẽ mức độ sử dụng bộ nhớ sẽ giảm đi sau khi nhận memory warning.

Ok, vậy là bạn đã giải quyết được thêm 1 vấn đề nữa. Tuy nhiên vẫn còn một loại memory leak cần được giải quyết.

## **Strong Reference Cycle**

Như đã nói ở trên, Strong Reference Cycle xảy ra khi 2 object cùng giữ tham chiếu mạnh tới nhau, dẫn đến việc cả 2 không thể bị deallocated. Trong phần này mình cùng tìm hiểu cách dùng Allocations để kiểm tra nhé.

Chọn `Product` > `Profile` > `Allocations`.

Lần này chúng ta sẽ không dùng generation để phân tích nữa. Thay vào đó chúng ta sẽ nhìn vào những object tồn tại trong bộ nhớ. Click nút **Record**, sau đó filter theo tên ứng dụng. Ở đây là **InstrumentsTutorial**.

Sẽ có 2 cột đáng chú ý đó là Persistent và Transient. Persistent hiển thị số đối tượng đang tồn tại, còn Transient hiển thị số lượng đối tượng đã deallocated. Các đối tượng trong Persistent thì sử dụng bộ nhớ còn Transient thì không.

## **Finding Persistent Objects**

Bạn sẽ thấy 1 Instance của ViewController bởi vì đó là màn hình hiện tại. Ngoài ra còn có 1 Instance của app là AppDelegate.

Trở lại app, thực hiện search và nhìn kết quả. Bạn sẽ thấy kết quả gồm **SearchResultsViewController** và **ImageCache**.

Bây giờ, ấn nút back trên app. Theo lý thuyết thì **SearchResultsViewController** sẽ bị deallocated, tuy nhiên nó vẫn hiển thị 1 Allocations. Thực hiện thêm 2 lần nữa, bây giờ kết quả là 3 Allocations. Xem ra là đã xảy ra Strong Reference Cycle ở đâu đó.

Có 2 chỗ đáng nghi đó là **SearchResultsViewController** và **SearchResultsCollectionViewCells**. Có thể đã xảy ra reference cycle giữa 2 class này.



## **Getting Visual**

Trong phần này, chúng ta sẽ sử dụng **Visual Memory Debugger** được giới thiệu từ Xcode8 - một tool rất hữu dụng có thể giúp tìm ra memory leaks và retain cycle. 

Thoát Instruments tool.

Trước khi start Visual Memory Debugger, cần config lại một số option trong Xcode Scheme. Chọn Edit Scheme > Diagnostic. Chọn checkbox **Malloc Stack** chọn option **Live Allocations Only**.

Start app và thực hiện search. Để bật **Visual Memory Debugger** các bạn làm step sau: 

1. Click **Debug Memory Graph**.
2. Click the entry for **SearchResultsCollectionViewCell**.
3. Bạn có thể click bất cứ đâu trong phần graph để xem chi tiết. 
4. Phần quan trọng nhất, **Memory Inspector** 

Trong ảnh thì Visual Memory Debugger hiển thị các thông tin sau:

HeapContent (Debug navigator pane): Show danh sách các instance allocated trong memory tại thời điểm pause app. 

Memory Graph: Show hình ảnh đại diện các đối tượng trong bộ nhớ. Các mũi tên giữa các đối tượng đại diện cho các tham chiếu (strong && weak reference)

Memory Inspector: Bao gồm các chi tiết như tên class, tham chiếu mạnh hay yếu...

Trong phần panel Debug navigator, các bạn chọn **SearchResultsViewController** và unfold mũi tên sẽ thấy từng instance của nó

Các mũi tên cùng trỏ tới **SearchResultsViewController**. Có vẻ như có vài closure cùng tham chiếu tới một ViewController instance. Chọn một trong các mũi tên để hiển thị thêm thông tin

Trong Memory Inspector bạn có thể thấy tham chiếu giữa closure và **SearchResultsViewController** là strong reference. Nếu bạn chọn **SearchResultsCollectionViewCell** và closure của nó thì nó cũng đang là một strong reference. Bạn có thể thấy tên của closure name đó `heartToggleHandler` được define trong **SearchResultsCollectionViewCell**.

Chọn 1 instance của **SearchResultsCollectionViewCell**
 để thấy chi tiết hơn trong **Memory Inspector**. Tiếp tục click chi tiết trong phần backtrace bạn sẽ thấy nó dẫn tới `collectionView(_:cellForItemAt:)`. Khi bạn di chuột vào thì sẽ xuất hiện một dấu mũi tên nhỏ. Click vào mũi tên và Xcode sẽ show đoạn mã code
 
Bạn có thể thấy đoạn code trong `collectionView(_:cellForItemAt:)`:

```swift
resultsCell.heartToggleHandler = { _ in
  self.collectionView.reloadItems(at: [indexPath])
}
```

Closure này giúp việc handle khi user tap vào icon trái tim ở collection view cell. Đây là nơi xảy ra Strong reference, nó sẽ rất khó phát hiện nếu bạn đã gặp case này trước đó. Nhờ **Visual Memory Debugger** mà chúng ta có thể phát hiện đc lỗi memory leak này. 

## Breaking That Cycle

Để phá vỡ được liên kết này chúng ta sẽ sử dụng từ khóa **weak** hoặc **unowned**

**Weak**: Sử dụng khi tham chiếu có thể bị nil, nếu đối tượng mà nó tham chiếu đến bị dellocated, thì tham chiếu sẽ trở thành nil. Nó là kiểu optional type.

**Unowned**: Sử dụng khi closure và đối tượng nó tham chiếu tới có cùng lifetime và cùng deallocated tại một thời điểm. Một unowned reference vẫn có thể nil và sẽ gặp lỗi `explicitly unwrapped optional` nếu nó vẫn được truy cập khi đã bị deallocated. Vì vậy, các bạn nên sử dụng một cách cẩn thận, chỉ khi bạn biết chắc chắn rằng nó không bị nil vào thời điểm nó được tham chiếu tới.

Quay lại project, để fix strong reference các bạn thêm vào closure như sau:

```swift
resultsCell.heartToggleHandler = { [weak self] _ in
  self?.collectionView.reloadItems(at: [indexPath])
}
```

Với việc define **weak** có nghĩa là **SearchResultsViewController** có thể bị deallocated ngay cả khi collection view cell vẫn giữ tham chiếu tới nó. Vì nó tham chiếu yếu và sau khi **SearchResultsViewController** bị deallocated thì các view và cell cũng lần lượt bị deallocated theo. Chỗ này các bạn cũng có thể dùng **unowned** nhé.

Mở lại Xcode Instrument Allocation để kiểm tra, thực hiện search sau đó back lại thì **SearchResultsViewController** và cell đã bị deallocated. Nó chỉ show ở transient và không show ở persistent.
