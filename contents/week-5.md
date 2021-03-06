# iOS 질문 모음 - 5

### Q.

> 왜 delegate 를 weak 으로 선언하나요?

delegation 패턴에서 delegate 가 weak 으로 선언된 것을 확인할 수 있는데요. 왜 delegate 를 weak 으로 선언하나요?

[질문 바로가기](https://stackoverflow.com/questions/8449040/why-use-weak-pointer-for-delegation)

### A.

* [retain cycle](https://docs.swift.org/swift-book/LanguageGuide/AutomaticReferenceCounting.html#ID51) 을 피하기 위해 delegate 를 weak 으로 선언합니다. retain cycle 은 두 클래스 인스턴스가 서로에 대한 강력한 참조를 가질 때 발생합니다.

  흔히 사용되는 UITableView 로 예를 들어보겠습니다. 

  ```swift
  class ViewController: UIViewController, UITableViewDelegate {
      // ViewController 가 tableView 를 소유
      var tableView: UITableView = UITableView()
      
      override func viewDidLoad() {
          super.viewDidLoad()
          view.addSubview(tableView)
          // tableView 의 delegate 로 자기 자신(ViewController)을 지정
          tableView.delegate = self
      }
  }
  
  ```

  편의를 위해 레이아웃 설정 코드는 생략하였습니다. ViewController 가 tableView 를 소유하고 있고, ViewController 를 tableView 의 delegate 로 지정합니다.

  위 코드를 그림으로 표현하면 다음과 같은데요.

  <img width="420" alt="스크린샷 2020-08-05 오후 10 02 26" src="https://user-images.githubusercontent.com/50410213/89417638-e31ba880-d769-11ea-8b63-42df3e364f91.png">

  ViewController 가 tableView 를 강력하게 참조하고, tableView 의 delegate 가 ViewController 를 약하게 참조합니다. 공식문서에서 UITableView 의 [delegate](https://developer.apple.com/documentation/uikit/uitableview/1614894-delegate) 를 찾아보면 weak 으로 선언되어있는 것을 확인할 수 있습니다.

  ```swift
  weak var delegate: UITableViewDelegate? { get set }
  ```

  만약 delegate 가 strong 으로 선언되어있었다면 서로가 서로를 강하게 참조하는 retain cycle 이 발생하여 두 객체가 메모리에서 해제되지 않는 문제가 발생할 것입니다.

* delegate 를 누구로 지정 해주냐에 따라 strong 으로 선언해도 되는 경우도 존재합니다.

  ```swift
  protocol CustomViewDelegate: class {
      func userDidTap()
  }
  
  class CustomView: UIView {
      var delegate: CustomViewDelegate?
      
      func mockDelegatecall() {
          delegate?.userDidTap()
      }
  }
  
  // UserViewDelegate 프로토콜을 채택한 클래스
  class Delegate: CustomViewDelegate {
      func userDidTap() {
          print("tap!")
      }
  }
  
  class ViewController: UIViewController {
      // 별도의 delegate 객체를 생성
      let customView = CustomView()
      let customViewDelegate = Delegate()
      
      override func viewDidLoad() {
          super.viewDidLoad()
          view.addSubview(customView)
          customView.delegate = customViewDelegate
      }
  }
  ```

  ViewController 가 customView 와 customViewDelegate 를 소유하고 있고 customView 의 delegate 로 customViewDelegate 를 지정해주었습니다.

  위 코드를 그림으로 표현하면 다음과 같습니다.

  <img width="488" alt="스크린샷 2020-08-05 오후 10 44 09" src="https://user-images.githubusercontent.com/50410213/89420438-aea9eb80-d76d-11ea-917a-23051dfc29ab.png">

  delegate 를 담당하는 별도의 객체를 생성함으로써 customView 내부에 delegate 를 weak 으로 선언하지 않았지만, retain cycle 이 발생하지 않습니다. 

  추가로 delegate 를 구조체로 지정하는 경우에도 weak 을 사용하지 않아도 됩니다. 구조체는 값 유형이기 때문에 retain cycle 을 발생시키지 않습니다.

* weak 으로 선언하지 않아도 되는 경우도 있으나, 복잡한 구조의 코드에서 weak 으로 선언하지 않은 delegate 는 예상치 못한 retain cycle 을 발생시킬 수 있으니 weak 으로 선언하는 것을 권장합니다.

### 참고할 만한 비슷한 질문, 자료

* [Swift delegation - when to use weak pointer on delegate](https://stackoverflow.com/questions/30056526/swift-delegation-when-to-use-weak-pointer-on-delegate?rq=1)
* [How can I make a weak protocol reference in 'pure' Swift (without @objc)](https://stackoverflow.com/questions/24066304/how-can-i-make-a-weak-protocol-reference-in-pure-swift-without-objc)
* [swift delegation - when to use weak reference, why 'delegate' is nil?](https://stackoverflow.com/questions/34611873/swift-delegation-when-to-use-weak-reference-why-delegate-is-nil)

-----

### Q.

> frame 과 bounds 의 차이가 무엇인가요?

UIView 과 UIView 를 상속받는 모든 뷰들은 frame 과 bounds 를 가지는데, 이 둘은 어떤 차이가 있나요?

[질문 바로가기](https://stackoverflow.com/questions/1210047/cocoa-whats-the-difference-between-the-frame-and-the-bounds)

### A.

* [frame](https://developer.apple.com/documentation/uikit/uiview/1622621-frame) 은 **부모 뷰의 좌표계**에서 뷰 위치와 크기를 정의합니다. 반면에 [bounds](https://developer.apple.com/documentation/uikit/uiview/1622580-bounds) 는 **자체 좌표계**에서 뷰의 위치와 크기를 정의합니다.

* 다음 그림은 같은 뷰에서의 frame 과 bounds 을 표현한 것입니다. 두 이미지에서 빨간색 점은 frame, bounds 의 원점을 나타냅니다. 각각을 설명할 때 frame 은 부모 뷰의 좌표계를 기준으로 하므로 부모 뷰가 필요하지만, bounds 는 자체 좌표계를 기준으로 해서 부모 뷰와는 관계가 없기에 표시해주지 않았습니다.

  ![1](https://i.stack.imgur.com/SI1lH.png)

  ```swift
  Frame
      origin = (0, 0)
      width = 80
      height = 130
  Bounds 
      origin = (0, 0)
      width = 80
      height = 130
  ```

  현재 상태에서의 frame 과 bounds 는 동일합니다.

  ![2](https://i.stack.imgur.com/SGdGA.png)

  ```swift
  Frame
      origin = (40, 60)
      width = 80
      height = 130
  Bounds 
      origin = (0, 0)
      width = 80
      height = 130
  ```

  뷰의 위치를 변경하면 frame 의 좌푯값이 바뀌는 것을 확인할 수 있습니다. frame 이 부모 뷰의 좌표계를 기준으로 하기 때문인데요. 이와 달리 bounds 는 자기 자신을 기준으로 봤을 땐 아무런 변동사항이 없으므로 값에 아무런 변화가 없습니다.

* 지금까지 frame 의 width, height 와 bounds 의 width, height 는 같았습니다. 하지만 이 두 값이 항상 같지만은 않습니다.

  ![3](https://i.stack.imgur.com/kBHgE.png)

  ```swift
  Frame
      origin = (20, 52)
      width = 118
      height = 187
  Bounds 
      origin = (0, 0)
      width = 80
      height = 130
  ```

  [transform](https://developer.apple.com/documentation/quartzcore/calayer/1410836-transform) 을 활용해 뷰의 중점을 기준으로 뷰를 회전한 모습입니다. 뷰 자체에는 변화가 없기에 여전히 bounds 는 변화가 없지만, frame 의 크기는 달라진 것을 확인할 수 있습니다. 

  [I Totally Didn't Understand Frames and Bounds](https://ashfurrow.com/blog/i-totally-didnt-understand-frames-and-bounds/) 에서는 frame 을 `부모 뷰의 좌표계를 기준으로 뷰에 적용된 모든 변형을 감싸는 가장 작은 상자` 라고 표현합니다. 이제 frame 과 bounds 의 차이가 어느 정도 보이지 않나요? 그럼 다음 예시로 넘어가겠습니다.

* 지금까지의 예시에서 bounds 의 원점은 항상 (0,0) 이었습니다. 그럼 언제 bounds 의 원점이 이동할까요?

  ![4](https://i.stack.imgur.com/71kvH.png)

  ```swift
  Frame
      origin = (40, 60)
      width = 80
      height = 130
  Bounds 
      origin = (0, 0)
      width = 80
      height = 130
  ```

  뷰의 자식 뷰가 너무 커서 한 번에 표시할 수 없는 경우입니다. 자신의 크기보다 큰 이미지 뷰를 자식 뷰로 가진 모습을 볼 수 있습니다. 뷰의 크기만큼만 이미지가 표시되고 있습니다.

  ![5](https://i.stack.imgur.com/4izhy.png)

  ```swift
  Frame
      origin = (40, 60)
      width = 80
      height = 130
  Bounds 
      origin = (280, 70)
      width = 80
      height = 130
  ```

  bounds 의 원점을 이동한 모습입니다. 부모 뷰를 기준으로 봤을 때 뷰의 프레임은 이동하지 않았지만, 뷰의 bounds 의 원점 좌표가 변경되었기 때문에 뷰가 표시하는 이미지 영역이 변경되었습니다. UIScrollView 가 이와 같은 원리로 동작합니다. UIScrollView 에 대한 자세한 정보를 알고 싶다면 [Understanding UIScrollView](https://oleb.net/blog/2014/04/understanding-uiscrollview/) 를 참고하세요.

* 그럼 frame 과 bounds 는 어떤 경우에 사용할까요?

  * frame 은 부모 뷰를 기준으로 위치와 크기를 계산하기 때문에 뷰의 위치와 관련된 계산을 하거나, 뷰의 크기를 변경하는 등 외부의 변경을 수행할 때 사용합니다.

  * bounds 는 [draw(_:)](https://developer.apple.com/documentation/uikit/uiview/1622529-drawrect) 를 활용해 뷰 내부에 그림을 그리거나, transform 변형한 뷰의 크기를 알고 싶을 때, 자식 뷰를 정렬하는 것과 같이 내부적인 변경을 수행할 때 사용합니다.

* [FrameVsBounds](https://github.com/maniramezan/FrameVsBounds) 에 frame 과 bounds 의 변화를 한눈에 보기 쉽게 만든 앱이 있습니다. 글만으론 frame 과 bounds 의 차이가 이해가 잘 안 된다면 참고해보세요.

  <img width="40%" alt="sample" src="https://github.com/maniramezan/FrameVsBounds/blob/master/images/IMG_E5039F2BB59E-1.jpeg?raw=true">

### 참고할 만한 비슷한 질문, 자료

* [UIView frame, bounds and center](https://stackoverflow.com/questions/5361369/uiview-frame-bounds-and-center)
* [iOS ) Frame과 Bounds의 차이 (1/2)](https://zeddios.tistory.com/203)
* [Stanford University - CS193P: Views, Drawing, Animation](https://www.slideshare.net/profmido/05-views)

----

### Q.

> App Transport Security 가 HTTP 통신을 차단해요.

<img width="1106" alt="스크린샷 2020-08-08 오전 10 31 52" src="https://user-images.githubusercontent.com/50410213/89701206-90abd900-d96f-11ea-8c6b-5cef47c9e8f6.png">

App Transport Security 에 의해 http 리소스 로드가 차단됩니다. 차단을 해제하려면 Info.plist 를 어떻게 수정해야 하나요?

[질문 바로가기](https://stackoverflow.com/questions/31254725/transport-security-has-blocked-a-cleartext-http/32560433#32560433)

### A.

* [ATS(App Transport Security)](https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/Articles/CocoaKeys.html#//apple_ref/doc/uid/TP40009251-SW33) 는 iOS 9 에서 도입된 개인정보 보호 기능입니다. 앱의 네트워크 연결을 취약점이 없는 산업 표준 프로토콜과 암호만 사용하도록 하여 개인 정보 보호 및 [데이터 무결성](https://ko.wikipedia.org/wiki/데이터_무결성)을 향상시킵니다. 이는 사용자가 개인 정보 유출에 대한 걱정 없이 앱을 사용할 수 있도록 해줍니다.

* 앱에서 네트워크 통신을 할 때 HTTPS 가 아닌 HTTP 를 쓰게 된다면 네트워크 통신에서 안전성을 보장하지 못하므로 ATS 가 보안을 위해 차단하게 됩니다. HTTP 를 꼭 사용해야만 한다면 Info.plist 에서 특정 도메인에 대해 예외처리를 해줄 수 있습니다.

  <img width="534" alt="스크린샷 2020-08-08 오전 11 02 29" src="https://user-images.githubusercontent.com/50410213/89704007-446e9200-d98b-11ea-8f72-120a4e03bb4f.png">

  먼저 + 버튼을 눌러 `Information Property List` 에 `App Transport Security Settings` 를 추가합니다.  `App Transport Security Settings` 에 예외처리할 도메인들을 담을 `Exception Domains` 를 만든 후, 예외처리할 도메인(예시에서는 `randomuser.me`)을 추가하고, `NSTemporaryExceptionAllowsInsecureHTTPLoads` 를 YES 로 설정해줍니다. 

  코드로 추가한다면 다음과 같습니다. Info.plist 에 마우스 오른쪽 클릭을 하면 나오는 메뉴에서 Open As -> Source Code 를 누르면 코드로 키를 추가할 수 있습니다.

  <img width="456" alt="스크린샷 2020-08-08 오후 3 37 11" src="https://user-images.githubusercontent.com/50410213/89704231-096d5e00-d98d-11ea-9310-3acd44a94b8b.png">

  기존 `<plist>` 의 `<dict>` 안에 아래의 코드블럭을 추가하면 됩니다.

  ```swift
  <key>NSAppTransportSecurity</key>
  <dict>
  	<key>NSExceptionDomains</key>
  	<dict>
  		<key>randomuser.me</key>
  		<dict>
  			<key>NSTemporaryExceptionAllowsInsecureHTTPLoads</key>
  			<true/>
  		</dict>
  	</dict>
  </dict>
  ```

* `App Transport Security Settings` 의  `Allow Arbitrary Loads` 를 Yes 로 설정하여 ATS 를 비활성화하는 방법도 있지만, 디버깅이나 개발 과정에서만 사용하고 실제 앱 출시에는 보안 문제가 발생할 수 있으므로 권장하지 않습니다.

* 이 외에도 ATS 의 설정을 바꿀 수 있는 여러 가지 키들이 존재합니다. 더 자세한 정보는 [App Transport Security dictionary primary keys](https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/Articles/CocoaKeys.html#//apple_ref/doc/uid/TP40009251-SW34) 와 [Exception domains dictionary keys](https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/Articles/CocoaKeys.html#//apple_ref/doc/uid/TP40009251-SW44) 를 참고하세요.

### 참고할 만한 비슷한 질문, 자료

* [How can I add NSAppTransportSecurity to my info.plist file?](https://stackoverflow.com/questions/31216758/how-can-i-add-nsapptransportsecurity-to-my-info-plist-file/31629980#31629980)
* [HTTP VS HTTPS 차이, 알면 사이트의 레벨이 보인다.](http://blog.wishket.com/http-vs-https-차이-알면-사이트의-레벨이-보인다/)
* [SSL, TLS, HTTPS 의 정의](https://www.digicert.com/kr/what-is-ssl-tls-https/)
